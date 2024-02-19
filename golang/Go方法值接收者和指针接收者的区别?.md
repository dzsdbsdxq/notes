如果方法的接收者是指针类型，无论调用者是对象还是对象指针，修改的都是对象本身，会影响调用者；

如果方法的接收者是值类型，无论调用者是对象还是对象指针，修改的都是对象的副本，不影响调用者；

```
package main

import "fmt"

type Person struct {
age int
}

// 如果实现了接收者是指针类型的方法，会隐含地也实现了接收者是值类型的IncrAge1方法。
// 会修改age的值
func (p *Person) IncrAge1() {
p.age += 1
}

// 如果实现了接收者是值类型的方法，会隐含地也实现了接收者是指针类型的IncrAge2方法。
// 不会修改age的值
func (p Person) IncrAge2() {
p.age += 1
}

// 如果实现了接收者是值类型的方法，会隐含地也实现了接收者是指针类型的GetAge方法。
func (p Person) GetAge() int {
return p.age
}

func main() {
// p1 是值类型
p := Person{age: 10}

// 值类型 调用接收者是指针类型的方法
p.IncrAge1()
fmt.Println(p.GetAge())
// 值类型 调用接收者是值类型的方法
p.IncrAge2()
fmt.Println(p.GetAge())

// ----------------------

// p2 是指针类型
p2 := &Person{age: 20}

// 指针类型 调用接收者是指针类型的方法
p2.IncrAge1()
fmt.Println(p2.GetAge())
// 指针类型 调用接收者是值类型的方法
p2.IncrAge2()
fmt.Println(p2.GetAge())
}

```

上述代码中：

实现了接收者是指针类型的 IncrAge1 函数，不管调用者是值类型还是指针类型，都可以调用IncrAge1方法，并且它的 age 值都改变了。

实现了接收者是指针类型的 IncrAge2 函数，不管调用者是值类型还是指针类型，都可以调用IncrAge2方法，并且它的 age 值都没有被改变。

通常我们使用**指针类型作为方法的接收者的理由**：

- 使用指针类型能够修改调用者的值。
- 使用指针类型可以避免在每次调用方法时复制该值，在值的类型为大型结构体时，这样做会更加高效。