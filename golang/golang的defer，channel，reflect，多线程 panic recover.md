> 题目来源：字节跳动

答案：村雨

__defer__

- defer是延迟的意思，在Go里可以放在某个函数或者方法调用的前面，让该函数或方法延迟执行
- 语法：

```go
defer function([parameter_list]) // 延迟执行函数
defer method([parameter_list]) // 延迟执行方法
```

defer本身是在某个函数体内执行，比如在函数A内调用了defer func_name()，只要defer func_name()这行代码被执行到了，那func_name这个函数就会被延迟到函数A return或者panic之前执行。

__注意__：如果是函数是因为调用了os.Exit()而退出，那defer就不会被执行了。参见Go语言里被defer的函数一定会执行么？

```go
defer func_name([parameter_list])
defer package_name.func_name([parameter_list]) // 例如defer fmt.Println("blabla")
```

- 如果在函数内调用了多次defer，那在函数return之前，defer的函数调用满足LIFO原则，先defer的函数后执行，后defer的函数先执行。比如在函数A内先后执行了defer f1(), defer f2(), defer f3()，那函数A return之前，会按照f3(), f2(), f1()的顺序执行，再return。
  **channel**
  __概念和语法__

- 定义：channel是一种类型，零值是nil。

  多个goroutine之间，可以通过channel来通信，一个goroutine可以发送数据到指定channel，其它goroutine可以从这个channel里接收数据。

  channel就像队列，满足FIFO原则，定义channel的时候必须指定channel要传递的元素类型。

- 语法：

  未初始化的channel变量的值是nil，为nil的channel不能用于通信。nil channel收发消息都会阻塞，可能引起死锁。

```go
/*channel_name是变量名，data_type是通道里的数据类型
channel_size是channel通道缓冲区的容量，表示最多可以存放的元素个数，这个参数是可选的，不给就表示没有缓冲区，通过cap()函数可以获取channel的容量
*/
var channel_name chan data_type = make(chan data_type, [channel_size])
```

```go
var ch1 chan int 
var ch2 chan string
var ch3 chan []int
var ch4 chan struct_type // 可以往通道传递结构体变量

ch5 := make(chan int)
ch6 := make(chan string, 100)
ch7 := make(chan []int)
ch8 := make(chan struct_type)
```

__channel三种操作__
channel有3种操作，发送数据，接收数据和关闭channel。发送和接收都是用<-符号

- 发送值到通道：channel <- value

```go
ch := make(chan int)
ch <- 10 // 把10发送到ch里
```

- 从通道接收值：value <- channel

```go
ch := make(chan int)
x := <-ch // 从通道ch里接收值，并赋值给变量x
<-ch // 从通道里接收值，不做其它处理
```

- 关闭通道: close(channel)，关闭nil channel会触发panic: close of nil channel

```go
ch := make(chan int)
close(ch) // 关闭通道
```

__channel缓冲区__
channel默认没有缓冲区，可以在定义channel的时候指定缓冲区容量，也就是缓冲区最多可以存储的元素个数，通过内置函数cap可以获取到channel的容量。

__无缓冲区情况__
channel无缓冲区的时候，往channel发送数据和从channel接收数据都会阻塞。

往channel发送数据的时候，必须有其它goroutine从channel里接收了数据，发送操作才可以成功，发送操作所在的goroutine才能继续往下执行。从channel里接收数据也是同理，必须有其它goroutine往channel里发送了数据，接收操作才可以成功，接收操作所在的goroutine才能继续往下执行。

```go
package main

import "fmt"
import "time"

type Cat struct {
	name string
	age int
}

func fetchChannel(ch chan Cat) {
	value := <- ch
	fmt.Printf("type: %T, value: %v
", value, value)
}


func main() {
	ch := make(chan Cat)
	a := Cat{"yingduan", 1}
	// 启动一个goroutine，用于从ch这个通道里获取数据
	go fetchChannel(ch)
	// 往cha这个通道里发送数据
	ch <- a
	// main这个goroutine在这里等待2秒
	time.Sleep(2*time.Second)
	fmt.Println("end")
}
```

对于上面的例子，有2个点可以思考下

- 如果go fetchChannel(ch)和下面的 ch<-a这2行交换顺序会怎么样？
  Answer: 如果交换了顺序，main函数就会堵塞在ch<-a这一行，因为这个发送是阻塞的，不会往下执行，这个时候没有任何goroutine会从channel接收数据，错误信息如下：

  fatal error: all goroutines are asleep - deadlock!

- 如果没有time.Sleep(2*time.Second)这一行，那程序运行结果会是怎么样？

  Answer: 可能main函数里的end和函数fetchChannel里的print内容都打印，也可能只会打印main函数里的end。因为fetchChannel里的value := <-ch执行之后，main里的ch<-a就不再阻塞，继续往下执行了，所以可能main里最后的fmt.Println比fetchChannel里的fmt.Printf先执行，main执行完之后程序就结束了，所有goroutine自动结束，就不再执行fetchChannel里的fmt.Printf了。main里加上time.Sleep就可以允许fetchChannel这个goroutine有足够的时间执行完成。

__有缓冲区情况__
可以在初始化channel的时候通过make指定channel的缓冲区容量。

ch := make(chan int, 100) // 定义了一个可以缓冲区容量为100的channel
对于有缓冲区的channel，对发送方而言：

如果缓冲区未满，那发送方发送数据到channel缓冲区后，就可以继续往下执行，不用阻塞等待接收方从channel里接收数据。
如果缓冲区已满，那发送方发送数据到channel会阻塞，直到接收方从channel里接收了数据，这样缓冲区才有空间存储发送方发送的数据，发送方所在goroutine才能继续往下执行。
对于接收方而言，在有值可以从channel接收之前，会一直阻塞。

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	// 下面2个发送操作不用阻塞等待接收方接收数据
	ch <- 10
	ch <- 20
	/*
	如果添加下面这行代码，就会一直阻塞，因为缓冲区已满，运行会报错
	fatal error: all goroutines are asleep - deadlock!
	
	ch <- 30
	*/
	
	fmt.Println(<-ch) // 10
	fmt.Println(<-ch) // 20
}
```

__遍历通道channel__
range迭代从channel里不断取数据

```go
package main

import "fmt"
import "time"


func addData(ch chan int) {
	/*
	每3秒往通道ch里发送一次数据
	*/
	size := cap(ch)
	for i:=0; i<size; i++ {
		ch <- i
		time.Sleep(3*time.Second)
	}
	// 数据发送完毕，关闭通道
	close(ch)
}


func main() {
	ch := make(chan int, 10)
	// 开启一个goroutine，用于往通道ch里发送数据
	go addData(ch)

	/* range迭代从通道ch里获取数据
	通道close后，range迭代取完通道里的值后，循环会自动结束
	*/
	for i := range ch {
		fmt.Println(i)
	}
}
```

对于上面的例子，有个点可以思考下：

- 如果删掉close(ch)这一行代码，结果会怎么样？

  Answer: 如果通道没有close，采用range从channel里循环取值，当channel里的值取完后，range会阻塞，如果没有继续往channel里发送值，go运行时会报错

  `fatal error: all goroutines are asleep - deadlock!`

- for死循环不断获取channel里的数据，如果channel的值取完后，继续从channel里获取，会存在2种情况

  - 如果channel已经被close了，继续从channel里获取值会拿到对应channel里数据类型的零值
  - 如果channel没有被close，也不再继续往channel里发送数据，接收方会阻塞报错

```go  
package main

import "fmt"
import "time"


func addData(ch chan int) {
	/*
	每3秒往通道ch里发送一次数据
	*/
	size := cap(ch)
	for i:=0; i<size; i++ {
		ch <- i
		time.Sleep(3*time.Second)
	}
	// 数据发送完毕，关闭通道
	close(ch)
}


func main() {
	ch := make(chan int, 10)
	// 开启一个goroutine，用于往通道ch里发送数据
	go addData(ch)

	/* 
	for循环取完channel里的值后，因为通道close了，再次获取会拿到对应数据类型的零值
	如果通道不close，for循环取完数据后就会阻塞报错
	*/
	for {
		value, ok := <-ch
		if ok {
			fmt.Println(value)
		} else {
			fmt.Println("finish")
			break
		}
	}
}
```

__单向通道__
如果channel作为函数的形参，可以控制限制数据和channel之间的数据流向，控制只能往channel发送数据或者只能从channel接收数据。

不做限制的时候，channel是双向的，既可以往channel写数据，也可以从channel读数据。

- 语法

```go
chan <- int // 只写，只能往channel写数据，不能从channel读数据
<- chan int // 只读，只能从channel读数据，不能往channel写数据
```

- 实例

```go
package main

import "fmt"
import "time"


func write(ch chan<-int) {
	/*
	参数ch是只写channel，不能从channel读数据，否则编译报错
	receive from send-only type chan<- int
	*/
	ch <- 10
}


func read(ch <-chan int) {
	/*
	参数ch是只读channel，不能往channel里写数据，否则编译报错
	send to receive-only type <-chan int
	*/
	fmt.Println(<-ch)
}

func main() {
	ch := make(chan int)
	go write(ch)
	go read(ch)

	// 等待3秒，保证write和read这2个goroutine都可以执行完成
	time.Sleep(3*time.Second)
}
```

__channel注意事项__

- channel被close后，如果再往channel里发送数据，会引发panic
- channel被close后，如果再次close，也会引发panic
- channel被close后，如果channel还有值，接收方可以一直从

- channel里获取值，直到channel里的值都已经取完。
- channel被close后，如果channel里没有值了，接收方继续从

- channel里取值，会得到channel里存的数据类型对应的默认零值，如果一直取值，就一直拿到零值。
  **reflect**
  不同语言的反射模型不尽相同，有些语言还不支持反射。《Go 语言圣经》中是这样定义反射的：

__Go 语言提供了一种机制在运行时更新变量和检查它们的值、调用它们的方法，但是在编译时并不知道这些变量的具体类型，这称为反射机制。__

为什么要用反射

需要反射的 2 个常见场景：

1.有时你需要编写一个函数，但是并不知道传给你的参数类型是什么，可能是没约定好；也可能是传入的类型很多，这些类型并不能统一表示。这时反射就会用的上了。
2.有时候需要根据某些条件决定调用哪个函数，比如根据用户的输入来决定。这时就需要对函数和函数的参数进行反射，在运行期间动态地执行函数。
但是对于反射，还是有几点不太建议使用反射的理由：

1.与反射相关的代码，经常是难以阅读的。在软件工程中，代码可读性也是一个非常重要的指标。
2.Go 语言作为一门静态语言，编码过程中，编译器能提前发现一些类型错误，但是对于反射代码是无能为力的。所以包含反射相关的代码，很可能会运行很久，才会出错，这时候经常是直接 panic，可能会造成严重的后果。
3.反射对性能影响还是比较大的，比正常代码运行速度慢一到两个数量级。所以，对于一个项目中处于运行效率关键位置的代码，尽量避免使用反射特性。
**panic和recover**

- `panic` 能够改变程序的控制流，调用 panic 后会立刻停止执行当前函数的剩余代码，并在当前 Goroutine 中递归执行调用方的 defer；
- `recover` 可以中止 panic 造成的程序崩溃。它是一个只能在 defer 中发挥作用的函数，在其他作用域中调用不会发挥作用；