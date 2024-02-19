> 题目来源：字节跳动

## 答案：ORVR

在go语言中，没有类的概念但是可以给类型（结构体，自定义类型）定义方法，所谓方法就是定义了接收者的函数，接收者定义在func关键字和函数名之间

method是附属在一个给定的类型上，语法和函数的声明语法几乎一样，只是再func后面增加了一个recevier（也就是method所依从的主体）

method 中的 receiver 可以是值传递，也可以是指针。指针的话，就可以直接修改 receiver 中的内容

继承：如果 struct 中的一个匿名段实现了一个 method，那么包含这个匿名段的 struct 也能调用该 method。 

重写：如果 struct 中的一个匿名段实现了一个 method，包含这个匿名段的 struct 是可以重写匿名字段的方法的。

第一条也是最重要的一条，方法是否要修改 receiver？

其次是效率的考虑，如果 receiver 非常大，比如说一个大 struct，使用指针将非常合适。

接下来是一致性，如果该类型的某些方法必须使用指针 receiver，剩下的也要使用指针。不论使用什么类型的 receiver，方法集要一致。

实例和实例指针可以调用值类型和指针类型 receiver 的方法。

如果通过 method express 方式，struct 值只能调用值类型 receiver 的方法，而 struct 指针是能调用值类型和指针类型 receiver 的方法的。

如果 receiver 是 map、func 或 chan，不要使用指针。

如果 receiver 是 slice，并且方法不会重新分配 slice，不要使用指针。

如果 receiver 是包含 sync.Mutex 或其它类似的同步字段的结构体，receiver 必须是指针，以避免复制。

如果 receiver 是大 struct 或 array，receiver 用指针效率会更高。那么，多大是大？假设要把它的所有元素作为参数传递给方法，如果这样会感觉太大，那对 receiver 来说也就太大了。

如果 receiver 是 struct、array 或 slice，并且它的任何元素都是可能发生改变的内容的指针，最好使用指针类型的 receiver，这会使代码可读性更高。

如果 receiver 是一个本来就是值类型的小 array 或 struct，没有可变字段，没有指针，或只是一个简单的基础类型，如 int 或 string，使用值类型的 receiver 更合适。

值类型的 receiver 可以减少可以生成的垃圾量，如果将值传递给值方法，可以使用栈上的副本而不是在堆上进行分配。编译器会尝试避免这种分配，但不会总成功。不要为此原因却不事先分析而选择值类型的 receiver。

最后，如有疑问，请使用指针类型的 receiver。