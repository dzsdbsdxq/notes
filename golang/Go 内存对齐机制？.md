### 什么是内存对齐

为了能让CPU可以更快的存取到各个字段，Go编译器会帮你把struct结构体做数据的对齐。**所谓的数据对齐，是指内存地址是所存储数据大小（按字节为单位）的整数倍，以便CPU可以一次将该数据从内存中读取出来。** 编译器通过在结构体的各个字段之间填充一些空白已达到对齐的目的。

### 对齐系数

不同硬件平台占用的大小和对齐值都可能是不一样的，每个特定平台上的编译器都有自己的默认"对齐系数"，32位系统对齐系数是4，64位系统对齐系数是8

不同类型的对齐系数也可能不一样，使用`Go`语言中的`unsafe.Alignof`函数可以返回相应类型的对齐系数，对齐系数都符合`2^n`这个规律，最大也不会超过8

```go
package main

import (
"fmt"
"unsafe"
)

func main() {
fmt.Printf("bool alignof is %d
", unsafe.Alignof(bool(true)))
fmt.Printf("string alignof is %d
", unsafe.Alignof(string("a")))
fmt.Printf("int alignof is %d
", unsafe.Alignof(int(0)))
fmt.Printf("float alignof is %d
", unsafe.Alignof(float64(0)))
fmt.Printf("int32 alignof is %d
", unsafe.Alignof(int32(0)))
fmt.Printf("float32 alignof is %d
", unsafe.Alignof(float32(0)))
}
```

可以查看到各种类型在Mac 64位上的对齐系数如下：

```
bool alignof is 1
string alignof is 8
int alignof is 8
int32 alignof is 4
float32 alignof is 4
float alignof is 8
```

### 优点

1. 提高可移植性，有些`CPU`可以访问任意地址上的任意数据，而有些`CPU`只能在特定地址访问数据，因此不同硬件平台具有差异性，这样的代码就不具有移植性，如果在编译时，将分配的内存进行对齐，这就具有平台可以移植性了

2. 提高内存的访问效率，32位CPU下一次可以从内存中读取32位（4个字节）的数据，64位CPU下一次可以从内存中读取64位（8个字节）的数据，这个长度也称为CPU的字长。CPU一次可以读取1个字长的数据到内存中，如果所需要读取的数据正好跨了1个字长，那就得花两个CPU周期的时间去读取了。因此在内存中存放数据时进行对齐，可以提高内存访问效率。

### 缺点

1. 存在内存空间的浪费，实际上是空间换时间

### 结构体对齐

对齐原则：

1. **结构体变量中成员的偏移量必须是成员大小的整数倍**
2. **整个结构体的地址必须是最大字节的整数倍**（结构体的内存占用是1/4/8/16byte...)

```
package main

import (
"fmt"
"runtime"
"unsafe"
)

type T1 struct {
i16  int16 // 2 byte
bool bool  // 1 byte
}

type T2 struct {
i8  int8  // 1 byte
i64 int64 // 8 byte
i32 int32 // 4 byte
}

type T3 struct {
i8  int8  // 1 byte
i32 int32 // 4 byte
i64 int64 // 8 byte
}

func main() {
fmt.Println(runtime.GOARCH) // amd64

t1 := T1{}
fmt.Println(unsafe.Sizeof(t1)) // 4 bytes

t2 := T2{}
fmt.Println(unsafe.Sizeof(t2)) // 24 bytes

t3 := T3{}
fmt.Println(unsafe.Sizeof(t3)) // 16 bytes
}
```

以T1结构体为例，实际存储数据的只有3字节，但实际用了4字节，浪费了1个字节：

i16并没有直接放在bool的后面，而是在bool中填充了一个空白后，放到了偏移量为2的位置上。如果i16从偏移量为1的位置开始占用2个字节，根据对齐原则2：构体变量中成员的偏移量必须是成员大小的整数倍，套用公式 1 % 2 = 1，就不满足对齐的要求，所以i16从偏移量为2的位置开始

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220502132935164.png)

以T2结构体为例，实际存储数据的只有13字节，但实际用了24字节，浪费了11个字节：

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220502133003644.png)

以T3结构体为例，实际存储数据的只有13字节，但实际用了16字节，浪费了3个字节：

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220502133303337.png)