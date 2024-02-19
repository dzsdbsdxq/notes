> 题目序号: 6563
>
> 题目来源: 腾讯
>
> 频次: 1

答案：咸鱼没有早餐

在 golang 中无法使用指针类型对指针进行强制转换

![image-20220501142757244](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220501142757244.png)

但可以借助 `unsafe` 包中的 `unsafe.Pointer` 转换

![image-20220501142724715](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/image-20220501142724715.png)

在 `src/unsafe.go` 中可以看到指针类型说明

```go
// ArbitraryType 与 IntegerType 在此只用于文档描述，实际并不 unsafe 包中的一部分
// 表示任意 go 的表达式
type ArbitraryType int

// 表示任意 integer 类型
type IntegerType int

type Pointer *ArbitraryType
```

对于指针类型 `Pointer` 强调以下四种操作

- 指向任意类型的指针都可以被转化成 Pointer
- Pointer 可以转化成指向任意类型的指针
- uintptr 可以转化成 Pointer
- Pointer 可以转化成 uintptr

> uintptr 在 `src/builtin/builtin.go` 中定义

其后描述了六种指针转换的情形

其一：**Conversion of a *T1 to Pointer to *T2**

转换条件：

- T2 的数据类型不大于 T1
- T1、T2 的内存模型相同

因此对于 `*int` 不能强制转换 `*float64` 可以变化为 `*int` -> `unsafe.Pointer` -> `*float64` 的过程