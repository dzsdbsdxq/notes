> 题目序号：1682
>
> 题目来源：字节跳动
>
> 频次：1

**答案1：**（小小）

Go 语言根据接口类型是否包含一组方法将接口类型分成了两类：

* 使用 `runtime.iface` 结构体表示包含方法的接口
* 使用 `runtime.eface` 结构体表示不包含任何方法的 `interface{}` 类型；

**空接口定义**

`runtime.eface` 结构体在 Go 语言中的定义是这样的：

```plain
type eface struct { // 16 字节
	_type *_type
	data  unsafe.Pointer
}
//runtime/type.go
type _type struct {
    size       uintptr
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32
    tflag      tflag
    align      uint8
    fieldalign uint8
    kind       uint8
    alg        *typeAlg
    // gcdata stores the GC type data for the garbage collector.
    // If the KindGCProg bit is set in kind, gcdata is a GC program.
    // Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
    gcdata    *byte
    str       nameOff
    ptrToThis typeOff
}
```

Go
由于 `interface{}` 类型不包含任何方法，所以它的结构也相对来说比较简单，只包含指向底层数据和类型的两个指针。从上述结构我们也能推断出 — Go 语言的任意类型都可以转换成 `interface{}`。我们都知道，runtime._type 是 Go 语言类型的运行时表示。下面是运行时包中的结构体，其中包含了很多类型的元信息，例如：类型的大小、哈希、对其以及种类等。

* `size` 字段存储了类型占用的内存空间，为内存空间的分配提供信息；
* `hash` 字段能够帮助我们快速确定类型是否相等；
* `equal` 字段用于判断当前类型的多个对象是否相等，该字段是为了减少 Go 语言二进制包大小从 `typeAlg` 结构体中迁移过来的[4](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/#fn:4)；
  我们只需要对 `runtime._type` 结构体中的字段有一个大体的概念，不需要详细理解所有字段的作用和意义。 `runtime._type` 是 Go 语言类型的运行时表示。下面是运行时包中的结构体，其中包含了很多类型的元信息，例如：类型的大小、哈希、对齐以及种类等。 

**非空接口定义**

另一个用于表示接口的结构体是 `runtime.iface`，这个结构体中有指向原始数据的指针 `data`，不过更重要的是 `runtime.itab` 类型的 `tab` 字段。

```plain
type iface struct { // 16 字节
	tab  *itab
	data unsafe.Pointer
}
type itab struct {
    inter  *interfacetype
    _type  *_type
    link   *itab
    hash   uint32 // copy of _type.hash. Used for type switches.
    bad    bool   // type does not implement interface
    inhash bool   // has this itab been added to hash?
    unused [2]byte
    fun    [1]uintptr // variable sized
}
```

Go
iface 结构体中有指向原始数据的指针 data，不过更重要的是 runtime.itab 类型的 tab 字段。

`runtime.itab` 结构体是接口类型的核心组成部分，每一个 `runtime.itab` 都占 32 字节，我们可以将其看成接口类型和具体类型的组合，它们分别用 `inter` 和 `_type` 两个字段表示：

除了 `inter` 和 `_type` 两个用于表示类型的字段之外，上述结构体中的另外两个字段也有自己的作用：

* `hash` 是对 `_type.hash` 的拷贝，当我们想将 `interface` 类型转换成具体类型时，可以使用该字段快速判断目标类型和具体类型 `runtime._type` 是否一致；
* `fun` 是一个动态大小的数组，它是一个用于动态派发的虚函数表，存储了一组函数指针。虽然该变量被声明成大小固定的数组，但是在使用时会通过原始指针获取其中的数据，所以 `fun` 数组中保存的元素数量是不确定的；