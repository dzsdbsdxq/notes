> **题目序号：**(1631 2000 3317 3468 96) 
>
> **题目来源:** 腾讯 Shein 小米 好未来
>
> **频次：** 5

**答案1：**（苦痛律动）

**引用类型与值类型**

`引用类型` 变量存储的是一个地址，这个地址存储最终的值。内存通常在堆上分配。通过 GC 回收。包括 指针、slice 切片、管道 channel、接口 interface、map、函数等。

`值类型`是 基本数据类型，int,float,bool,string, 以及数组和 struct 特点：变量直接存储值，内存通常在栈中分配，栈在函数调用后会被释放

对于`引用类型`的变量，我们不光要声明它，还要为它分配内容空间

对于`值类型`的则不需要显示分配内存空间，是因为go会默认帮我们分配好

**new()**

```go
func new(Type) *Type
```

new()对类型进行内存分配,入参为类型,返回为类型的指针，指向分配类型的内存地址

**make()**

```go
func make(t Type, size ...IntegerType) Type
```

make()也是用于内存分配的，但是和new不同，它只用于channel、map以及切片的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

注意，因为这三种类型是引用类型，所以必须得初始化，但是不是置为零值，这个和new是不一样的。

简而言之make()用于初始化slice, map, channel等内置数据结构

>  来源:
>
>  1. https://learnku.com/articles/45323
>
>  2. https://zhuanlan.zhihu.com/p/33041142
>
>  3. https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-make-and-new/ (深入了解)