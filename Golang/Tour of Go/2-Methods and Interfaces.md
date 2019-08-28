# Methods and interfaces

## Methods

​	Go没有类，然而可以在类型里定义方法。类型可以看成一个对象，方法则是对象里的属性。一个方法是一个有特殊接受者(receiver)参数的函数。这个接受者在 `func` 关键字和函数名之间：

```go
package main
import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

// func 和 Abs 之间就是一个接受者
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```

​	所以，method 只是一个有接受者参数的 function。可以定义method的类型不止struct，其他的类型也可以：

```go
package main
import (
	"fmt"
	"math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```

​	需要注意的是：只可以在同一个包里的已定义的type声明method。不可以给别的包定义的type声明method。

## Pointer receivers

​	声明method的时候可以使用指针做接受者。也就是receiver有两种接受的类型：

- 值类型：将接受者复制一份传入method，method中对传入参数的修改不会反映到来源。
- 指针类型：可以理解成传入引用，在method中对传入参数的修改会反映给来源。但指针类型的receiver更常见。

```go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

// 值 receiver
func (v Vertex) Abs() float64 {
	v.X = 1		// 在这里对 v 的修改不会反映到 main 中的 v
	v.Y = 1
	return math.Sqrt(v.X*v.X + v.Y*v.Y)   // 由于前面的修改 所以这里返回根号二
}

// 指针 receiver
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f  // 若将参数中的 * 去除，
	v.Y = v.Y * f	 // 那么这两句的修改将不会反映到 main 中的 v
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)						 // 这里传入的 receiver 是指针
	fmt.Println(v.Abs())   // 这里传入的 receiver 是值 ====> 根号2
  fmt.Println(v)				 // ====> {30, 40}
}
```

​	关于何时使用值或者指针 receiver：

- 当需要修改指针 receiver 指向的值时；
- 当指针 receiver 指向一个大 struct 时，因为此时传值的话，复制的开销太大影响性能。

​    通常来讲，所有的 method 都应该既有 值receiver 和指针receiver，但不能混淆两者。

## Interfaces

​	接口类型定义成一个方法签名的集合。接口类型的值可以包含任何实现这些方法的值，也就是说 `Abser` 作为一个接口可以接到任何类型的值上，在以下代码中体现为可以接 `MyFloat`，也可以接 `vertex`。

​	接口的实现是隐性的。一个类型通过实现它的方法来实现一个接口，但这是隐性的声明，没有 "implements" 这样的关键字。隐式的接口把接口的定义和实现分离，并且接口可以在没有任何预先安排的情况下出现在任意包里。接口 `Abser` 的定义和实现是分离的，第7行是定义，而在第28行和第39行是对接口的实现。

```go
package main
import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	// In the following line, v is a Vertex (not *Vertex)
	// and does NOT implement Abser.
	a = v

	fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

## Interface values 接口值

​	在底层来说，接口值可以认为是一个值和一个具体类型的组合：`(value, type)`

​	一个接口值包含一个具体类型下的确切的值。调用接口定义的方法会执行接口值的类型的同名的方法。

```go
package main

import (
	"fmt"
	"math"
)

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	fmt.Println(t.S)
}

type F float64

func (f F) M() {
	fmt.Println(f)
}

func main() {
	var i I		// 声明 i 为接口 I

  i = &T{"Hello"}   // i 的接口值为 T{"Hello"} 的指针
	describe(i)
	i.M()							// 调用接口 i 的方法，此时会去调用类型 T 的同名方法

  i = F(math.Pi)		// i 的接口值为 F(math.Pi)
	describe(i)
	i.M()							// 与31行相同
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)   // 可以看出接口本质上是值和类型
}

>>>>>>>>>>>>>>>>>>>>>> output:
(&{Hello}, *main.T)
Hello
(3.141592653589793, main.F)
3.141592653589793
<<<<<<<<<<<<<<<<<<<<<<
```

## Interface values with nil underlying values

​	如果接口本身的具体的值是 nil，那么方法会以 nil 作为 receiver 来调用。一些语言会触发空指针异常，但在Go里通常会编写优雅的处理 nil receiver 的方法。

```go
package main

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}

func main() {
	var i I

	var t *T		// 初始化为 nil
	i = t       // 值为初始化的 nil
	describe(i)
  i.M()				// M() 的 receiver 为 nil

	i = &T{"hello"}
	describe(i)
	i.M()
}

func describe(i I) {
	fmt.Printf("(%v, %T)\n", i, i)
}
```

​	对于没有给接口赋值的情况，接口自身的值和类型都为 nil，此时调用接口的方法则会报 runtime error。

```go
type I interface {
	M()
}

func main() {
	var i1 I			// i1 的值为 nil，类型为 nil
	i1.M()				// 报 runtime error
  
  var i2 interface{}  // i2 的值为 nil，类型为 nil
}
```

## Type assertions 类型断言

​	类型断言提供对于一个接口值中的具体值的访问。

​	`t := i.(T)`

​	这个语句断言接口值 `i` 包含具体类型 `T` 并且将值赋给变量 `t`。如果接口 `i` 不包含类型 `T`，那就会触发一个 panic 。为了测试一个接口值包含的特定类型，一个类型断言可以返回两个值： 一个值和一个表示这个断言是否成功的布尔值。

​	`t, ok := i.(T)`

​	如果 `i` 包含 `T`， 那么 `t` 将会有 `i` 的值并且 `ok` 也会是 true。反之 `ok` 会是 false，而 `t` 会是类型 `T` 的零值，并且没有 panic 发生。

```go
package main

import "fmt"

func main() {
	var i interface{} = "hello"

	s := i.(string)
	fmt.Println(s)				// =====> hello

	s, ok := i.(string)
	fmt.Println(s, ok)		// =====> hello true

	f, ok := i.(float64)
	fmt.Println(f, ok)		// =====> 0 false

	f = i.(float64) 			// panic
	fmt.Println(f)
}
```

## Type switches 类型的 switch 语句

​	类型的 switch 由一系列的几个类型断言构成。type switch 和常规的 switch 语句相似，但 case 后面接具体的类型，并且这些值是与给定接口值的类型做比较的。比如：

```go
switch v := i.(type) {
	case T:
  	// do something
	case S:
  	// do something
  default:
  	// do something
}
```

​	在关键字 switch 后的声明跟类型断言 `i.(T)` 的语法一样，但特定的类型 `T` 替换替换成关键字 `type` 。

## Stringers

​	一个最广泛使用的接口是 `fmt` 包中定义的 `Stringer` 。

```go
type Stringer interface {
  String() string
}
```

​	`Stringer` 是一个可以将自身描述成字符串的类型。`fmt` 包靠这个接口来打印值。

## Exercise: Stringers

Make the `IPAddr` type implement `fmt.Stringer` to print the address as a dotted quad.

For instance, `IPAddr{1, 2, 3, 4}` should print as `"1.2.3.4"`.

```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr) String() string {
	var result string
	for i := 0; i < 4; i++ {
		result += fmt.Sprint(ip[i])
		if i < 3 {
			result += "."
		}
	}
	return result
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}
```

## Errors

​	Go 使用 `error` 值来解释错误状态。`error` 类型是一个跟 `fmt.Stringer` 相似的内置接口：

```go
type error interface {
  Error() string
}
```

​	函数经常返回一个 `error` 值，并且调用代码来通过判断这个错误是否为 `nil` 来处理错误。**一个值为 nil 的 `error` 意味着成功；一个非 nil 的 `error` 意味着失败。**

```go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}
```

## Exercise: Errors

`Sqrt` should return a non-nil error value when given a negative number, as it doesn't support complex numbers.

Create a new type

```
type ErrNegativeSqrt float64
```

and make it an `error` by giving it a

```
func (e ErrNegativeSqrt) Error() string
```

method such that `ErrNegativeSqrt(-2).Error()` returns `"cannot Sqrt negative number: -2"`.

**Note:** A call to `fmt.Sprint(e)` inside the `Error` method will send the program into an infinite loop. You can avoid this by converting `e` first: `fmt.Sprint(float64(e))`. Why?

Change your `Sqrt` function to return an `ErrNegativeSqrt` value when given a negative number.

```go
package main

import (
	"fmt";
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return x, ErrNegativeSqrt(x)
	}
	return math.Sqrt(x), nil  	// 这里偷懒用包里的函数
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```

## Readers

​	`io` 包制定了 `io.Reader` 接口，它表示数据流的读取端。Go 的基础库里包含了很多这个接口的实现，比如文件，网络连接，压缩，密码等（files, network connections, compressors, ciphers）。

​	`io.Reader` 接口有一个 `Read` 方法： `func (T) Read(b []byte) (n int, err error)`

​	`Read` 用数据填充给定的字节切片，并且返回被填充的字节数和一个错误值。如果读取到流的末端则返回一个 `io.EOF` 错误。

```go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

## Exercise: Readers

Implement a `Reader` type that emits an infinite stream of the ASCII character `'A'`.

​	`'A'` 的 ASCII 值为65。

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.

// 传入一个 byte 类型的切片，将该切片用 65 填充满。
func (myReader MyReader) Read(bytes []byte) (int, error) {
	for i := range bytes {
		bytes[i] = 65
	}
	return len(bytes), nil
}

func main() {
	reader.Validate(MyReader{})
}
```

## Exercise： rot13Reader

A common pattern is an [io.Reader](https://golang.org/pkg/io/#Reader) that wraps another `io.Reader`, modifying the stream in some way.

For example, the [gzip.NewReader](https://golang.org/pkg/compress/gzip/#NewReader) function takes an `io.Reader` (a stream of compressed data) and returns a `*gzip.Reader` that also implements `io.Reader` (a stream of the decompressed data).

Implement a `rot13Reader` that implements `io.Reader` and reads from an `io.Reader`, modifying the stream by applying the [rot13](https://en.wikipedia.org/wiki/ROT13) substitution cipher to all alphabetical characters.

The `rot13Reader` type is provided for you. Make it an `io.Reader` by implementing its `Read` method.

Rot13:

![image-20190823175143760](http://ww2.sinaimg.cn/large/006y8mN6gy1g69rdfu5v3j31560o2juz.jpg)

```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func (rot13 rot13Reader) Read(bytes []byte) (int, error) {
	n, err := rot13.r.Read(bytes)
	var gap int8
	for i, v := range bytes {
		if int8(v) < 91 {
			gap = int8(v)-78
		} else {
			gap = int8(v)-110
		}
		if gap < 0 {
			bytes[i] = byte(int8(bytes[i]) + 13)
		} else {
			bytes[i] = byte(int8(bytes[i]) - 13)
		}
	}
	return n, err
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}
```

## Images

​	图片包定义了 `Image` 接口：

```go
type Image interface {
  ColorModel() color.Model
  Bounds() Rectangle         // Bounds 方法的返回值其实是 image 包里定义的 image.Rectangle
  At(x, y int) color.Color
}
```

​	具体细节可以查看 [image/color package](https://golang.org/pkg/image/color/)

```go
package main

import (
	"fmt"
	"image"
)

func main() {
	m := image.NewRGBA(image.Rect(0, 0, 100, 100))
	fmt.Println(m.Bounds())
	fmt.Println(m.At(0, 0).RGBA())
}

```

## Exercise: Images

Remember the picture generator you wrote earlier? Let's write another one, but this time it will return an implementation of `image.Image` instead of a slice of data.

Define your own `Image` type, implement [the necessary methods](https://golang.org/pkg/image/#Image), and call `pic.ShowImage`.

- `Bounds` should return a `image.Rectangle`, like `image.Rect(0, 0, w, h)`.

  `Bounds` 应该返回一个 `image.Rectangle` ，就像 `image.Rect(0, 0, w, h)`。

- `ColorModel` should return `color.RGBAModel`.

  `ColorModel` 直接返回 `color.RGBAModel`。

- `At` should return a color; the value `v` in the last picture generator corresponds to `color.RGBA{v, v, 255, 255}` in this one.

  `At` 返回一个颜色；最后一个图片生成器的值 `v` 对应为 `color.RGBA{v, v, 255, 255}

