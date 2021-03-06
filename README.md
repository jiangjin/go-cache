# go-cache

go-cache is an in-memory key:value store/cache similar to memcached that is
suitable for applications running on a single machine. Its major advantage is
that, being essentially a thread-safe `map[string]interface{}` with expiration
times, it doesn't need to serialize or transmit its contents over the network.

Any object can be stored, for a given duration or forever, and the cache can be
safely used by multiple goroutines.

Although go-cache isn't meant to be used as a persistent datastore, the entire
cache can be saved to and loaded from a file (using `c.Items()` to retrieve the
items map to serialize, and `NewFrom()` to create a cache from a deserialized
one) to recover from downtime quickly. (See the docs for `NewFrom()` for caveats.)

### Updates

add the limitation of max capacity on the total memory consumption.

- first version: If the cache is full, stop to set the new key.

- second version: If the cache is full, run some replacement algorithm.

### Installation

`go get github.com/pmylund/go-cache`

### Usage

	import (
		"fmt"
		"github.com/pmylund/go-cache"
		"time"
	)

	func main() {

		// Create a cache with a default expiration time of 5 minutes, and which
		// purges expired items every 30 seconds
		c := cache.New(5*time.Minute, 30*time.Second)

		// Set the value of the key "foo" to "bar", with the default expiration time
		c.Set("foo", "bar", cache.DefaultExpiration)

		// Set the value of the key "baz" to 42, with no expiration time
		// (the item won't be removed until it is re-set, or removed using
		// c.Delete("baz")
		c.Set("baz", 42, cache.NoExpiration)

		// Get the string associated with the key "foo" from the cache
		foo, found := c.Get("foo")
		if found {
			fmt.Println(foo)
		}

		// Since Go is statically typed, and cache values can be anything, type
		// assertion is needed when values are being passed to functions that don't
		// take arbitrary types, (i.e. interface{}). The simplest way to do this for
		// values which will only be used once--e.g. for passing to another
		// function--is:
		foo, found := c.Get("foo")
		if found {
			MyFunction(foo.(string))
		}

		// This gets tedious if the value is used several times in the same function.
		// You might do either of the following instead:
		if x, found := c.Get("foo"); found {
			foo := x.(string)
			// ...
		}
		// or
		var foo string
		if x, found := c.Get("foo"); found {
			foo = x.(string)
		}
		// ...
		// foo can then be passed around freely as a string

		// Want performance? Store pointers!
		c.Set("foo", &MyStruct, cache.DefaultExpiration)
		if x, found := c.Get("foo"); found {
			foo := x.(*MyStruct)
			// ...
		}

		// If you store a reference type like a pointer, slice, map or channel, you
		// do not need to run Set if you modify the underlying data. The cached
		// reference points to the same memory, so if you modify a struct whose
		// pointer you've stored in the cache, retrieving that pointer with Get will
		// point you to the same data:
		foo := &MyStruct{Num: 1}
		c.Set("foo", foo, cache.DefaultExpiration)
		// ...
		x, _ := c.Get("foo")
		foo := x.(*MyStruct)
		fmt.Println(foo.Num)
		// ...
		foo.Num++
		// ...
		x, _ := c.Get("foo")
		foo := x.(*MyStruct)
		foo.Println(foo.Num)

		// will print:
		// 1
		// 2

	}


### Reference

`godoc` or [http://godoc.org/github.com/pmylund/go-cache](http://godoc.org/github.com/pmylund/go-cache)
