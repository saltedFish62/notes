# Concurrency 并发

## Goroutines

​	goroutine 是一个由 Go runtime 管理的轻量级线程。goroutine 的本质是协程，是实现并行计算的核心。

```go
go f(x, y, z)
```

​	开始一个新的 goroutine running

```go
f(x, y, z)
```

​	`f`, `x`, `y`, `z` 的求值发生在当前协程并且 `f` 的执行发生在一个新的协程中。

​	这些协程都在同样的地址空间中运行，所以访问共享的内存必须同步。`sync` 包提供了原语，尽管你不太需要它们，因为 Go 里还有其他原语。

```go
package main
import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```

## Channels

​	Channels 是一种类型化的管道，通过它可以使用操作符 `<-` 发送和接受数据。

```go
ch <- v    // 发送 v 到 channel ch
v := <-ch  // 接受来自 ch 的数据，并赋值给 v
```

​	数据沿箭头的方向流动。

​	就像 maps  和 Slices 一样，channels 必须在使用之前被创建：

```go
ch := make(chan int)
```

​	默认的，发送和接受会阻塞直到另一端已经准备好。这允许 goroutines 可以不需要另外的锁或者条件变量就可以同步。示例代码把一个切片里的数组累加的工作分配给两个 goroutines。一旦两个 goroutines 都完成了计算，就会计算最终的结果。

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```

## Buffered Channels

​	Channels 可以被缓存。向 `make` 函数提供缓存池大小作为第二个参数来初始化 buffered channel： `ch := make(chan int, 100)`

​	发送给一个 buffered channel 只会在缓存池满的时候阻塞。当缓存池为空时接受会产生阻塞。

## Range and Close

​	一个发送者可以 `close(关闭)` 一个 channel 来表示不会再发送数据。接受者可以通过使用两个参数来接受 channel 的赋值来测试 channel 是否已被关闭。

```go
v, ok := <-ch
```

​	`ok` 是 `false` 如果没有更多值可以接收，并且 channel 是关闭的。

​	循环 `for i := range c` 从 channel 中重复接收数据直到它关闭。

> 注意：只有发送者该关闭 channel，接收者则永远不能关闭。向一个已关闭的信道发送数据将会导致一个 panic

> 注意：channels 不像文件，你不需要经常关闭他们。只当需要告诉接收者不再有数据传入时，关闭 channel 才是必要的，比如终止一个 `range` 循环。

```go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

## Select

​	`select` 语句使 goroutine 在多交流操作中等待。一个 `select` 会阻塞直到其中一个 case 可以运行，然后执行这个 case。如果多个 case 同时可以运行，则会随机选择一个执行。

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {    // highlight here
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

​	selection 有默认值 default。当 select 的其他 case 不满足时，执行该默认的条件。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select {
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

## Exercise: Equivalent Binary Trees

There can be many different binary trees with the same sequence of values stored at the leaves. For example, here are two binary trees storing the sequence 1, 1, 2, 3, 5, 8, 13.

![img](http://ww2.sinaimg.cn/large/006y8mN6gy1g6ddvv60zvj30dn0513ym.jpg)

This example uses the `tree` package, which defines the type:

```
type Tree struct {
    Left  *Tree
    Value int
    Right *Tree
}
```

**1.** Implement the `Walk` function.

**2.** Test the `Walk` function.

The function `tree.New(k)` constructs a randomly-structured (but always sorted) binary tree holding the values `k`, `2k`, `3k`, ..., `10k`.

Create a new channel `ch` and kick off the walker:

```
go Walk(tree.New(1), ch)
```

Then read and print 10 values from the channel. It should be the numbers 1, 2, 3, ..., 10.

**3.** Implement the `Same` function using `Walk` to determine whether `t1` and `t2` store the same values.

**4.** Test the `Same` function.

`Same(tree.New(1), tree.New(1))` should return true, and `Same(tree.New(1), tree.New(2))` should return false.

```go
package main

import (
	"fmt"

	"golang.org/x/tour/tree"
)

// Walk walks the tree t sending all values
// from the tree to the channel ch.
func Walk(t *tree.Tree, ch chan int) {
	WalkTree(t, ch)  // 在使用 go 关键字调用 Walk 时，它不能调用自身来递归，
  								 // 所以需要 WalkTree 来递归遍历树
	close(ch)
}

func WalkTree(t *tree.Tree, ch chan int) {
	if t.Left != nil {
		WalkTree(t.Left, ch)
	}
	ch <- t.Value
	if t.Right != nil {
		WalkTree(t.Right, ch)
	}
}

// Same determines whether the trees
// t1 and t2 contain the same values.
func Same(t1, t2 *tree.Tree) bool {
	ch1, ch2 := make(chan int), make(chan int)
	go Walk(t1, ch1)
	go Walk(t2, ch2)
	for v := range ch1 {
		if v != <-ch2 {
			return false
		}
	}
	return true
}

func main() {
	ch := make(chan int)
	go Walk(tree.New(1), ch)
	for v := range ch {
		fmt.Println(v)
	}
	fmt.Println("should return true: ", Same(tree.New(1), tree.New(1)))
	fmt.Println("should return false: ", Same(tree.New(1), tree.New(2)))
}
```

## sync.Mutex 同步互斥锁

> Mutex - Mutual exclusion 互斥锁

​	当我们只需要一个协程可以在同一时间访问一个变量而避免冲突，则可以考虑 互斥锁。Go 的基础库提供 `sync.Mutex` 和它的两个方法： `Lock` , `Unlock`

​	我们可以通过调用 `Lock` 和 `Unlock` 来定义在互斥锁中执行的代码块。我们也可以使用 `defer` 来保证互斥锁将会在 `Value` 方法被解锁。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}

// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}

func main() {
	c := SafeCounter{v: make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("somekey")
	}

	time.Sleep(time.Second)
	fmt.Println(c.Value("somekey"))
}
```

## Exercise: Web Crawler 爬虫

In this exercise you'll use Go's concurrency features to parallelize a web crawler.

Modify the `Crawl` function to fetch URLs in parallel without fetching the same URL twice.

*Hint*: you can keep a cache of the URLs that have been fetched on a map, but maps alone are not safe for concurrent use!

```go
package main

import (
	"fmt"
	"sync"
)

type Fetcher interface {
	// Fetch returns the body of URL and
	// a slice of URLs found on that page.
	Fetch(url string) (body string, urls []string, err error)
}

type vistedUrls struct {
	list map[string]bool
	mux sync.Mutex
	wg sync.WaitGroup
}

type response struct {
	url string
	body string
}

// Crawl uses fetcher to recursively crawl
// pages starting with url, to a maximum of depth.
func Crawl(url string, depth int, fetcher Fetcher, visited *vistedUrls) {
	// TODO: Fetch URLs in parallel.
	// TODO: Don't fetch the same URL twice.
	// This implementation doesn't do either:
  
	defer visited.wg.Done()
  
	if depth <= 0 {
		return
	}

	// record visited url
	visited.mux.Lock()
	if visited.list[url] {
		visited.mux.Unlock()
		return
	}
	visited.list[url] = true
	visited.mux.Unlock()

	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)
	for _, u := range urls {
		visited.wg.Add(1)
		go Crawl(u, depth-1, fetcher, visited)
	}
}

func main() {
	visited := vistedUrls{ list: make(map[string]bool)}
	visited.wg.Add(1)
	go Crawl("https://golang.org/", 4, fetcher, &visited)
	visited.wg.Wait()
}

// fakeFetcher is Fetcher that returns canned results.
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher is a populated fakeFetcher.
var fetcher = fakeFetcher{
	"https://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"https://golang.org/pkg/",
			"https://golang.org/cmd/",
		},
	},
	"https://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"https://golang.org/",
			"https://golang.org/cmd/",
			"https://golang.org/pkg/fmt/",
			"https://golang.org/pkg/os/",
		},
	},
	"https://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
	"https://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"https://golang.org/",
			"https://golang.org/pkg/",
		},
	},
}

```

