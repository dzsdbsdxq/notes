> 题目来源：拼多多

## 答案：ORVR

在Go语言中interface是一个非常重要的概念，也是与其它语言相比存在很大特色的地方。interface也是一个Go语言中的一种类型，是一种比较特殊的类型，存在两种interface，一种是带有方法的interface，一种是不带方法的interface。Go语言中的所有变量都可以赋值给空interface变量，实现了interface中定义方法的变量可以赋值给带方法的interface变量，并且可以通过interface直接调用对应的方法，实现了其它面向对象语言的多态的概念。

两种不同的interface在Go语言内部被定义成如下的两种结构体*（源码基于Go的1.9.2版本）*：

```go
// 没有方法的interface

type eface struct {

    _type *_type

    data unsafe.Pointer

}

// 记录着Go语言中某个数据类型的基本特征

type _type struct {

    size    uintptr

    ptrdata  uintptr

    hash    uint32

    tflag   tflag

    align   uint8

    fieldalign uint8

    kind    uint8

    alg    *typeAlg

    gcdata  *byte

    str    nameOff

    ptrToThis typeOff

}

// 有方法的interface

type iface struct {

    tab *itab

    data unsafe.Pointer

}

type itab struct {

    inter *interfacetype

    _type *_type

    link  *itab

    hash  uint32

    bad  bool

    inhash bool

    unused [2]byte

    fun  [1]uintptr

}

// interface数据类型对应的type

type interfacetype struct {

    typ   _type

    pkgpath name

    mhdr  []imethod

}
```

可以看到两种类型的interface在内部实现时都是定义成了一个2个字段的结构体，所以任何一个interface变量都是占用16个byte的内存空间。

**在Go语言中_type这个结构体非常重要，记录着某种数据类型的一些基本特征，比如这个数据类型占用的内存大小（size字段），数据类型的名称（nameOff字段）等等**。每种数据类型都存在一个与之对应的_type结构体（Go语言原生的各种数据类型，用户自定义的结构体，用户自定义的interface等等）。如果是一些比较特殊的数据类型，可能还会对_type结构体进行扩展，记录更多的信息，比如interface类型，就会存在一个interfacetype结构体，除了通用的_type外，还包含了另外两个字段pkgpath和mhdr，后文在对这两个字段的作用进行解析。除此之外还有其它类型的数据结构对应的结构体，比如structtype，chantype，slicetype，有兴趣的可以在$GOROOT/src/runtime/type.go文件中查看。

