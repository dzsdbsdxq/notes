> 题目序号：2044
>
> 题目来源：腾讯
>
> 频次：1

## 答案：栾龙生

1. **golang多态**

   golang中采用接口实现多态。golang里面有一个接口类型interface，任何类型只要实现了接口类型，都可以赋值，如果接口类型是空，那么所有的类型都实现了它。

```go
package main

import "fmt"

type Person interface {
	Say()
}

type student struct {
	age int
	grade int
}

type teacher struct {
	age int
}

func (s *student) Say() {
	fmt.Println("I'm a student")
}

func (t *teacher) Say() {
	fmt.Println("I'm a teacher")
}

func  Hello(p Person) {
	p.Say()
}

func main() {
	t := &teacher{}
	s := &student{}
	Hello(t)
	Hello(s)
}
```

2.**golang父类方法重写**

go语言中的继承可以通过结构体来实现。子类重写父类的同名method函数；调用时，是先调用子类的method，如果子类没有，才去调用父类的method。

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

type student struct {
	Person
	grade int
}

func (p *Person) Say() {
	fmt.Printf("Name: %v, Age: %v\n", p.name, p.age)
}

func (s *student) Say() {
	fmt.Printf("Name: %v, Age: %v, Grade: %v\n", s.name, s.age, s.grade)
}

func main() {
	s := &student{
		Person: Person{
			name: "lls",
			age:  24,
		},
		grade: 2,
	}
	s.Say()
}
//output: Name: lls, Age: 24, Grade: 2
```

