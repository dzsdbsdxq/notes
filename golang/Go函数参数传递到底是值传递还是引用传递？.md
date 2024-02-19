先说下结论：

**Go语言中所有的传参都是值传递（传值），都是一个副本，一个拷贝。**

**参数如果是非引用类型（int、string、struct等这些），这样就在函数中就无法修改原内容数据；如果是引用类型（指针、map、slice、chan等这些），这样就可以修改原内容数据。**

**是否可以修改原内容数据，和传值、传引用没有必然的关系。在C++中，传引用肯定是可以修改原内容数据的，在Go语言里，虽然只有传值，但是我们也可以修改原内容数据，因为参数是引用类型**

**引用类型和引用传递是2个概念，切记**！！！

**什么是值传递？**

将实参的值传递给形参，形参是实参的一份拷贝，实参和形参的内存地址不同。函数内对形参值内容的修改，是否会影响实参的值内容，取决于参数是否是引用类型

**什么是引用传递？**

将实参的地址传递给形参，函数内对形参值内容的修改，将会影响实参的值内容。Go语言是没有引用传递的，在C++中，函数参数的传递方式有引用传递。

下面分别针对Go的值类型（int、struct等）、引用类型（指针、slice、map、channel），验证是否是值传递，以及函数内对形参的修改是否会修改原内容数据

**int类型**

形参和实际参数内存地址不一样，证明是指传递；参数是值类型，所以函数内对形参的修改，不会修改原内容数据

```go
package main

import "fmt"

func main() {
var i int64 = 1
fmt.Printf("原始int内存地址是 %p
", &i)
modifyInt(i) // args就是实际参数
fmt.Printf("改动后的值是: %v
", i)
}

func modifyInt(i int64) { //这里定义的args就是形式参数
fmt.Printf("函数里接收到int的内存地址是：%p
", &i)
i = 10
}

原始int内存地址是 0xc0000180b8
函数里接收到int的内存地址是：0xc0000180c0
改动后的值是: 1
```

**指针类型**

形参和实际参数内存地址不一样，证明是指传递，由于形参和实参是指针，指向同一个变量。函数内对指针指向变量的修改，会修改原内容数据

```go
package main

import "fmt"

func main() {
var args int64 = 1                  // int类型变量
p := &args                          // 指针类型变量
fmt.Printf("原始指针的内存地址是 %p
", &p)   // 存放指针类型变量
fmt.Printf("原始指针指向变量的内存地址 %p
", p) // 存放int变量
modifyPointer(p)                    // args就是实际参数
fmt.Printf("改动后的值是: %v
", *p)
}

func modifyPointer(p *int64) { //这里定义的args就是形式参数
fmt.Printf("函数里接收到指针的内存地址是 %p 
", &p)
fmt.Printf("函数里接收到指针指向变量的内存地址 %p
", p)
*p = 10
}

原始指针的内存地址是 0xc000110018
原始指针指向变量的内存地址 0xc00010c008
函数里接收到指针的内存地址是 0xc000110028 
函数里接收到指针指向变量的内存地址 0xc00010c008
改动后的值是: 10
```

**slice类型**

形参和实际参数内存地址一样，不代表是引用类型；下面进行详细说明slice还是值传递，传递的是指针

```go
package main

import "fmt"

func main() {
var s = []int64{1, 2, 3}
// &操作符打印出的地址是无效的，是fmt函数作了特殊处理
fmt.Printf("直接对原始切片取地址%v 
", &s)
// 打印slice的内存地址是可以直接通过%p打印的,不用使用&取地址符转换
fmt.Printf("原始切片的内存地址： %p 
", s)
fmt.Printf("原始切片第一个元素的内存地址： %p 
", &s[0])
modifySlice(s)
fmt.Printf("改动后的值是: %v
", s)
}

func modifySlice(s []int64) {
// &操作符打印出的地址是无效的，是fmt函数作了特殊处理
fmt.Printf("直接对函数里接收到切片取地址%v
", &s)
// 打印slice的内存地址是可以直接通过%p打印的,不用使用&取地址符转换
fmt.Printf("函数里接收到切片的内存地址是 %p 
", s)
fmt.Printf("函数里接收到切片第一个元素的内存地址： %p 
", &s[0])
s[0] = 10
}

直接对原始切片取地址&[1 2 3] 
原始切片的内存地址： 0xc0000b8000 
原始切片第一个元素的内存地址： 0xc0000b8000 
直接对函数里接收到切片取地址&[1 2 3]
函数里接收到切片的内存地址是 0xc0000b8000 
函数里接收到切片第一个元素的内存地址： 0xc0000b8000 
改动后的值是: [10 2 3]
```

`slice`是一个结构体，他的第一个元素是一个指针类型，这个指针指向的是**底层数组的第一个元素**。当参数是`slice`类型的时候，fmt.printf通过%p打印的slice变量的地址其实就是内部存储数组元素的地址，所以打印出来形参和实参内存地址一样。

```go
type slice struct {
array unsafe.Pointer // 指针
len   int
cap   int
}
```

因为slice作为参数时本质是传递的指针，上面证明了指针也是值传递，所以参数为slice也是值传递，指针指向的是同一个变量，函数内对形参的修改，会修改原内容数据

单纯的从slice这个结构体看，我们可以通过modify修改存储元素的内容，但是永远修改不了len和cap，因为他们只是一个拷贝，如果要修改，那就要传递&slice作为参数才可以。

**map类型**

形参和实际参数内存地址不一样，证明是值传递

```go
package main

import "fmt"

func main() {
m := make(map[string]int)
m["age"] = 8

fmt.Printf("原始map的内存地址是：%p
", &m)
modifyMap(m)
fmt.Printf("改动后的值是: %v
", m)
}

func modifyMap(m map[string]int) {
fmt.Printf("函数里接收到map的内存地址是：%p
", &m)
m["age"] = 9
}

原始map的内存地址是：0xc00000e028
函数里接收到map的内存地址是：0xc00000e038
改动后的值是: map[age:9]
```

通过make函数创建的map变量本质是一个`hmap`类型的指针`*hmap`，所以函数内对形参的修改，会修改原内容数据

```go
//src/runtime/map.go
func makemap(t *maptype, hint int, h *hmap) *hmap {
mem, overflow := math.MulUintptr(uintptr(hint), t.bucket.size)
if overflow || mem > maxAlloc {
hint = 0
}

// initialize Hmap
if h == nil {
h = new(hmap)
}
h.hash0 = fastrand()
}
```

**channel类型**

形参和实际参数内存地址不一样，证明是值传递

```go
package main

import (
"fmt"
"time"
)

func main() {
p := make(chan bool)
fmt.Printf("原始chan的内存地址是：%p
", &p)
go func(p chan bool) {
fmt.Printf("函数里接收到chan的内存地址是：%p
", &p)
//模拟耗时
time.Sleep(2 * time.Second)
p <- true
}(p)

select {
case l := <-p:
fmt.Printf("接收到的值是: %v
", l)
}
}

原始chan的内存地址是：0xc00000e028
函数里接收到chan的内存地址是：0xc00000e038
接收到的值是: true
```

通过make函数创建的chan变量本质是一个`hchan`类型的指针`*hchan`，所以函数内对形参的修改，会修改原内容数据

```go
// src/runtime/chan.go
func makechan(t *chantype, size int) *hchan {
elem := t.elem

// compiler checks this but be safe.
if elem.size >= 1<<16 {
throw("makechan: invalid channel element type")
}
if hchanSize%maxAlign != 0 || elem.align > maxAlign {
throw("makechan: bad alignment")
}

mem, overflow := math.MulUintptr(elem.size, uintptr(size))
if overflow || mem > maxAlloc-hchanSize || size < 0 {
panic(plainError("makechan: size out of range"))
}
}
```

**struct类型**

形参和实际参数内存地址不一样，证明是值传递。形参不是引用类型或者指针类型，所以函数内对形参的修改，不会修改原内容数据

```go
package main

import "fmt"

type Person struct {
Name string
Age  int
}

func main() {
per := Person{
Name: "test",
Age:  8,
}
fmt.Printf("原始struct的内存地址是：%p
", &per)
modifyStruct(per)
fmt.Printf("改动后的值是: %v
", per)
}

func modifyStruct(per Person) {
fmt.Printf("函数里接收到struct的内存地址是：%p
", &per)
per.Age = 10
}

原始struct的内存地址是：0xc0000a6018
函数里接收到struct的内存地址是：0xc0000a6030
改动后的值是: {test 8}
```