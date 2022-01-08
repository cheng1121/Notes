# go基础语法

### 包
每个Go程序都是由包构成的,程序从main包开始运行


- 以下代码通过导入路径"fmt"和"math/rand"来使用这两个包
```
///多个导入语句
import "fmt"
import "math/rand"
///分组导入
import (
    "fmt"
    "math/rand"
)

func main() {
	fmt.Println("hello world", rand.Intn(2000))
}

```
按照约定，包名与导入路径的最后一个元素一致，如```“math/rand”```包中的源码均以 ```package rand```语句开始

### 导入
上方代码使用了两种方式
- 有```()```的是分组导入
- 正常导入语句

### 导出名
名字已大写字母开头，那么它就是已导出的，
如果未已大写字母开头则是未导出的。
*未导出的在该包外无法访问*

### 函数

- 函数声明
```
func add(a int, b int) int {
	return a + b
}
```
- func 表示函数的关键字
- add 函数名称
- a,b是参数名称
- a,b后边跟着的int是参数类型
- ()后{}前的int是函数的返回值
- 多个参数类型相同的话可以质保留最后一个类型,如
  ```
  func remove(a, b, c int) int {
	return c
  }

  ```

#### 多值返回
直接上例子
```
func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}

func swap(a string, b string) (string, string) {
	return b, a
}

```

#### 命名返回
Go 的返回值可被命名，它们会被视作定义在函数顶部的变量。

返回值的名称应当具有一定的意义，它可以作为文档使用。

没有参数的 return 语句返回已命名的返回值。也就是 直接 返回。

直接返回语句应当仅用在下面这样的短函数中。在长的函数中它们会影响代码的可读性。
```
//命名返回
func swapInt(a int, b int) (y int, x int) {
	y = b
	x = a
	return
}

func main(){
  y, x := swapInt(10, 8)
	fmt.Println(y, x)
}

```

### 变量
使用关键字var声明变量，类型在后边
- 声明
1. var声明
   - 可以在函数内或者函数外声明变量
```
var s bool
var a, b, c int

```
2. 短变量声明
   - 仅可以在函数内声明
```
a := 30
```

- 初始化

变量声明可以包含初始值，每个变量对应一个。

如果初始化值已存在，则可以省略类型；变量会从初始值中获得类型。
```
var s bool = true

var s, t bool =true,false

```


### 基本数据类型
- bool
- string
- int int8 int16 int32 int64
- uint uint8 uint16 uint32 uint64 uintptr
   - int，uint和uintptr在32位系统上通常是32位宽，在64位系统上则为64位宽。 当你需要一个整数值时应使用 int 类型，除非你有特殊的理由使用固定大小或无符号的整数类型。
- byte //uint8的别名
- rune //int32的别名
- float32 float64
- complex64 complex128

### 零值
没有明确初始值的变量声明会被赋予它们的**零值**  
零值是：
- 数值类型是0
- 布尔类型为false
- 字符串为""(空字符串)

### 类型转换
表达式`T(v)`,将`v`转为类型`T`
```
var x, y int = 3, 4
var f float64 = math.Sqrt(float64(x*x + y*y))
var z uint = uint(f)
fmt.Println(x, y, z)
```
或者更简单的形式：
```
i := 42
f := float64(i)
u := uint(f)
```
### 类型推导
在声明一个变量而不指定其类型时（即使用不带类型的 := 语法或 var = 表达式语法），变量的类型由右值推导得出。  
右值声明了类型时，新变量的类型与其相同：
```
var i int
j := i // j 也是一个 int
```
不过当右边包含未指明类型的数值常量时，新变量的类型就可能是 int, float64 或 complex128 了，这取决于常量的精度：
```
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

### 常量
常量的声明与变量类似，只不过是使用 const 关键字。

常量可以是字符、字符串、布尔值或数值。

常量不能用 := 语法声明。
```
const  a = "哈哈哈"
```
- 数值常量
数值常量是高精度的 值。  
一个未指定类型的常量由上下文来决定其类型
```
const (
	// 将 1 左移 100 位来创建一个非常大的数字
	// 即这个数的二进制是 1 后面跟着 100 个 0
	Big = 1 << 100
	// 再往右移 99 位，即 Small = 1 << 1，或者说 Small = 2
	Small = Big >> 99
)
```

## 循环
go只有一种循环结构：for循环  
基本的`for`循环由三部分组成，使用`;`分隔
- 初始化语句：在第一次迭代前执行
- 条件表达式：在每次迭代前执行
- 后置语句：在每次迭代的末尾执行  

初始化语句通常为一句短变量声明，该变量声明仅在`for`语句的作用域中可见  
一旦表达式的布尔值为`false`，循环迭代就会终止

注意：和 C、Java、JavaScript 之类的语言不同，Go 的 for 语句后面的三个构成部分外没有小括号， 大括号 { } 则是必须的。
```
sum := 0
for i := 0; i < 10; i++ {
	sum += i
}
```
- 初始化语句和后置语句是可选的
```
sum := 1
for ; sum < 10; {
	sum += sum
}
```
- `for`也是go中的“while”循环
可以去掉分号
```
sum := 1
for sum < 100{
    sum +=sum
}
```
- 无限循环
省略循环条件后，就是无限循环
```
for{}
```

## if语句
和`for`类似，表达式外无需小括号`()`，而`{}`是必须的
```
if a < b{
}
```
- if的简短语句  
同 for 一样， if 语句可以在条件表达式前执行一个简单的语句。
该语句声明的变量作用域仅在 if 之内。
```
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	}
	return lim
}
```
- if和else  
在 if 的简短语句中声明的变量同样可以在任何对应的 else 块中使用。
```
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// 这里开始就不能使用 v 了
	return lim
}
```

## switch语句
switch 是编写一连串 if - else 语句的简便方法。它运行第一个值等于条件表达式的 case 语句。

Go 的 switch 语句类似于 C、C++、Java、JavaScript 和 PHP 中的，不过 Go 只运行选定的 case，而非之后所有的 case。 实际上，Go 自动提供了在这些语言中每个 case 后面所需的 break 语句。 除非以 fallthrough 语句结束，否则分支会自动终止。 Go 的另一点重要的不同在于 switch 的 case 无需为常量，且取值不必为整数。
```
switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
```
- switch的case语句从上到下顺次执行，直到匹配成功时停止
- 没有条件的`switch`同`switch true`一样
```
t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
```

## defer语句
defer语句会将函数推迟到外层函数返回之后执行  
推迟调用的函数其参数会立即求值，但直到外层函数返回前该函数都不会被调用。
```
func main() {
	defer fmt.Println("world")
	fmt.Println("hello")
}
```
结果：
```
hello
world
```
- 推迟的函数调用会被压入一个栈中。当外层函数返回时，被推迟的函数会按照后进先出(LIFO)的顺序调用


## 指针
Go拥有指针。指针保存了值的内存地址  
类型`*T`是指向`T`类型的指针。其零值为nil。
```
var p *int
```
`&`操作符会生成指向其操作数的指针
```
i := 42
p = &i
```
* 操作符表示指针指向的底层值
```
//通过指针p读取i
fmt.Print(*p)
//通过指针p设置i
*p = 21
```
这就是通常所说的“间接引用”或"重定向".
与C不同，Go没有指针运算  
[什么时候使用指针？](https://xuchao918.github.io/2019/05/10/Go语言中什么时候使用指针/)

## 结构体
一个结构体`(struck)`就是一组字段`(field)`
```
//声明一个结构体
type User struct{
	age int
	height int
	weight int 
	name string
	gender string
}
```
- type和struck: 关键字
- User：结构体名称
- age int: 字段,使用.访问如:
```
user := User{10, 30, 40, "名字", 1}
user.age = 40
fmt.Println(user.age)
//输出：40
```
#### 结构体指针
结构体字段可以通过结构体指针来访问  
如果我们有一个指向结构体的指针`p`，那么可以通过`(*p).age`来访问其字段age。不过这么写太啰嗦了，所以语言也允许我们使用隐式间接引用，直接写`p.age`就可以
```
user := User{10, 30, 40, "名字", 1}
user.age = 40
p := &user
p.height = 60
fmt.Println(user.age, p.height)
```

#### 结构体文法
结构体文法通过直接列出字段的值来分配一个结构体  
使用`Name:`语法可以仅列出部分字段  
特殊的前缀`&`返回一个指向结构体的指针  
```

var (
	///创建一个User类型的结构体
	u1 = User{10, 30, 40, "名字", 1}
	///其他字段为零值
	u2 = User{age: 20, height: 180}
	//创建一个*User类型的结构体 （指针）
	u3 = &User{age: 25, weight: 60}
)


func main() {
	fmt.Println(u1, u2, u3)
}
//结果：{10 30 40 名字 1} {20 180 0  0} &{25 0 60  0}
```

## 函数值
函数也是值。它们可以像其它值一样传递。  
函数值可以用作函数的参数或返回值
```
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```
## 函数的闭包
go函数可以是一个闭包。闭包是一个函数值，它引用了其 函数体之外的变量。该函数可以访问并赋予其已用的变量的值，换句话说，该函数被这些变量"绑定"在一起  

```
func adder() func(int) int {
	sum := 0
    //返回一个闭包
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}



```