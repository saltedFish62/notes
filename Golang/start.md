# begin

## Packages 包

​	每个GO程序都由若干个包组成。程序都会从 `main` 包开始运行。

​	以下程序通过 `import` 路径来使用 `fmt` 和 `math/rand` 包。

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}

```

​	有这样的惯例，包的名字要和引用路径的最后一个元素一样。比如，math/rand 包就包含以 packge rand 开头的文件。

​	**注意：** 这些程序执行的环境是确定的，所以每次你运行上面的示例程序时，`rand.Intn` 会返回相同的数字。

## Imports 引入

​	有以下两种等价的引用方式：

```go
>>>>>>>>>>>>>>>>>>>>
import "fmt"
import "math"
<<<<<<<<<<<<<<<<<<<<
import (
  "fmt"
  "math"
)
```

​	第二种引用方式更优雅。

## Exported names 导出名

 在 Go 里面，一个以大写字母开头的名字是被导出的名字。比如 `Pizza` 是一个被导出的名字，同样 `Pi` 也是从 `math` 包中导出的名字。

​	`pizza` 和 `pi` 首字母不是大写，所以他们不会被导出。当引用一个包，你值能引用他导出的名字。任何“未导出”的名字不允许保外访问。

​	所以，在写包的时候要注意名字首字母的大小写，首字母大写则为导出名，否则包外禁止访问。

## Functions 函数

​	函数可以传入零个或更多参数。在以下例子中，`add` 传入两个 `int` 类型的参数。注意参数名后面的类型 `int` 。

```go
package main

import "fmt"

func add(x int, y int) int {
	return x + y
}

func main() {
	fmt.Println(add(42, 13))
}

```

- `func` 是声明函数的关键字；
- `add` 是函数的名字；
- `(x int, y int)` 是函数的参数表;
- `x int` 中的 `ini` 是 `x` 的类型， `y` 同理；
- 参数表后的 `int` 是函数返回值的类型；
- `main` 函数没有返回值，所以参数表后没有类型。

> 这样声明函数是为了从左到右读方便，文章 [article on Go's declaration syntax](https://blog.golang.org/gos-declaration-syntax) 讨论了c语言和go语言声明上的不同。

​	如果参数表中多个参数都是同一类型，可以省略前几个参数的类型，都用最后一个表示。例如：

```go
x int, y int
// ==>>
x, y int
```

## Multiple results 复合结果

​	函数可以返回任意个结果，例如以下 `swap` 函数就返回两个字符串：

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}

```

## Named return values 命名的返回值

​	go的返回值可以被命名，他们会被当成在函数顶部定义的变量。这些名字应该用于记录返回值。一个没有参数返回的 `return` 语句会返回命名返回值，这叫做 _“naked” return_。

​	*Naked return* 语句只可以用在短函数中，就像以下例子一样。若用于常函数则会大大降低可读性。

```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}

```

## Variables 变量

​	var 语句声明了一系列变量；就如同在函数的参数列表一样，类型会在最后标识。一个 `var` 语句可以在包或者函数的级别中，以下例子则两者都有：

```go
package main

import "fmt"

var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}

```

## Variables with initializers 有初始值的变量

​	一个变量声明可以包含初始值，每个变量一个。如果存在初始值，就可以省略类型；变量将会以初始值的类型做为自己的类型。

```go
package main

import "fmt"

var i, j int = 1, 2

func main() {
	var c, python, java = true, false, "no!"
	fmt.Println(i, j, c, python, java)
}

```

## Short variable declarations 短变量声明

​	在函数里，`:=` 短分配语句可以用来代替 `var` 语句并且给变量一个默认类型。在函数外，每个语句都由关键字（`var` , `func` , 等）开始，所以 `:=` 结构是不允许的。

```go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

```

## Basic types 基础类型

> - bool
>
> - string
>
> - int int8 int16 int32 int64
>
> - uint uint8 uint16 uint32 uint64 uintptr
>
> - byte // alias for uint8
>
> - rune // alias for int32
>
>   ​		 // represents a Unicode code point
>
> - float32 float64
>
> - complex64 complex128

​	`int` , `uint` 和 `uintptr` 类型通常来讲，在32位系统是32位宽，在64位系统是64位宽的。除非有特殊的原因使用一个固定位宽或者正整数类型，否则就用 `int` 类型。

```go
package main

import (
	"fmt"
	"math/cmplx"
)

var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

func main() {
	fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
	fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
	fmt.Printf("Type: %T Value: %v\n", z, z)
}
```

## Zero values 零值

​	如果变量在声明时没有一个确切的初始值，那么它将会被赋予零值。

​	零值包括：

- 数字类型的 0
- 布尔类型的 false
- 字符串类型的 “” （即空字符串）

## Type conversions 类型转换

​	有表达式 `T(v)` ，该表达式将值 `v` 的类型转换成 `T` 。

​	数字类型的转换有如下示例：

```go
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
// or more simply:
i := 42
f := float64(i)
u := uint(f)
```

​	与c不同的是，在go中，不同类型的赋值需要一个显性转换。否则将会报错。

## Type inference 类型引用

​	当不使用特定的一个类型来声明变量时，变量的类型将会参考右手边的值。比如：

```go
var i int
j := i // j is an int
```

## Constants 常量

​	常量的声明和变量相似，但使用 `const` 关键字。常量可以是字符，字符串，布尔值，或者数字。常量不能使用 `:=` 句法。

## Numeric Constants 数字常量

​	数字常量是高精度值。一个没有明确声明类型的常量根据它的上下文来确定类型。

```go
package main

import "fmt"

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
	fmt.Println(needInt(Big))
}

```

## For 循环

​	Go 只有一个循环结构，`for` 循环。大致的结构与c相似，但是省去了小括号，并且大括号不可省略。例如：

```go
func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

## For is Go's "while" For是Go里的 “while”

​	例如：

```go
func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

## Forever "死循环"

```go
func main() {
  for {
  }
}
```

## If

​	Go 的 If 语句没有小括号，但大括号是必须的。

​	跟 For 类似，`if` 语句可以在条件之前先执行一个短语句。该短语句声明的变量值在这个域中生效直到 `if` 结束。

```go
package main

import (
	"fmt"
	"math"
)

func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}

func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
```

## Switch

​	语法与c相似，但不需要写 `break;` 就会在执行符合条件的语句之后跳出 `switch` ，而不会继续执行后续的语句。

​	case 后的条件语句可以是一个可执行的求值语句。

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Print("Go runs on ")
	switch os := "darwin"; os {
	case "darwin":
		fmt.Println("OS X.")
		fmt.Println("next line")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
>>>>>>>>>>>>>>>>>>>>>>> output:
Go runs on OS X.
next line
<<<<<<<<<<<<<<<<<<<<<<<
```

​	switch 还可以不接条件，当 case 后的语句返回值为 true 时执行。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}
```

## Defer 推迟

​	一个推迟语句会将执行推迟到调用者函数返回后。推迟的调度会把参数先运算出来，但函数的调用会到调用者返回才执行。推迟的函数会被推进一个栈里，在调用者函数返回时，以先进后出的顺序执行。

```go
package main

import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
>>>>>>>>>>>>>>>>>>>>> output:
done
9
8
7
6
5
4
3
2
1
0
<<<<<<<<<<<<<<<<<<<<<
```

## Pointers 指针

​	指针保存值的内存地址。值 `T` 的指针的类型是 `T` 。它的值是 `nil` 。

​	操作符&会产生一个指针指向它的操作数。

```go
i := 42
p = &i
```

​	操作符 `*` 表示指针的值。

```go
fmt.Println(*p)
*p = 21
```

​	与c不同的是，Go没有指针算术。

## Structs 结构

​	一个结构体是字段的集合。访问结构体里的字段使用点表示法，同样指针也可以访问结构体里的字段。

```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	v.X = 4						// 点表示法访问结构体字段
	fmt.Println(v.X)
  
  p := &v						// 指针访问结构体字段
  p.Y = 1e9         // 原本该写为 (*p).Y 为了方便系统允许我们简写成如此
  fmt.Println(v.Y)
}
```

## Arrays 数组

​	类型 `[n]T` 是一个有 `n` 个 `T` 类型的值的数组。与c不同的是，[] 放在了类型的前面。

```go
// in C:
int arr[10];

// in Go:
var arr [10]int
```

​	数组的长度是类型的一部分，就是说数组的类型实际上应该描述成 `3位的int类型`，因此数组的长度不可变。

## Slices 剪切

​	一个数组有固定的大小，它的元素可以组成动态大小的、灵活的视图，即一个切片。实际上，切片用的比数组更多。`[]T` 类型是一个元素类型为 `T` 的切片。一个切片的语法为： `a[low : high]` 。实际上，切片只会取下标为 `low` 到 `high-1` 的元素。

```go
package main

import "fmt"

func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13}

	var s []int = primes[1:4]
	fmt.Println(s)
}
>>>>>>>>>>>>>>>> output:
[3, 5, 7]
<<<<<<<<<<<<<<<<
```

​	Slices其实像是数组的引用。slice不会存储任何数据，他只描述一个数组的一截。如果原数组的数据发生了变化，那么所有引用这一截的slice都可以看到更改。

## Slice length and capacity

​	slice既有长度也有容量：

- 长度：slice包含的元素个数；
- 容量：从slice的首个元素算起到slice基于的数组的最后一个元素的个数。

```go
package main

import "fmt"

func main() {
	s := []int{2, 3, 5, 7, 11, 13}
  b := s[0:]
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[:4]
	printSlice(s)

	// Drop its first two values.
	s = s[2:]
  s[0] = 99
	printSlice(s)
  printSlice(b)

	s = s[ : 4]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
>>>>>>>>>>>>>>>>>>>>> output: 
len=6 cap=6 [2 3 5 7 11 13]
len=0 cap=6 []
len=4 cap=6 [2 3 5 7]
len=2 cap=4 [99 7]
len=6 cap=6 [2 3 99 7 11 13]
len=4 cap=4 [99 7 11 13]
<<<<<<<<<<<<<<<<<<<<<
```

​	第6行代码执行时，先在堆中创建了一个 `[]int{2, 3, 5, 7, 11, 13}` 的切片 T，然后再将 s 指向它。此时s是一个指向 T 的切片。

​	第10行执行时，s 则缩小len，取 T 的前0个元素，也就是空切片，但此时的容量是6，因为基础数组（underlying array）是 T 。

​	第14行执行时，s 扩张了len，取 T 的前4个元素，也就是下标为 0， 1， 2，3 的元素。所以可以看到此时 s 的 len = 4， cap = 6。

​	第18行执行时，s drop了前两个元素，于是切片 s 的头就往后移动了两个元素，之后重新切片就没办法再将头两个元素切回来了，正如第24行的切片结果。而不管 s 如何切片，最终都是引用 T，所以当对 s 进行修改的时候，都将反映到 T 上，而所有基于 T 的切片也都会反映出对 T 的修改。b 的第三个元素会发生修改就是因为这个原因。

## Nil slices

​	slice 的零值是 `nil`。一个 `nil slice` 的长度和容量都是0，并且没有基础数组。可以用 `var s []int` 来声明一个 `nil slice` 。

## Creating a slice with make

​	slice 可以由内置函数 `make` 创建；这是创建动态大小数组的方式。`make` 函数申请了一个置零的数组并且返回了这个数组的切片：

```go
a := make([]int, 5)  // len(a) = 5
```

​	要确定容量，再传第三个参数到 `make` 函数中：

```go
b := make([]int, 0, 5)  // len(b)=0, cap(b)=5

b = b[:cap(b)]  // len(b)=5, cap(b)=5
b = b[1: ]      // len(b)=4, cap(b)=4
```

## Slices of slices

​	slice 可以包含任何类型，包括其他 slice。其实有点像任意类型的多维数组。

```go
// []string 类型的切片 的切片
board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}

// 访问时：
board[0][0] = "_"
```

## Appending to a slice

​	给一个 slice 追加新元素是常见操作，所以Go提供了一个内置的 `append` 函数。它的原型为： `func append(s []T, vs ... T) []T`。可以看到他的参数表，首个参数为原切片，后续为要追加的元素，最终返回一个新的切片。如果 s 的基础数组太小而不能适用所有要追加的值，那么一个更大的数组就会被分配，返回的分片也会指向新分配的数组。

```go
package main

import "fmt"

func main() {
	var s []int
	printSlice(s)

	// append works on nil slices.
	s = append(s, 0)
	printSlice(s)

	// The slice grows as needed.
	s = append(s, 1)
	printSlice(s)

	// We can add more than one element at a time.
	s = append(s, 2, 3, 4)
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
>>>>>>>>>>>>>>>> output:
len=0 cap=0 []
len=1 cap=1 [0]
len=2 cap=2 [0 1]
len=5 cap=8 [0 1 2 3 4]
<<<<<<<<<<<<<<<<
```

> 官网的教程跑出来的结果跟我本地跑出来的结果有出入，以本地输出为准。

- 当cap<1024 扩容则成倍扩容，也就是原本的cap = oldCap + oldCap
- 当cap>1024 扩容则按1.25倍来扩容，也就是cap = oldCap*1.25

## Range

​	for 循环可以用 `range` 的形式对 slice 或者 map 来迭代。当使用 range 迭代一个切片时，每个迭代将会返回两个值，第一个是索引，第二个是该索引对应元素的拷贝。也可以用 _ 作为占位符来忽略索引或者拷贝。例如：

```go
for i, _ := range pow
for _, value := range pow
```

​	如果只需要索引，可以省略第二个元素： `for i := range pow`

```go
package main

import "fmt"

func main() {
	pow := make([]int, 10)
	for i := range pow {
		pow[i] = 1 << uint(i) // == 2**i
	}
	for _, value := range pow {
		fmt.Printf("%d\n", value)
	}
}
```

## Exercise: Silces

[go to the exercise](https://tour.golang.org/moretypes/18)

Implement `Pic`. It should return a slice of length `dy`, each element of which is a slice of `dx` 8-bit unsigned integers. When you run the program, it will display your picture, interpreting the integers as grayscale (well, bluescale) values.

The choice of image is up to you. Interesting functions include `(x+y)/2`, `x*y`, and `x^y`.

(You need to use a loop to allocate each `[]uint8` inside the `[][]uint8`.)

(Use `uint8(intValue)` to convert between types.)

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
	pic := make([][]uint8, dy)
	for y := range pic {
		pic[y] = make([]uint8, dx)
		
		for x := range pic[y] {
			pic[y][x] = uint8(x+y)
		}
	}
	return pic
}

func main() {
	pic.Show(Pic)
}
```

## Maps 映射

​	map 将键映射到值。一个 map 的零值为 `nil`。一个 `nil map` 没有键，也不能增加键。`make` 函数返回可用的、已初始化的、给定类型的 map。要使用map就要先声明一个map：

```go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex	// 声明

func main() {
	m = make(map[string]Vertex)  // 分配空间
	m["Bell Labs"] = Vertex{     // 定义 "Bell Labs"
		40.68433, -74.39967,
	}
	m["Somewhere"] = Vertex{     // 定义 "Somewhere"
		123, -123,
	}
	fmt.Println(m["Bell Labs"])
	fmt.Println(m["Somewhere"])
}
>>>>>>>>>>>>>>>>>>>>> output:
{40.68433 -74.39967}
{123 -123}
<<<<<<<<<<<<<<<<<<<<<
```

​	map 也可以使用字面量来声明和定义：

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

// 如果高层级类型只是一个类型名，就可以省略字面量中元素的类型名
// 以上定义就成了
var m = map[string]Vertex {
  "Bell Labs": {40.68433, -74.39967},
  "Google": 	 {37.42202, -122.08408},
}
```

## Mutating Maps

- 插入(insert):  `m[key] = elem`
- 查询(retrieve):  `x := m[key]`
- 删除(delete): `delete(m, key)`
- 测试(test): `v, ok := m[key]`, 若 key 存在，则 ok 为true，且返回值 v ，否则 ok 为false，且 v 为值的类型的初始化值。

## Exercise: Maps

Implement `WordCount`. It should return a map of the counts of each “word” in the string `s`. The `wc.Test` function runs a test suite against the provided function and prints success or failure.

You might find [strings.Fields](https://golang.org/pkg/strings/#Fields) helpful.

```go
package main

import (
	"golang.org/x/tour/wc";
	"strings"
)

func WordCount(s string) map[string]int {
	arr := strings.Fields(s)
	resultMap := make(map[string]int)
	for _, v := range arr {
		if _, ok := resultMap[v]; ok {
			resultMap[v] += 1
		} else {
			resultMap[v] = 1
		}
	}
	return resultMap
}

func main() {
	wc.Test(WordCount)
}
```

