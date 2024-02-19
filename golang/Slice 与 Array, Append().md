> **题目来源：** 实在太多 
>
> **频次：** 40+

**答案1：**(苦痛律动) +

**Array**

数组（Array）是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因其长度的不可变动，数组在Go中很少直接使用。把一个大数组传递给函数会消耗很多内存。一般采用数组的切片

几种初始化方式

```go
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
arr3 := [3]int{0:3,1:4}
```

**Slice**

Slice是一种数据结构，描述与Slice变量本身分开存储的Array的连续部分。 Slice不是Array。Slice描述了Array的一部分。

slice底层是一个struct

```go
// runtime/slice.go
type slice struct {
    array unsafe.Pointer// 指向数组的指针
    len   int
    cap   int
}
```

创建slice的几种方法

```go
// 直接通过make创建，可以指定len、cap
s4 := make([]int, 5, 10)

// 通过数组/slice 切片生成
var data [10]int
s2 := data[2:8]
s3 := s2[1:3]

// append()
s6 = append(s4,6)

// 直接创建
s1 := []int{1, 2}
```

**append() 底层逻辑**

1. 计算追加后slice的总长度n
2. 如果总长度n大于原cap，则调用growslice func进行扩容（cap最小为n，具体扩容规则见growslice）
3. 对扩容后的slice进行切片，长度为n，获取slice s，用以存储所有的数据
4. 根据不同的数据类型，调用对应的复制方法，将原slice及追加的slice的数据复制到新的slice

**growslice 计算cap的逻辑**

1. 原cap扩容一倍，即doublecap
2. 如果指定cap大于doublecap则使用cap，否则执行如下
3. 如果原数据长度小于1024，则使用doublecap
4. 否则在原cap的基础上每次扩容1/4，直至不小于cap

**1.18更新**

已经不是 doublecap 

```go
// Go 1.18的扩容实现代码如下，et是切片里的元素类型，old是原切片，cap等于原切片的长度+append新增的元素个数。
func growslice(et *_type, old slice, cap int) slice {
  // ...
  newcap := old.cap
  doublecap := newcap + newcap
  if cap > doublecap {
    newcap = cap
  } else {
    const threshold = 256
    if old.cap < threshold {
      newcap = doublecap
    } else {
      // Check 0 < newcap to detect overflow
      // and prevent an infinite loop.
      for 0 < newcap && newcap < cap {
        // Transition from growing 2x for small slices
        // to growing 1.25x for large slices. This formula
        // gives a smooth-ish transition between the two.
        newcap += (newcap + 3*threshold) / 4
      }
      // Set newcap to the requested cap when
      // the newcap calculation overflowed.
      if newcap <= 0 {
        newcap = cap
      }
    }
  }
```

newcap += (newcap + 3*threshold) / 4
newcap是扩容后的容量，先根据原切片的长度、容量和要添加的元素个数确定newcap大小，最后再对newcap做内存对齐得到最后的newcap。

**扩容的整体逻辑（对应上述append()的2）**

1. 按照原slice的cap及指定cap计算扩容后的cap
2. 根据计算出cap申请内存(创建新的数组)
3. 将原slice的数据拷贝到新内存中（新数组）
4. 返回新slice，新slilce指向新数组，len为原slice的len，cap为扩容后的cap

正常我们使用，因slice的长度相对较小，append是扩容使用的是doublecap。
使用append后会产生新的slice，必须重新赋值到原slice上，才能更新原slice的数据。

**典型例题**

```go
data := [10]int{}
slice := data[5:8]
slice = append(slice,9)// slice=? data=?
slice = append(slice,10,11,12)// slice=? data=?
```

```go
//第一次append后结果
slice=[0 0 0 9]
data=[0 0 0 0 0 0 0 0 9 0]
//第二次append后结果
[0 0 0 9 10 11 12]
[0 0 0 0 0 0 0 0 9 0]
```

可以看到第一次append的结果影响到了原data的数据，第二次append的结果并没有影响到了data的数据，这是为什么呢？

未append前，slice的cap是5。第一次append一个元素，未超出cap，因此直接存入数据到数组中。第二次append三个元素，append后的元素长度为7，已大于原slice的cap，因此slice需要扩容，扩容后创建了新的数组，复制了data的数据到新数组内，然后存入append的数据，变动的是新数组，原数组data自然不受影响。

append存在对原数据影响的情况，使用时还是需要注意，如有必要，先copy原数据后再进行slice的操作。

**总结**

1. slice本身并非指针，append追加元素后，意味着底层数组数据（或数组）、len、cap会发生变化，因此append后需要返回新的slice。

2. append在追加元素时，当前cap足够容纳元素，则直接存入数据，否则需要扩容后重新创建新的底层数组，拷贝原数组元素后，再存入追加元素。

3. cap的扩容意味着内存的重新分配，数据的拷贝等操作，为了提高append的效率，若是能预估cap的大小的话，尽量提前声明cap，避免后期的扩容操作。

> **来源：**
>
> 1. https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array/
> 2. https://learnku.com/docs/the-way-to-go/declarations-and-initialization/3612
> 3. https://blog.csdn.net/xz_studying/article/details/106311831 这个讲得十分详细
> 4. https://blog.csdn.net/xz_studying/article/details/106483759 同上
> 5. https://zhuanlan.zhihu.com/p/450057106