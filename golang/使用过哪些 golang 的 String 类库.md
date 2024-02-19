> 题目序号：6675
> 题目来源：小米
> 频次：15

答案：苏木

**string.builder**

`Go` 语言提供了一个专门操作字符串的库 `strings`，可以用于字符串查找、替换、比较等。

使用 `strings.Builder` 可以进行字符串拼接，提供了 `writeString` 方法拼接字符串，使用方式如下：

```go
var builder strings.Builder
builder.WriteString("asong")
builder.String()
```

`strings.builder` 的实现原理很简单，结构如下：

```go
type Builder struct {
    addr *Builder // of receiver, to detect copies by value
    buf  []byte // 1
}
```

`addr` 字段主要是做 `copycheck`，`buf` 字段是一个 `byte` 类型的切片，这个就是用来存放字符串内容的，提供的 `writeString()` 方法就是向切片 `buf` 中追加数据：

```go
func (b *Builder) WriteString(s string) (int, error) {
 b.copyCheck()
 b.buf = append(b.buf, s...)
 return len(s), nil
}
```

提供的 `String` 方法就是将 `[]byte` 转换为 `string` 类型，这里为了避免内存拷贝的问题，使用了强制转换来避免内存拷贝：

```go
func (b *Builder) String() string {
 return *(*string)(unsafe.Pointer(&b.buf))
}
```

**bytes.Buffer**

因为 `string` 类型底层就是一个 `byte` 数组，所以我们就可以 `Go` 语言的 `bytes.Buffer` 进行字符串拼接。`bytes.Buffer` 是一个一个缓冲 `byte` 类型的缓冲器，这个缓冲器里存放着都是 `byte`。使用方式如下：

```go
buf := new(bytes.Buffer)
buf.WriteString("asong")
buf.String()
```

`bytes.buffer` 底层也是一个 `[]byte` 切片，结构体如下：

```go
type Buffer struct {
 buf      []byte // contents are the bytes buf[off : len(buf)]
 off      int    // read at &buf[off], write at &buf[len(buf)]
 lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

因为 `bytes.Buffer` 可以持续向 `Buffer` 尾部写入数据，从 `Buffer` 头部读取数据，所以 `off` 字段用来记录读取位置，再利用切片的 `cap` 特性来知道写入位置，这个不是本次的重点，重点看一下 `WriteString` 方法是如何拼接字符串的：

```go
func (b *Buffer) WriteString(s string) (n int, err error) {
 b.lastRead = opInvalid
 m, ok := b.tryGrowByReslice(len(s))
 if !ok {
  m = b.grow(len(s))
 }
 return copy(b.buf[m:], s), nil
}
```

切片在创建时并不会申请内存块，只有在往里写数据时才会申请，首次申请的大小即为写入数据的大小。如果写入的数据小于64字节，则按64字节申请。采用 `动态扩展slice` 的机制，字符串追加采用 `copy` 的方式将追加的部分拷贝到尾部，copy是内置的拷贝函数，可以减少内存分配。

但是在将 `[]byte` 转换为 `string` 类型依旧使用了标准类型，所以会发生内存分配：

```go
func (b *Buffer) String() string {
 if b == nil {
  // Special case, useful in debugging.
  return "<nil>"
 }
 return string(b.buf[b.off:])
}
```

**参考资料**
Go 字符串拼接的6种最快方式--strings.builder 
https://www.cnblogs.com/cheyunhua/p/15769717.html

Go标准库(一):strings包(字符串操作)使用 
https://zhuanlan.zhihu.com/p/375953005

Go语言中字符串转换包strconv 
https://zhuanlan.zhihu.com/p/160814669