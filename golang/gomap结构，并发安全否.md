> 题目序号：（2323）
>
> 题目来源：滴滴
>
> 频次：1

## 答案1：（ORVR）

Go中Map是一个KV对集合。底层使用hash table，用链表来解决冲突 ，出现冲突时，不是每一个Key都申请一个结构通过链表串起来，而是以bmap为最小粒度挂载，一个bmap可以放8个kv。

在哈希函数的选择上，会在程序启动时，检测 cpu 是否支持 aes，如果支持，则使用aes hash，否则使用memhash。

hash函数,有加密型和非加密型。加密型的一般用于加密数据、数字摘要等，典型代表就是md5、sha1、sha256、aes256 这种,非加密型的一般就是查找。

在map的应用场景中，用的是查找。

选择hash函数主要考察的是两点：性能、碰撞概率。

每个map的底层结构是hmap，是有若干个结构为bmap的bucket组成的数组。每个bucket底层都采用链表结构。

```go
// A header for a Go map.

type hmap struct {

// 元素个数，调用 len(map) 时，直接返回此值

    count int

    flags uint8

    // buckets 的对数 log_2

    B uint8

    // overflow 的 bucket 近似数

    noverflow uint16

    // 计算 key 的哈希的时候会传入哈希函数

    hash0 uint32

    // 指向 buckets 数组，大小为 2^B

    // 如果元素个数为0，就为 nil

    buckets unsafe.Pointer

    // 扩容的时候，buckets 长度会是 oldbuckets 的两倍

    oldbuckets unsafe.Pointer

    // 指示扩容进度，小于此地址的 buckets 迁移完成

    nevacuate uintptr

    extra *mapextra // optional fields

}
```

bmap 就是我们常说的“桶”，桶里面会最多装 8 个 key，这些 key之所以会落入同一个桶，是因为它们经过哈希计算后，哈希结果是“一类”的，关于key的定位我们在map的查询和赋值中详细说明。

在桶内，又会根据key计算出来的hash值的高8位来决定 key到底落入桶内的哪个位置（一个桶内最多有8个位置)。

当map的key和value都不是指针，并且 size都小于128字节的情况下，会把bmap标记为不含指针，这样可以避免gc时扫描整个hmap。

但是，我们看bmap其实有一个overflow的字段，是指针类型的，破坏了 bmap 不含指针的设想，这时会把overflow移动到 hmap的extra 字段来。

这样随着哈希表存储的数据逐渐增多，我们会扩容哈希表或者使用额外的桶存储溢出的数据，不会让单个桶中的数据超过 8 个，不过溢出桶只是临时的解决方案，创建过多的溢出桶最终也会导致哈希的扩容。

go的map本身是不支持并发安全的，在查找、赋值、遍历、删除的过程中都会检测写标志，一旦发现写标志置位（等于1），则直接 panic。赋值和删除函数在检测完写标志是复位之后，先将写标志位置位，才会进行之后的操作。