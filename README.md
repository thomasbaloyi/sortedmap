# 📚 sortedmap

`sortedmap` provides an effective sorted map implementation for Go.
It uses a heap to maintain order and iterators under the hood.

---

[![Build Status](https://github.com/egregors/sortedmap/workflows/build/badge.svg)](https://github.com/egregors/sortedmap/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/egregors/sortedmap)](https://goreportcard.com/report/github.com/egregors/sortedmap)
[![Coverage Status](https://coveralls.io/repos/github/egregors/sortedmap/badge.svg)](https://coveralls.io/github/egregors/sortedmap)
[![godoc](https://godoc.org/github.com/egregors/sortedmap?status.svg)](https://godoc.org/github.com/egregors/sortedmap)

## Features

* 🚀 Efficient sorted map implementation
* 🔧 Customizable sorting by key or value
* 🐈 Zero dependencies
* 📦 Easy to use API (inspired by the stdlib `maps` and `slices` packages)

## Installation

To install the package, run:

```sh
go get github.com/egregors/sortedmap
```

## Usage

### `New` — create an empty sorted map

`New` creates an empty `SortedMap`. You supply a `less` function that defines the sort order.

```go
m := sm.New[map[string]int, string, int](func(i, j sm.KV[string, int]) bool {
    return i.Key < j.Key
})
m.Insert("Bob", 31)
m.Insert("Alice", 26)
fmt.Println(m.CollectKeys()) // [Alice Bob]
```

### `NewFromMap` — create a sorted map from an existing map

`NewFromMap` initializes a `SortedMap` from a regular Go map and a `less` function.

```go
m := sm.NewFromMap(map[string]int{
    "Bob":   31,
    "Alice": 26,
    "Eve":   84,
}, func(i, j sm.KV[string, int]) bool {
    return i.Key < j.Key
})
fmt.Println(m.CollectKeys()) // [Alice Bob Eve]
```

You can sort by value instead of key, or combine both for stable ordering:

```go
type Person struct {
    Name string
    Age  int
}

// Sort by age; break ties alphabetically by name
m := sm.NewFromMap(map[string]Person{
    "Bob":   {"Bob", 26},
    "Alice": {"Alice", 26},
    "Eve":   {"Eve", 84},
}, func(i, j sm.KV[string, Person]) bool {
    if i.Val.Age == j.Val.Age {
        return i.Key < j.Key
    }
    return i.Val.Age < j.Val.Age
})
fmt.Println(m.CollectKeys()) // [Alice Bob Eve]
```

### `Insert` — add or update an entry

`Insert` adds a new key-value pair. If the key already exists it is replaced, and the heap is updated to reflect the new value.

```go
m.Insert("Charlie", 34)
fmt.Println(m.CollectKeys()) // [Alice Bob Charlie Eve]
```

### `Get` — retrieve a value by key

`Get` returns the value for a key and a boolean indicating whether the key was found.

```go
val, ok := m.Get("Alice")
if ok {
    fmt.Println(val) // 26
}

_, ok = m.Get("nobody")
fmt.Println(ok) // false
```

### `Delete` — remove an entry

`Delete` removes a key from the map. It returns a pointer to the removed value and a boolean indicating whether the key existed.

```go
val, existed := m.Delete("Bob")
if existed {
    fmt.Println(*val) // 31
}
fmt.Println(m.CollectKeys()) // [Alice Charlie Eve]
```

### `Len` — number of entries

`Len` returns the number of key-value pairs currently in the map.

```go
fmt.Println(m.Len()) // 3
```

### `All` — iterate in sorted order

`All` returns an `iter.Seq2[K, V]` that yields key-value pairs in the order defined by the `less` function. Compatible with `range` in Go 1.23+.

```go
for k, v := range m.All() {
    fmt.Printf("%s: %d\n", k, v)
}
// Alice: 26
// Bob: 31
// Eve: 84
```

### `Keys` — iterate over keys in sorted order

`Keys` returns an `iter.Seq[K]` of keys in sorted order.

```go
for k := range m.Keys() {
    fmt.Println(k)
}
// Alice
// Bob
// Eve
```

### `Values` — iterate over values in sorted order

`Values` returns an `iter.Seq[V]` of values in sorted order.

```go
for v := range m.Values() {
    fmt.Println(v)
}
// 26
// 31
// 84
```

### `Collect` — export as a regular map

`Collect` returns a regular Go map. Note that Go maps are unordered — use `CollectAll`, `CollectKeys`, or `CollectValues` when insertion order matters.

```go
plain := m.Collect()
fmt.Println(plain) // map[Alice:26 Bob:31 Eve:84]  (order not guaranteed)
```

### `CollectAll` — export as a slice of key-value pairs

`CollectAll` returns a `[]KV[K, V]` slice in sorted order.

```go
pairs := m.CollectAll()
for _, kv := range pairs {
    fmt.Printf("%s: %d\n", kv.Key, kv.Val)
}
// Alice: 26
// Bob: 31
// Eve: 84
```

### `CollectKeys` — export keys as a slice

`CollectKeys` returns a `[]K` slice of keys in sorted order.

```go
fmt.Println(m.CollectKeys()) // [Alice Bob Eve]
```

### `CollectValues` — export values as a slice

`CollectValues` returns a `[]V` slice of values in sorted order.

```go
fmt.Println(m.CollectValues()) // [26 31 84]
```

## API and Complexity

| Method          | Description                                                          | Complexity |
|-----------------|----------------------------------------------------------------------|------------|
| `New`           | Creates a new `SortedMap` with a comparison function                 | O(1)       |
| `NewFromMap`    | Creates a new `SortedMap` from an existing map with a comparison     | O(n log n) |
| `Get`           | Retrieves the value associated with a key                            | O(1)       |
| `Delete`        | Removes a key-value pair from the map                                | O(n)       |
| `All`           | Returns a sequence of all key-value pairs in the map                 | O(n log n) |
| `Keys`          | Returns a sequence of all keys in the map                            | O(n log n) |
| `Values`        | Returns a sequence of all values in the map                          | O(n log n) |
| `Insert`        | Adds or updates a key-value pair in the map                          | O(log n)   |
| `Collect`       | Returns  a regular map with an *unordered* content off the SortedMap | O(n log n) |
| `CollectAll`    | Returns a slice of key-value pairs                                   | O(n log n) |
| `CollectKeys`   | Returns a slice of the map’s keys                                    | O(n log n) |
| `CollectValues` | Returns a slice of the map's values                                  | O(n log n) |
| `Len`           | Returns length of underlying map                                     | O(1)       |

## Benchmarks

```shell
BenchmarkNew-10                         165887913           7.037 ns/op
BenchmarkNewFromMap-10                  419106              2716 ns/op
BenchmarkSortedMap_Get-10               191580795           5.327 ns/op
BenchmarkSortedMap_Delete-10            3328420             365.0 ns/op
BenchmarkSortedMap_All-10               1000000000          0.3116 ns/op
BenchmarkSortedMap_Keys-10              1000000000          0.3118 ns/op
BenchmarkSortedMap_Values-10            1000000000          0.3117 ns/op
BenchmarkSortedMap_Insert-10            6665839             182.5 ns/op
BenchmarkSortedMap_Collect-10           649450              1835 ns/op
BenchmarkSortedMap_CollectAll-10        1237276             972.4 ns/op
BenchmarkSortedMap_CollectKeys-10       1250041             964.9 ns/op
BenchmarkSortedMap_CollectValues-10     1294760             927.7 ns/op
BenchmarkSortedMap_Len-10               1000000000          0.3176 ns/op
```

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
