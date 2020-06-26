# goava

Goava is a set of libraries based on draft version of Go 2 generics.

[Read the blog post about this project](https://snake-ci.com/blog/go2go-stream)

It's not ready for production or even development use.

Bugs found while working on this project:
* [39834][bug-1]
* [39839][bug-2]
* [39853][bug-3]
* [39878][bug-4]

Read more about the draft design of Go Generics:
[The Next Step for Generics][golang-post]

## Table Of Contents

* [Stream(T)](#streamt)
* [Result(T)](#resultt)

# Stream(T)

### Initialize stream

```go
Of([]int{1, 2, 3, 4})

or

Of([]string{"q", "w", "e"})
```

## Filtering a stream

Let's get rid of some numbers:

```go
stream.
    Of([]int{1, 2, 3, 4, 5}).
    Filter(func(x int) bool {
        return x%2 == 0
    }).
    Slice()
```

## Mapping stream of A to stream of B

Let's get rid of some numbers
```go
stream.Map(
    stream.Of([]int{1, 2}),
    func(x int) string { return fmt.Sprint(x) },
).Slice()
```

## Matching

* `Stream(Type).AnyMatch(func(Type) bool)` returns true if any of elements in stream match given predicate
* `Stream(Type).NoneMatch(func(Type) bool)` returns true if none of elements in stream match given predicate
* `Stream(Type).AllMatch(func(Type) bool)` returns true if all of elements in stream match given predicate

## Collecting Stream(Type) to a map[Key]Value

Define a stream of movies with name and score that users gave:
```go
items := stream.Of(
    []MovieVote{
        {Name: "A", Score: 1},
        {Name: "A", Score: 2},
        {Name: "B", Score: 9},
        {Name: "B", Score: 10},
        {Name: "B", Score: 8},
        {Name: "C", Score: 7},
        {Name: "C", Score: 8},
        {Name: "C", Score: 7},
    },
)
```

Now let's collect create a hashmap where key is a name of movie and value is sum of scores:

```go
result := stream.Collect(
    items,
    collector.ToMap(MovieVote, string, int)(
        func(movie MovieVote) (string, int) {
            return movie.Name, movie.Score
        },
        func(total, score int) int {
            return total + score
        },
    ),
)
```

The first argument to `ToMap` is a mapper, it receives original item and returns a tuple of key and value.  The second
argument to `ToMap` is a accumulator, it current value and new value given by mapper or during combining parallel stream
chunks. Yes, *parallel*.

A stream can be switched to parallel mode, so mapping and accumulation will be done in concurrent mode.

```go
items = items.Parallel()
```

A stream can be switched back to sequential mode by calling `Sequential()` function.

# Result(T)

The main goal was to bring Rust's powerful [Result<T>](https://doc.rust-lang.org/std/result/enum.Result.html) to Go.
Unfortunately, it can't be ported completely due to lack of pattern matching in Go.

The only possible way was to use channels and select {}, but it would be too complicated and it would be just an imitation.

Methods implemented so far:
### IsOk() bool
IsOk returns true if the result is Ok.

### IsErr() bool
IsErr returns true if the result is Err.

### And(res Result(T)) Result(T)
And returns res if the result is Ok, otherwise returns the Err value of self.

### OrElse(func() Result(T)) Result(T)
OrElse Calls fn if the result is Err, otherwise returns the Ok value of self.

### Or(res Result(T)) Result(T)
Returns res if the result is Err, otherwise returns the Ok value of self.

### Ok() T
OK returns the contained Ok value.
Panics if the result is an Err

### Err() error
Err returns the contained Err value. Panics if the result is Ok

### OkOr(T) T
OkOr returns the contained Ok value or a provided default.
### OkOrElse(fn func() T) T
OkOrElse returns the contained Ok value or calls provided fn.

### Expect(msg string) T
Expect returns the contained Ok value.
Panics if the value is an Err, with a panic message including the passed message, and the content of the Err.

### ExpectErr(msg string) error
Expect returns the contained Err value.
Panics if the value is an Ok, with a panic message including the passed
message, and the content of the Ok.

## Example

```go
func Success() Result(int) {
    return Ok(42)
}

func Fail() Result(int) {
    return Err(int)(errors.New("failure"))
}

func main() {
    fmt.Println(Fail().OrElse(Success))

    result := Success().And(Fail())
    if result.IsOk() {
        fmt.Println("it's ok", result.Ok())
    } else {
        fmt.Println("it's not ok, but here is default value:", result.OkOr(1))
    }
}
```


### License

MIT

[bug-1]: https://github.com/golang/go/issues/39834
[bug-2]: https://github.com/golang/go/issues/39839
[bug-3]: https://github.com/golang/go/issues/39853
[bug-4]: https://github.com/golang/go/issues/39878
[golang-post]: https://blog.golang.org/generics-next-step
