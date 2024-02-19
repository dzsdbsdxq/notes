> **题目序号：**976
>
> **题目来源**：好未来 
>
> **频次**：1

**答案1：**（peace）

**封装**
封装就是把抽象出的字段和字段的操作封装在一起，数据被保护在内部，程序的其他包只有通过被授权的操作（方法）才能对字段进行操作。
实现如下面代码所示，需要注意的是，在golang内，除了slice、map、channel和显示的指针类型属于引用类型外，其它类型都属于值类型。

- 引用类型作为函数入参传递时，函数对参数的修改会影响到原始调用对象的值；
- 值类型作为函数入参传递时，函数体内会生成调用对象的拷贝，所以修改不会影响原始调用对象。所以在下面GetName中，接收器使用 this *Person 指针对象定义。当传递的是小对象，且不需要更改调用对象时，使用值类型做为接收器；大对象或者需要更改调用对象时使用指针类型作为接收器。

```go
type Person struct {
	name string
	age int
}

func NewPerson() Person {
	return Person{}
}

func (this *Person) SetName(name string) {
	this.name = name
}
func (this *Person) GetName() string {
	return this.name
}

func (this *Person) SetAge(age int) {
	this.age = age
}
func (this *Person) GetAge() string {
	return this.age
}

func main() {
	p := NewPerson()
	p.SetName("xiaofei")
	fmt.Println(p.GetName())
}
```

**继承**
当多个结构体存在相同的属性（字段）和方法时，可以从这些结构体中抽象出一个基结构体A，在A中定义这些相同的属性和方法。其他的结构体不需要重新定义这些属性和方法，只需嵌套一个匿名结构体A即可。
在golang中，如果一个struct嵌套了另一个匿名结构体，那么这个结构体可以直接访问匿名结构体的字段和方法，从而实现继承特性。
同时，一个struct还可以嵌套多个匿名结构体，那么该struct可以直接访问嵌套的匿名结构体的字段和方法，从而实现多重继承。
代码如下：

```go
type Student struct {
	Person
	StuId int
}

func (this *Student) SetId(id int) {
	this.StuId = id
}
func (this *Student) GetId() int {
	return this.StuId
}
func main() {
	stu := oop.Student{}

	stu.SetName("xiaofei")  // 可以直接访问Person的Set、Get方法
	stu.SetAge(22)
	stu.SetId(123)

	fmt.Printf("I am a student，My name is %s, my age is %d, my id is %d", stu.GetName(), stu.GetAge(), stu.GetId)
}
```

**多态**
基类指针可以指向任何派生类的对象，并在运行时绑定最终调用的方法的过程被称为多态。多态是运行时特性，而继承则是编译时特性，也就是说继承关系在编译时就已经确定了，而多态则可以实现运行时的动态绑定。

```go
// 小狗和小鸟都是动物，都会移动和叫，它们共同的方法就可以提炼出来定义为一个抽象的接口。
type Animal interface {
	Move()
	Shout()
}

type Dog struct {
}

func (dog Dog) Move() {
	fmt.Println("I am dog, I moved by 4 legs.")
}
func (dog Dog) Shout() {
	fmt.Println("WANG WANG WANG")
}

type Bird struct {
}

func (bird Bird) Move() {
	fmt.Println("I am bird, I fly with 2 wings")
}
func (bird Bird) Shout() {
	fmt.Println("ji ji ji ")
}

type ShowAnimal struct {
}

func (s ShowAnimal) Show(animal Animal) {
	animal.Move()
	animal.Shout()
}

func main() {
	show := ShowAnimal{}
	dog := Dog{}
	bird := Bird{}

	show.Show(dog)
	show.Show(bird)
}
```
