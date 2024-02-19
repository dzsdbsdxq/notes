> 题目来源: 字节跳动 
>
> 频次: 1

## 答案：苦痛律动

Go语言中的Map是一种无序的键值对集合。Map可以通过key在O(1)的时间复杂度内进行查询、更改、删除操作，key到value间的映射由哈希函数实现。Go的Map相当于C++的Map，Java的HashMap，Python的Dict。

**map**

**操作**

```go
// 标准声明方法 cap可选
var map_variable map[key_data_type]value_data_type
map_variable = make(map[key_data_type]value_data_type, [cap])
// 或者使用make函数 声明时直接初始化 cap可选
map_variable := make(map[key_data_type]value_data_type [cap])
// 访问 添加
map_variable[key] = value
// 删除
delete(map_variable, key)
```

**源码**

```go
// A header for a Go map.
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed
	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
	extra *mapextra // optional fields
}

// mapextra holds fields that are not present on all maps.
type mapextra struct {
	// If both key and elem do not contain pointers and are inline, then we mark bucket
	// type as containing no pointers. This avoids scanning such maps.
	// However, bmap.overflow is a pointer. In order to keep overflow buckets
	// alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
	// overflow and oldoverflow are only used if key and elem do not contain pointers.
	// overflow contains overflow buckets for hmap.buckets.
	// oldoverflow contains overflow buckets for hmap.oldbuckets.
	// The indirection allows to store a pointer to the slice in hiter.
	overflow    *[]*bmap
	oldoverflow *[]*bmap
	// nextOverflow holds a pointer to a free overflow bucket.
	nextOverflow *bmap
}

// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value
	// for each key in this bucket. If tophash[0] < minTopHash,
	// tophash[0] is a bucket evacuation state instead.
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
// 基于go 1.17
```

**map的一些关键点**

1. Map哈希冲突解决

当两个或者两个以上的元素被放在同一个bucket中，表示已经发生了哈希冲突。由于一个bucket最多储存8个键值对，bucket已满时会创建新的bucket，然后将旧的bucket和新的bucket使用链表连接起来，overflow存的即为新的bucket的地址。

2. Map扩容机制

源码中有负载因子load factor，用途是评估哈希表当前的时间复杂度，其与哈希表当前包含的键值对数、桶数量等相关。如果负载因子越大，则说明空间使用率越高，但产生哈希冲突的可能性更高。而负载因子越小，说明空间使用率低，产生哈希冲突的可能性更低。

loadfactor = key_num / bucket_num (默认6.5)

除了负载因子，Go语言还有溢出桶的概念。结合源码我们发现，每个桶最多可以存储8个元素，如果超过8个，会使用定义相同的溢出桶来存储或者扩容。溢出桶overflow buckets过多的判定有一个固定值的阈值2^15 = 32768。

3. Map的扩容原理
   1. 触发loadfactor的最大值，负载因子已达到当前界限。这种情况表示容量不足，直接扩容为原来大小的二倍
   2. 溢出桶overflow buckets过多，这种情况指的是桶分配了很多，但实际用到的桶的数量很少，即key过于分散，这时扩容指的是内存整理，把相近的key集中到一个桶，给另外的桶腾出空间。

扩容伴随着数据迁移，Go语言的map采用的是渐进式迁移的策略，不是一次性迁移所有的数据，每次最多移动两个桶，每次调用增删改都会查看是否迁移完毕，如果没有迁移完毕就尝试继续搬迁。渐进式迁移能使得数据迁移的均摊时间复杂度降低到O(1)。

4. 切片作为map的值

特别注明，Go语言中可以使用切片作为map的值，这种情况下一个key对应多个value。

**sync.Map**

Go语言中的Map同样不是线程安全的。Go所提供的线程安全的版本是位于sync包下的Map。

如果并发地读写普通的Map，会报错误fatal error: concurrent map read and map write，map内部会对并发操作进行检查并提前发现。

你可以在Map上加锁，但这样性能不高。Go语言在1.9版本提供了效率较高的sync.Map

sync.Map有以下特性：

1. 无需初始化，直接声明即可使用
2. sync.Map不能像map那样读写，而是使用sync.Map提供的方法，Store(key, value)用于存储，Load(key)用于取值，Delete(key)表示删除。
3. 遍历操作需要使用Range()函数配合一个回调函数，通过回调函数返回内部遍历出来的值。range参数中回调函数的返回值在需要继续迭代遍历时，返回true，终止迭代遍历时，返回false。如果成功读取Load()函数返回两个值，key对应的value以及ture表示成功读取；如果map中没有这个key，则返回nil表示失败的false。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var capId sync.Map

	capId.Store("Beijing", 88)
	capId.Store("London", 80)
	capId.Store("Tokyo", "89") // 注意value是字符串

	fmt.Println(capId.Load("Beijing"))
	fmt.Println(capId.Load("Tokyo"))

	fmt.Println()

	value, _ := capId.Load("Beijing")
	fmt.Println(value)

	fmt.Println()

	capId.Delete("Tokyo")
	capId.Range(func(k, v interface{}) bool {
		fmt.Printf("%s对应的id是%d\n", k, v)
		return true
	})
}

// output:

// 88 true
// 89 true
// <nil> false
// 
// 88
// 
// Beijing对应的id是88
// London对应的id是80
```

**注意**

1. sync.Map因为没有初始化过程，无法指定key和value的数据类型，所以干脆就支持了所有的数据类型。它不限制一个map内所有的key和value都必须是相同的类型。
2. sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量，sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能。

[参考文章](