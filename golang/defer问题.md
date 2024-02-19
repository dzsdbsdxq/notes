> 题目序号：（5296）
>
> 题目来源：富途证券
>
> 频次：1

答案1：（Kjj）

**defer数据结构：**

```go
type _defer struct {
    sp      uintptr   //函数栈指针
    pc      uintptr   //程序计数器
    fn      *funcval  //函数地址
    link    *_defer   //指向自身结构的指针，用于链接多个defer
}
```

	defer后面一定要接一个函数，所以defer的数据结构更一般函数类似，也有栈指针、程序计数器、函数地址等等。与函数不同的是它含有一个指针，可用于指向另一个defer，每个goroutine数据结构中实际上也有一个defer指针，该指针指向一个defer的[链表](https://so.csdn.net/so/search?q=链表&spm=1001.2101.3001.7020)，每次声明一个defer时就将defer插入到单链表表头，每次执行defer就从单链表表头取出一个defer执行。

![image-20220428112450188](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220428112450188.png)

​	如图所示，新声明的defer（B()）总是添加到链表头部，函数返回前执行defer则是从链表首部依次取出执行，形成一个栈结构。

**defer的功能：**

​	defer用来声明一个延迟函数，可以定义多个延时函数，这些函数会放入到一个栈中，当函数执行到最后时，这些defer语句会按照逆序执行，最后该函数返回。

​	通常用defer来做一些资源的释放，比如关闭io操作。

**defer使用有几个需注意的点：**

以下内容多数来自： 原文作者：Aceld
									转自链接：https://learnku.com/articles/42255#7fa787

**1、defer 的执行顺序**

​	一个函数中使用多个defer时，它们是一个 “栈” 的关系，也就是先进后出，先在后面的defer先执行。

```go
func main() {
   defer func1()
   defer func2()
   defer func3()
}
func func1() {
   fmt.Print("A")
}
func func2() {
   fmt.Print("B")
}
func func3() {
   fmt.Print("C")
}
```

![image-20220428112511476](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220428112511476.png)

执行输出为：C B A

**2、defer 与 return 谁先谁后**

​	根据代码运行情况可以理解为：return 之后的语句先执行，defer 后的语句后执行。不过，defer执行时是可以改变return中的返回值的。

**3、当defer被声明时，其参数就会被实时解析**

```go
func a() {
	i := 0
	defer fmt.Println(i)
	i++
	return
}
```

运行结果是0
这是因为defer后面定义的是一个带变量的函数: fmt.Println(i). 但这个变量(i)在defer被声明的时候，就已经确定值了，这里的变量为整型为值传递，个人理解是为defer后的函数拷贝了一个i变量且=0。 

但若defer后的函数不带变量呢：

```go
func a() {
	i := 0
	defer func() {//defer1
		i++//2+1
		fmt.Println("a defer1:", i)//i=3
	}()
	defer func() {//defer2
		i++//1+1
		fmt.Println("a defer2:", i)//i=2
	}()
	i++//i=1
}
func main() {
	a()
}
```

运行结果：

a defer2: 2

a defer1: 3

无变量传入，即使defer的函数内部有外部定义的变量也不会在defer声明的时候确定值，将在外部函数执行完返回的时候依次执行相应操作（i++）。

**4、有名函数返回值遇见 defer 情况**

​	先 return，再 defer，所以在执行完 return 之后，还要再执行 defer 里的语句，依然可以修改本应该返回的结果。

​	**a.已定义返回值：**

```go
func DeferFunc1(i int) (t int) {
	t = i
	defer func() {
		t += 3
	}()
	return t
}
func main()  {
	fmt.Println(DeferFunc1(1))
}
```

运行结果：4

- 将返回值 t 赋值为传入的 i，此时 t 为 1

- 执行 return 语句将 t 赋值给 t（等于啥也没做）

- 执行 defer 方法，将 t + 3 = 4

- 函数返回 4

  因为 t 的作用域为整个函数所以修改有效。

​	**b.未定义返回值：**

```go
func DeferFunc2(i int) int {
	t := i
	defer func() {
		t += 3
	}()
	return t
}
func main()  {
	fmt.Println(DeferFunc2(1))
}
```

运行结果：1

- 创建变量 t 并赋值为 1
- 执行 return 语句，注意这里是将 t 赋值给返回值，此时返回值为 1（这个返回值并不是 t）
- 执行 defer 方法，将 t + 3 = 4
- 函数返回返回值 1

**5、defer 遇见 panic**
	**a.第一种情况：遇到panic不捕获**

```go
func main() {
	defer fmt.Println("defer1")
	defer fmt.Println("defer2")
	panic("发生异常")
	defer fmt.Println("defer3")
}
```

运行结果：

defer2
defer1
panic: 发生异常

panic后的defer不会入栈（后面的代码运行不到）。

​	**b.第二种情况：defer 遇见 panic，并捕获异常**

```go
func defer_call() {
	defer func() {
		fmt.Println("defer: panic 之前1, 捕获异常")
		if err := recover(); err != nil {
			fmt.Println(err)
		}
	}()
	defer func() { fmt.Println("defer: panic 之前2, 不捕获") }()

	panic("异常内容")  //触发defer出栈

	defer func() { fmt.Println("defer: panic 之后, 永远执行不到") }()
}
func main() {
	defer_call()

	fmt.Println("main 正常结束")
}
```

运行结果：

defer: panic 之前2, 不捕获
defer: panic 之前1, 捕获异常
异常内容
main 正常结束

​	defer 最大的功能是 panic 后依然有效，main函数正常运行，所以 defer 可以保证你的一些资源一定会被关闭，从而避免一些异常出现的问题。

​	**c.第三种情况：defer中含panic**

```go
func main()  {
	defer func() {
		if err := recover(); err != nil{
			fmt.Println(err)
		}else {
			fmt.Println("fatal")
		}
	}()
	defer func() {
		panic("defer panic1")
	}()
	defer func() {
		panic("defer panic2")
	}()
	panic("panic")
}
```

运行结果：defer panic1

​	触发 panic("panic") 后 defer 顺序出栈执行，第一个被执行的 defer 中有 panic("defer panic") 异常语句，这个异常将会覆盖掉 main 中的异常 panic("panic")，"defer panic1"又会覆盖掉"defer panic2"，最后这个异常被栈底的defer捕获到。

**6、 defer 下的函数参数包含子函数**

```go
func function(index int, value int) int {
	fmt.Print(index)
	return index
}

func main() {
	defer function(1, function(3, 0))
	defer function(2, function(4, 0))
}
```

运行结果：3 4 2 1

​	这里，有 4 个函数，他们的 index 序号分别为 1，2，3，4。那么这 4 个函数的先后执行顺序是什么呢？这里面有两个 defer， 所以 defer 一共会压栈两次，先进栈 1，后进栈 2。 那么在压栈 function1 的时候，需要连同函数地址、函数形参一同进栈，那么为了得到 function1 的第二个参数的结果，所以就需要先执行 function3 将第二个参数算出，那么 function3 就被第一个执行。同理压栈 function2，就需要执行 function4 算出 function2 第二个参数的值。然后函数结束，先出栈 fuction2、再出栈 function1.