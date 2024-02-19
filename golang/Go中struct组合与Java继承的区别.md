> 题目序号：（4549） 
>
> 题目来源：快手
>
> 频次:  1

答案：Zbbxd

两者都是在编译期实现的。  
Go语言的继承通过匿名组合完成：基类以Struct的方式定义，子类只需要把基类作为成员放在子类的定义中，支持多继承。
Java的继承通过extends关键字完成，不支持多继承。


示例Go代码：

```Go
package main

import "fmt"

type People struct{}

func (p *People) ShowA() {
    fmt.Println("people showA")
    p.ShowB()
}

func (p *People) ShowB() {
    fmt.Println("people showB")
}
type Teacher struct {
    People
}
func (t *Teacher) ShowB() {
    fmt.Println("teacher showB")
}
func main() {
    t := Teacher{}
    t.ShowA()
}

//输出结果：
//people showA
//people showB


```

示例java代码

```java
public class People {
	public void showA() {
		System.out.println("ShowA");
		this.showB();
	}
	public void showB() {
		System.out.println("ShowB");
	}
}

public class Teacher extends People{
	
	public void showB() {
		System.out.println("ShowTeacherB");
	}
	
	public static void main(String[] args){
		Teacher t = new Teacher();
		t.showA();
	}
}

//ShowA  
//ShowTeacherB
```

在编译期，Go的编译器会自动给Teacher生成一个ShowA函数，内部调用一个People实例的showA函数，这样调用顺序是Teacher.ShowA()->People.ShowA()->People.ShowB()。Go的继承在运行时仍然是一个组合关系，Teacher仍然要通过People的实例去调用People的函数。

```go
func (t *Teacher) ShowA() {
    t.People.ShowA()
}

```

Java的编译器则会把People ShowA函数的内容拷贝过来给Teacher生成一个ShowA函数，里面的this在运行时对应的就是Teacher的实例，所以调用顺序是Teacher.showA()->Teacher.showB()。Java在运行时Teacher实例内部不会再有People的实例存在。

```java
public void showA() {
	System.out.println("ShowA");
	this.showB();
}
```

通过以上两段代码发现，go调用ShowB()方法时并没有去子类调用，而是使用父类的ShowB()方法
Go并不支持完全的面向对象，但这并不代表Go不支持面向对象，只是有些地方在使用的时候需要去注意，避免踩坑。