> 题目序号：（3241）
>
> 题目来源：腾讯
>
> 频次：1

## 答案1：（One）

**使用defer的优势**

defer一般用于资源的释放和异常的捕捉, 作为Go语言的特性之一.

defer 语句会将其后面跟随的语句进行延迟处理. 意思就是说 跟在defer后面的语言 将会在程序进行最后的return之后再执行.

在 defer 归属的函数即将返回时，将延迟处理的语句按 defer 的逆序进行执行，也就是说，先被 defer 的语句最后被执行，最后被 defer 的语句，最先被执行。

1.1 资源的释放
一般我们写读取文件的代码如下

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		return
	}
	dst, err := os.Create(dstName)
	if err != nil {
		return 
	}
	dst.Close()
	src.Close()
	return
}

```

在程序最开始，os.Open及os.Create打开了两个文件资源描述符，并在最后通过file.Close方法得到释放，在正常情况下，该程序能正常运行，一旦在dstName文件创建过程中出现错误，程序就直接返回，src资源将得不到释放。因此需要在所有错误退出时释放资源，即修改为如下代码才能保证其在异常情况下的正确性。

```go
dst, err := os.Create(dstName)
	if err != nil {
        src.Close()
	}
return
```

即在每个err里面如果发生了异常, 要及时关闭src的资源.
这个问题出现在加锁中也非常常见

```go
l.lock()

// 如果下面发生了异常
// 我们需要在每个err处理块中都加入l.unlock()来解锁
// 不然资源就得不到释放, 就会产生死锁
if err != nil {
	l.unlock()
	return
}
```

但是这样做未免太麻烦了, defer优雅的帮我们解决了这个问题
比如我们可以这样

```go
src, err := os.Open(srcName)
	defer src.Close()
	if err != nil {
		return
	}
	dst, err := os.Create(dstName)
	defer dst.Close()
	if err != nil {
		return 
	}
	------------------------------------------
	l.lock()
	defer l.unlock()
	......
	if err != nil {
		return 
	}
	......

```

这样写的话, 就不需要在每个异常处理块中都加上Close() 或者 unlock()语句了

**defer 常用场景**

通过defer，我们可以在代码中优雅的关闭/清理代码中所使用的变量。defer作为golang清理变量的特性，有其独有且明确的行为。

defer经常和<font color='red'> panic </font>以及 <font color='red'>recover</font> 一起使用，判断是否有异常，进行收尾操作。

```go
package main
 
import "fmt"
 
func main(){
    defer func(){ // 必须要先声明defer，否则不能捕获到panic异常
        fmt.Println("c")
        if err:=recover();err!=nil{
            fmt.Println(err) // 这里的err其实就是panic传入的内容，55
        }
        fmt.Println("d")
    }()
    f()
}
 
func f(){
    fmt.Println("a")
    panic(55)
    fmt.Println("b")
    fmt.Println("f")
}

```

**输出结果：**

```go
a
c
55
d
```

**defer 规则**

1. 当申明defer 时，参数就已经解析了

```go
func a() {
	i := 0
	defer fmt.Println(i)   
	i++
	return
}

//输出 
```

上面我们说过，defer函数会在return之后被调用。那么这段函数执行完之后，是不用应该输出1呢？

读者自行编译看一下，结果输出的是0. why？

这是因为虽然我们在defer后面定义的是一个带变量的函数: fmt.Println(i). 但这个变量(i)在defer被声明的时候，就已经确定其确定的值了

2. defer执行顺序为先进后出

当同时定义了多个defer代码块时，golang安装先定义后执行的顺序依次调用defer。

```go
func b() {
	for i := 0; i < 4; i++ {
		defer fmt.Print(i)
	}
}
// 输出
3
2
1
0

```

3. defer可以读取有名返回值

```go
func c() (i int) {
	defer func() { i++ }()
	return 1
}

```

我们说过defer是在return调用之后才执行的。 这里需要明确的是defer代码块的作用域仍然在函数之内，结合上面的函数也就是说，defer的作用域仍然在c函数之内。因此defer仍然可以读取c函数内的变量