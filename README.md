# goava

Goava is a set of libraries based on draft version of Go 2 generics.

Read more about go 2 generics: [https://blog.golang.org/generics-next-step](https://blog.golang.org/generics-next-step)

[Read the blog post about this project.](https://snake-ci.com/blog/go2go-stream)

## Stream(T)

### Initialize stream

```go
Of([]int{1, 2, 3, 4})

or

Of([]string{"q", "w", "e"})
```

## Filtering a stream

Let's get rid of some numbers
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

The first argument to `ToMap` is a mapper, it receives original item and returns a tuple of key and value.
The second argument to `ToMap` is a accumulator, it current value and new value given by mapper or during combining
parallel stream chunks. Yes, *parallel*.

A stream can be switched to parallel mode, so mapping and accumulation will be done in concurrent mode.

```go
items = items.Parallel()
```

A stream can be switched back to sequential mode by calling `Sequential()` function.

### License

MIT
