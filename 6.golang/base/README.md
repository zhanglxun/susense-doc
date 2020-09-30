## 基础



### 资料更新中

```html
官网
https://golang.org/

go指南
https://tour.go-zh.org/welcome/1

gorm 手册
https://jasperxu.github.io/gorm-zh/

go-admin
https://github.com/wenjianzhang/go-admin

掘金的两个小册
https://juejin.im/book/6844733830151012359/section/6844733830134235144
https://juejin.im/book/6844733730678898702/section/6844733730704080910

```

### 安装环境

下载最新版面，macos 直接下载 Apple macOS对应的版本

```
https://golang.org/dl/
```



环境变量

```bash
cat ~/.bash_profile

export GOPATH=/Users/jarvischang/go
export GOROOT=/usr/local/go
export GOBIN=/Users/jarvischang/go/bin
export PATH=/usr/local/go/bin:$PATH:$GOBIN

export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
```

常用的go 命令

```bash
查看环境变量
go env

根据要求和实际情况从互联网上下载或更新指定的代码包及其依赖包
go get github路径

用于编译我们指定的源码文件或代码包以及它们的依赖包
go build

go install

go clean

go run

go test 
```



### hello world

```go
package main
import "fmt"

func main() {
	println("hello jarvis!")
}
```



#### 测试Test的支持

!> 编写测试

你打算如何测试这个程序？将你「领域」内的代码和外界（会引起副作用）分离开会更好。`fmt.Println` 会产生副作用（打印到标准输出），我们发送的字符串在自己的领域内。

所以为了更容易测试，我们把这些问题拆分开

```go
package main

import "fmt"

func Hello() string {
    return "Hello, world"
}

func main() {
    fmt.Println(Hello())
}
```

新建一个`hello_test.go`的新文件，来为 `hello`函数编写测试。

```go
package main

import "testing"

func TestHello(t *testing.T) {
    got := Hello()
    want := "Hello, world"

    if got != want {
        t.Errorf("got '%q' want '%q'", got, want)
    }
}
```

编写测试和函数很类似，其中有一些规则

- 程序需要在一个名为 `xxx_test.go` 的文件中编写
- 测试函数的命名必须以单词 `Test` 开始
- 测试函数只接受一个参数 `t *testing.T`
- 被调用的方法必须是大写首字母

类型为 `*testing.T` 的变量 `t` 是你在测试框架中的 hook（钩子），所以当你想让测试失败时可以执行 `t.Fail()` 之类的操作

#### TDD及多需求

根据此说明，可以实验

```html
https://studygolang.gitbook.io/learn-go-with-tests/go-ji-chu/hello-world
```



### 数据类型

#### 布尔类型(bool)

> 值：true 和 false，默认值为 false

```go
package main

import "fmt"

func main() {
    var v1, v2 bool         // 声明变量，默认值为 false
    v1 = true               // 赋值
    v3, v4 := false, true   // 声明并赋值

    fmt.Print("v1:", v1)   // v1 输出 true
    fmt.Print("\nv2:", v2) // v2 没有重新赋值，显示默认值：false
    fmt.Print("\nv3:", v3) // v3 false
    fmt.Print("\nv4:", v4) // v4 true
}
```

#### 数字类型

> 数字类型比较多，默认值都是 0。定义`int`类型时，默认根据系统类型设置取值范围，32位系统与`int32`的值范围相同，64位系统与`int64`的值范围相同。见下表:



| 类型       | 名称                                     | 存储空间   | 值范围                                                       | 数据级别       |
| ---------- | ---------------------------------------- | ---------- | ------------------------------------------------------------ | -------------- |
| uint8      | 无符号8位整形                            | 8-bit      | 0 ~ 255                                                      | 百             |
| uint16     | 无符号16位整形                           | 16-bit     | 0 ~65535                                                     | 6万多          |
| uint32     | 无符号32位整形                           | 32-bit     | 0 ~ 4294967295                                               | 40多亿         |
| uint64     | 无符号64位整形                           | 64-bit     | 0 ~ 18446744073709551615                                     | 大到没概念     |
| int8       | 8位整形                                  | 8-bit      | -128 ~ 127                                                   | 正负百         |
| int16      | 16位整形                                 | 16-bit     | -32768 ~ 32767                                               | 正负3万多      |
| int32      | 32位整形                                 | 32-bit     | -2147483648 ~ 2147483647                                     | 正负20多亿     |
| int64      | 64位整形                                 | 64-bit     | -9223372036854775808 ~ 9223372036854775807                   | 正负大到没概念 |
| int        | 系统决定                                 | 系统决定   | 32位系统为`int32`的值范围，64位系统为`int64`的值范围         |                |
| -          |                                          |            |                                                              |                |
| float32    | 32位浮点数                               | 32-bit     | IEEE-754 1.401298464324817070923729583289916131280e-45 ~ 3.402823466385288598117041834516925440e+38 | 精度6位小数    |
| float64    | 64位浮点数                               | 64-bit     | IEEE-754 4.940656458412465441765687928682213723651e-324 ~ 1.797693134862315708145274237317043567981e+308 | 精度15位小数   |
| -          |                                          |            |                                                              |                |
| complex64  | 复数，含 float32 位实数和 float32 位虚数 | 64-bit     | 实数、虚数的取值范围对应 float32                             |                |
| complex128 | 复数，含 float64 位实数和 float64 位虚数 | 128-bit    | 实数、虚数的取值范围对应 float64                             |                |
| -          |                                          |            |                                                              |                |
| byte       | 字符型，unit8 别名                       | 8-bit      | 表示 UTF-8 字符串的单个字节的值，对应 ASCII 码的字符值       |                |
| rune       | 字符型，int32 别名                       | 32-bit     | 表示 单个 Unicode 字符                                       |                |
| uintptr    | 无符号整型                               | 由系统决定 | 能存放指针地址即可                                           |                |



#### 字符串(string )

> Go 语言默认编码都是 UTF-8。

```go
package main

import "fmt"

func main() {
    var str1 string // 默认值为空字符串 ""
    str1 = `hello world`
    str2 := "你好世界"

    str := str1 + " " + str2 // 字符串连接
    fmt.Println(str1)
    fmt.Println(str2)
    fmt.Println(str) // 输出：hello world 你好世界

    // 遍历字符串
    l := len(str)
    for i := 0; i < l; i++ {
        chr := str[i]
        fmt.Println(i, chr) // 输出字符对应的编码数字
    }
}
```



#### 指针(point)

> 指针其实就是指向一个对象（任何一种类型数据、包括指针本身）的地址值，对指针的操作都会映射到指针所指的对象上。

```go
package main

import (
    "fmt"
)

func main() {
    var p *int // 定义指向int型的指针，默认值为空：nil

    // nil指针不指向任何有效存储地址，操作系统默认不能访问
    //fmt.Printf("%x\n", *p) // 编译报错

    var a int = 10
    p = &a        // 取地址
    add := a + *p // 取值

    fmt.Println(a)   // 输出：10
    fmt.Println(p)   // 输出：0xc0420080b8
    fmt.Println(add) // 输出：20
}
```



#### 数组(array)

> 数组为一组相同数据类型数据的集合，数组定义后大小固定，不能更改，每个元素称为`element`，声明的数组元素默认值都是对应类型的0值。而且数组在Go语言中是一个值类型（value type），**所有值类型变量在赋值和作为参数传递时都会产生一次复制动作**，即对原值的拷贝。

```go
package main

import "fmt"

func main() {
    // 1.声明后赋值
    // var <数组名称> [<数组长度>]<数组元素>
    var arr [2]int   // 数组元素的默认值都是 0
    fmt.Println(arr) // 输出：[0 0]
    arr[0] = 1
    arr[1] = 2
    fmt.Println(arr) // 输出：[1 2]

    // 2.声明并赋值
    // var <数组名称> = [<数组长度>]<数组元素>{元素1,元素2,...}
    var intArr = [2]int{1, 2}
    strArr := [3]string{`aa`, `bb`, `cc`}
    fmt.Println(intArr) // 输出：[1 2]
    fmt.Println(strArr) // 输出：[aa bb cc]

    // 3.声明时不设定大小，赋值后语言本身会计算数组大小
    // var <数组名称> [<数组长度>]<数组元素> = [...]<元素类型>{元素1,元素2,...}
    var arr1 = [...]int{1, 2}
    arr2 := [...]int{1, 2, 3}
    fmt.Println(arr1) // 输出：[1 2]
    fmt.Println(arr2) // 输出：[1 2 3]
    //arr1[2] = 3 // 编译报错，数组大小已设定为2

    // 4.声明时不设定大小，赋值时指定索引
    // var <数组名称> [<数组长度>]<数组元素> = [...]<元素类型>{索引1:元素1,索引2:元素2,...}
    var arr3 = [...]int{1: 22, 0: 11, 2: 33}
    arr4 := [...]string{2: "cc", 1: "bb", 0: "aa"}
    fmt.Println(arr3) // 输出：[11 22 33]
    fmt.Println(arr4) // 输出：[aa bb cc]

    // 遍历数组
    for i := 0; i < len(arr4); i++ {
        v := arr4[i]
        fmt.Printf("i:%d, value:%s\n", i, v)
    }
}
```



#### 切片(slice)

>因为数组的长度定义后不可修改，所以需要切片来处理可变长数组数据。切片可以看作是一个可变长的数组，是一个引用类型。它包含三个数据：1.指向原生数组的指针，2.切片中的元素个数，3.切片已分配的存储空间大小
 **声明一个切片，或从数组中取一段作为切片数据：**



```go
package main

import "fmt"

func main() {
    var sl []int             // 声明一个切片
    sl = append(sl, 1, 2, 3) // 往切片中追加值
    fmt.Println(sl)          // 输出：[1 2 3]

    var arr = [5]int{1, 2, 3, 4, 5} // 初始化一个数组
    var sl1 = arr[0:2]              // 冒号:左边为起始位（包含起始位数据），右边为结束位（不包含结束位数据）；不填则默认为头或尾
    var sl2 = arr[3:]
    var sl3 = arr[:5]

    fmt.Println(sl1) // 输出：[1 2]
    fmt.Println(sl2) // 输出：[4 5]
    fmt.Println(sl3) // 输出：[1 2 3 4 5]

    sl1 = append(sl1, 11, 22) // 追加元素
    fmt.Println(sl1)          // 输出：[1 2 11 22]
}
```

**使用`make`直接创建切片，语法：make([]类型, 大小，预留空间大小)**，make() 函数用于声明`slice`切片、`map`字典、`channel`通道。

```go
package main

import "fmt"

func main() {
    var sl1 = make([]int, 5)          // 定义元素个数为5的切片
    sl2 := make([]int, 5, 10)         // 定义元素个数5的切片，并预留10个元素的存储空间（预留空间不知道有什么用？）
    sl3 := []string{`aa`, `bb`, `cc`} // 直接创建并初始化包含3个元素的数组切片

    fmt.Println(sl1, len(sl1)) // 输出：[0 0 0 0 0] 5
    fmt.Println(sl2, len(sl2)) // 输出：[0 0 0 0 0] 5
    fmt.Println(sl3, len(sl3)) // [aa bb cc] 3

    sl1[1] = 1 // 声明或初始化大小中的数据，可以指定赋值
    sl1[4] = 4
    //sl1[5] = 5 // 编译报错，超出定义大小
    sl1 = append(sl1, 5)       // 可以追加元素
    fmt.Println(sl1, len(sl1)) // 输出：[0 1 0 0 4 5] 6

    sl2[1] = 1
    sl2 = append(sl2, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11)
    fmt.Println(sl2, len(sl2)) // 输出：[0 1 0 0 0 1 2 3 4 5 6 7 8 9 10 11] 16

    // 遍历切片
    for i := 0; i < len(sl2); i++ {
        v := sl2[i]
        fmt.Printf("i: %d, value:%d \n", i, v)
    }
}
```



#### 字典 / 映射(map)

> map 是一种键值对的无序集合，与 slice 类似也是一个引用类型。map 本身其实是个指针，指向内存中的某个空间。
>  声明方式与数组类似，声明方式：**var 变量名 map[key类型]值类型** 或直接使用 make 函数初始化：**make(map[key类型]值类型, 初始空间大小)**
>  其中`key`值可以是任何可以用`==`判断的值类型，对应的值类型没有要求。



```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    // 声明后赋值
    var m map[int]string
    fmt.Println(m) // 输出空的map：map[]
    //m[1] = `aa`    // 向未初始化的map中赋值报错：panic: assignment to entry in nil map

    // 声明并初始化，初始化使用{} 或 make 函数（创建类型并分配空间）
    var m1 = map[string]int{}
    var m2 = make(map[string]int)
    m1[`a`] = 11
    m2[`b`] = 22
    fmt.Println(m1) // 输出：map[a:11]
    fmt.Println(m2) // 输出：map[b:22]

    // 初始化多个值
    var m3 = map[string]string{"a": "aaa", "b": "bbb"}
    m3["c"] = "ccc"
    fmt.Println(m3) // 输出：map[a:aaa b:bbb c:ccc]

    // 删除 map 中的值
    delete(m3, "a") // 删除键 a 对应的值
    fmt.Println(m3) // 输出：map[b:bbb c:ccc]

    // 查找 map 中的元素
    v, ok := m3["b"]
    if ok {
        fmt.Println(ok)
        fmt.Println("m3中b的值为：", v) // 输出：m3中b的值为： bbb
    }
    // 或者
    if v, ok := m3["b"]; ok { // 流程处理后面讲
        fmt.Println("m3中b的值为：", v) // 输出：m3中b的值为： bbb
    }

    fmt.Println(m3["c"]) // 直接取值，输出：ccc

    // map 中的值可以是任意类型
    m4 := make(map[string][5]int)
    m4["a"] = [5]int{1, 2, 3, 4, 5}
    m4["b"] = [5]int{11, 22, 33}
    fmt.Println(m4)                // 输出：map[a:[1 2 3 4 5] b:[11 22 33 0 0]]
    fmt.Println(unsafe.Sizeof(m4)) // 输出：8，为8个字节，map其实是个指针，指向某个内存空间
}
```



#### 通道(channel)

> 说到通道 channel，则必须先了解下 Go 语言的 goroutine 协程（轻量级线程）。channel 就是为 goroutine 间通信提供通道。goroutine 是 Go 语言提供的语言级的协程，是对 CPU 线程和调度器的一套封装。



>  channel 也是类型相关的，一个 channel 只能传递一种类型的值。
>  声明：**var 通道名 chan 通道传递值类型** 或 make 函数初始化：**make(chan 值类型, 初始存储空间大小)**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    var ch1 chan int            // 声明一个通道
    ch1 = make(chan int)        // 未初始化的通道不能存储数据，初始化一个通道
    ch2 := make(chan string, 2) // 声明并初始化一个带缓冲空间的通道

    // 通过匿名函数向通道中写入数据，通过 <- 方式写入
    go func() { ch1 <- 1 }()
    go func() { ch2 <- `a` }()

    v1 := <-ch1 // 从通道中读取数据
    v2 := <-ch2
    fmt.Println(v1) // 输出：1
    fmt.Println(v2) // 输出：a

    // 写入，读取通道数据
    ch3 := make(chan int, 1) // 初始化一个带缓冲空间的通道
    go readFromChannel(ch3)
    go writeToChannel(ch3)

    // 主线程休眠1秒，让出执行权限给子 Go 程，即通过 go 开启的 goroutine，不然主程序会直接结束
    time.Sleep(1 * time.Second)
}

func writeToChannel(ch chan int) {
    for i := 1; i < 10; i++ {
        fmt.Println("写入：", i)
        ch <- i
    }
}

func readFromChannel(ch chan int) {
    for i := 1; i < 10; i++ {
        v := <-ch
        fmt.Println("读取：", v)
    }
}

// ------  输出：--------
1
a
写入： 1
写入： 2
写入： 3
读取： 1
读取： 2
读取： 3
写入： 4
写入： 5
写入： 6
读取： 4
读取： 5
读取： 6
写入： 7
写入： 8
写入： 9
读取： 7
读取： 8
读取： 9
```



#### 结构体(strut)

> 结构体是一种聚合的数据类型，是由零个或多个任意类型的值聚合成的实体。每个值称为结构体的成员。

> 示例

```go
package main

import "fmt"

// 定义一个结构体 person
type person struct {
    name string
    age  int
}

func main() {
    var p person   // 声明一个 person 类型变量 p
    p.name = "max" // 赋值
    p.age = 12
    fmt.Println(p) // 输出：{max 12}

    p1 := person{name: "mike", age: 10} // 直接初始化一个 person
    fmt.Println(p1.name)                // 输出：mike

    p2 := new(person) // new函数分配一个指针，指向 person 类型数据
    p2.name = `张三`
    p2.age = 15
    fmt.Println(*p2) // 输出：{张三 15}
}
```



#### 接口(interface)

> 接口用来定义行为。Go 语言不同于面向对象语言，没有类的概念，也没有传统意义上的继承。Go 语言中的接口，用来定义一个或一组行为，某些对象实现了接口定义的行为，则称这些对象实现了（implement）该接口，类型即为该接口类型。
>  定义接口也是使用 type 关键字，格式为：

```go
// 定义一个接口
type InterfaceName interface {
    FuncName1(paramList) returnType
    FuncName2(paramList) returnType
    ...
}
```

> 示例

```go
package main

import (
    "fmt"
    "strconv"
)

// 定义一个 Person 接口
type Person interface {
    Say(s string) string
    Walk(s string) string
}

// 定义一个 Man 结构体
type Man struct {
    Name string
    Age  int
}

// Man 实现 Say 方法
func (m Man) Say(s string) string {
    return s + ", my name is " + m.Name
}

// Man 实现 Walk 方法，strconv.Itoa() 数字转字符串
func (m Man) Walk(s string) string {
    return "Age: " + strconv.Itoa(m.Age) + " and " + s
}

func main() {
    var m Man       // 声明一个类型为 Man 的变量
    m.Name = "Mike" // 赋值
    m.Age = 30
    fmt.Println(m.Say("hello"))    // 输出：hello, my name is Mike
    fmt.Println(m.Walk("go work")) // 输出：Age: 30 and go work

    jack := Man{Name: "jack", Age: 25} // 初始化一个 Man 类型数据
    fmt.Println(jack.Age)
    fmt.Println(jack.Say("hi")) // 输出：hi, my name is jack
}
```

#### 错误(error)

`error` 类型本身是 Go 语言内部定义好的一个接口，接口里定义了一个 Error() 打印错误信息的方法，源码如下：

```go
type error interface {
    Error() string
}
```

> 示例

```go
package main

import (
    "errors"
    "fmt"
)

func main() {
    // 使用 errors 定制错误信息
    var e error
    e = errors.New("This is a test error")
    fmt.Println(e.Error()) // 输出：This is a test error

    // 使用 fmt.Errorf() 定制错误信息
    err := fmt.Errorf("This is another error")
    fmt.Println(err) // 输出：This is another test error
}
```



### 包、变量和函数





### 指针和错误



### 并发

