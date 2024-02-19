> **题目序号：**（67，94，6832，2995，858，1036，1048，1380，1507，1859） 
>
> **题目来源**：好未来、小米、腾讯、小米、滴滴、腾讯、字节跳动、畅天游
>
> **频次**：13

**答案1：**（栾龙生）

Go语言的map使用Hash表和搜索树作为底层实现，一个Hash表可以有多个bucket，而每个bucket保存了map中的一个或一组键值对。

**源码：**`runtime/map.go:hmap`

```go
// go 1.17
type hmap struct {
    count      int            //元素个数，调用len(map)时直接返回
    flags      uint8          //标志map当前状态,正在删除元素、添加元素.....
    B          uint8          //单元(buckets)的对数 B=5表示能容纳32个元素
    noverflow  uint16        //单元(buckets)溢出数量，如果一个单元能存8个key，此时存储了9个，溢出了，就需要再增加一个单元
    hash0      uint32         //哈希种子
    buckets    unsafe.Pointer //指向单元(buckets)数组,大小为2^B，可以为nil
    oldbuckets unsafe.Pointer //扩容的时候，buckets长度会是oldbuckets的两倍
    nevacute   uintptr        //指示扩容进度，小于此buckets迁移完成
    extra      *mapextra      //与gc相关 可选字段
}
```

下图展示了一个拥有4个bucket的map。

![image-20220403085916040](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220403085916040.png)

本例中，hmap.B=2，hmap.buckets数组的长度是4 (2^B^)。元素经过Hash运算后会落到某个bucket中进行存储。

**bucket的数据结构**

数据结构源码：`runtime/map.go/bmap`

```go
// A bucket for a Go map.
type bmap struct {
	tophash [bucketCnt]uint8
}
//实际上编译期间会生成一个新的数据结构
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

bmp也就是bucket，由初始化的结构体可知，里面最多存8个key，每个key落在桶的位置有hash出来的结果的高8位决定。

其中tophash是一个长度为8的整型数组，Hash值相同的键存入当前bucket时会将Hash值的高位存储在该数组中，以便后续匹配。

整体图如下

![image-20220403092625292](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220403092625292.png)

**答案2：**（小小）

**底层结构**

Map 底层是由`hmap`和`bmap`两个结构体实现的。

```go
// A header for a Go map.
type hmap struct {
    // 元素个数，调用 len(map) 时，直接返回此值
	count     int
	flags     uint8
	// buckets 的对数 log_2
	B         uint8
	// overflow 的 bucket 近似数
	noverflow uint16
	// 计算 key 的哈希的时候会传入哈希函数
	hash0     uint32
    // 指向 buckets 数组，大小为 2^B
    // 如果元素个数为0，就为 nil
	buckets    unsafe.Pointer
	// 扩容的时候，buckets 长度会是 oldbuckets 的两倍
	oldbuckets unsafe.Pointer
	// 指示扩容进度，小于此地址的 buckets 迁移完成
	nevacuate  uintptr
	extra *mapextra // optional fields
}
```

其中，`B`是`buckets`数组的长度的对数，也就是说`buckets`数组的长度就是`2^B`。`buckets`里面存储了 `key` 和 `value`，后面会再讲。`buckets`指向`bmap`结构体：

```go
type bmap struct {
	tophash [bucketCnt]uint8
}
```

编译期间`bmap`会变成一个新的结构：

```go
type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr
}
```

`bmap`被称之为“桶”。一个桶里面会最多装 8 个 key，key 经过哈希计算后，哈希结果是“一类”的将会落入到同一个桶中。在桶内，会根据`key`计算出来的`hash`值的高 8 位来决定`key`到底落入桶内的哪个位置。注：一个桶内最多有8个位置。
这也是为什么`map`无法使用`cap()`来求容量的关键原因：`map`的容量是编译器进行**计算后**得出的一个结果，由于桶的存在，`map`在内存中实际存放的大小不一定同`make`出来后的`map`的大小一致。
![](https://cdn.nlark.com/yuque/0/2022/png/21525551/1648976018069-e8483366-3316-48c0-98c9-09a54f94a230.png#clientId=ubf612ea9-8c75-4&crop=0&crop=0&crop=1&crop=1&from=paste&id=u74f7d5e1&margin=%5Bobject%20Object%5D&originHeight=1558&originWidth=2248&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=ueeaf199e-d512-4963-a680-e463e882413&title=)
有一点需要注意：当`map`的`key`和`value`都不是指针，并且`size`都小于 128 字节的情况下，会把 `bmap`标记为不含指针，这样可以避免`gc`时扫描整个`hmap`。尽管如此，但如图所示，`bmap`是有一个`overflow`的字段，该字段是指针类型，这就破坏了`bmap`不含指针的设想，这时会把`overflow`移动到`extra`字段来。

```go
type mapextra struct {
	// overflow[0] contains overflow buckets for hmap.buckets.
	// overflow[1] contains overflow buckets for hmap.oldbuckets.
	overflow [2]*[]*bmap

	// nextOverflow 包含空闲的 overflow bucket，这是预分配的 bucket
	nextOverflow *bmap
}
```

[参考文章](https://qcrao.com/2019/05/22/dive-into-go-map/)