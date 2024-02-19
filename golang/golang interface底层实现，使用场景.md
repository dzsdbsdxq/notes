> 题目序号：（1508）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（jimyag）

interface 底层结构

根据 interface 是否包含有 method，底层实现上用两种 struct 来表示：`iface `和 `eface`。`eface`表示不含 `method `的 `interface `结构，或者叫 `empty interface`。对于 Go 中的大部分数据类型都可以抽象出来` _type` 结构，同时针对不同的类型还会有一些其他信息。

```go
    type eface struct {
        _type *_type
        data  unsafe.Pointer
    }
    
    type _type struct {
        size       uintptr // type size
        ptrdata    uintptr // size of memory prefix holding all pointers
        hash       uint32  // hash of type; avoids computation in hash tables
        tflag      tflag   // extra type information flags
        align      uint8   // alignment of variable with this type
        fieldalign uint8   // alignment of struct field with this type
        kind       uint8   // enumeration for C
        alg        *typeAlg  // algorithm table
        gcdata    *byte    // garbage collection data
        str       nameOff  // string form
        ptrToThis typeOff  // type for pointer to this type, may be zero
    }
    
```

`iface `表示 `non-empty interface` 的底层实现。相比于 `empty interface`，`non-empty` 要包含一些 `method`。`method `的具体实现存放在 `itab.fun` 变量里。

```go
    type iface struct {        tab  *itab        data unsafe.Pointer    }        // layout of Itab known to compilers    // allocated in non-garbage-collected memory    // Needs to be in sync with    // ../cmd/compile/internal/gc/reflect.go:/^func.dumptypestructs.    type itab struct {        inter  *interfacetype        _type  *_type        link   *itab        bad    int32        inhash int32      // has this itab been added to hash?        fun    [1]uintptr // variable sized    }
```

interface的使用要满足2个条件才有意义：

1. 实现了interface的几个struct是相似关系（比如docker和kvm都是虚拟机）、平级的，并且输入输出参数完全一致。（这点是interface的本质，能实现interface的肯定是满足这个条件）
2. 在业务逻辑上，调用实现interface的struct是不确定的，是通过某种方式传递进来，而不是顺序的业务逻辑，比如structA、structB、structC如果是有顺序的则是错误的，下面这样是错误的：

```go
func main() {    var i interfaceX    i = &structA{...}    i.Add()    i = &structB{...}    i.Add()    i = &structC{...}    i.Add()}
```

这样逻辑是正确的：

```go
var i interfaceXswitch opt {case "A":    i = &structA{}case "B":    i = &structB{}case "C":    i = &structC{}}i.Add()i.Del()
```

就是说调用者对于实现interface的struct是根据某个参数（通过API传递过来，或者配置文件传递过来，或者etcd传递过来）来选择某个struct，这种逻辑才适用interface。而如果程序逻辑是被调用者依次执行，则不适用interface。

总结适用interface的调用者业务逻辑（伪代码）：

```go
type I interface {    ...}var i Iswitch opt {    //opt通过某种方式传递进来，而不是写死case "A":    i = &structA{...}case "B":    i = &structB{...}case "C":    i = &structC{...}default:    errors.New("not support")}
```
