比较常用的Hash冲突解决方案有链地址法和开放寻址法：

**链地址法**

当哈希冲突发生时，创建新**单元**，并将新单元添加到冲突单元所在链表的尾部。

**开放寻址法**

当哈希冲突发生时，从发生冲突的那个**单元**起，按照一定的次序，从哈希表中寻找一个空闲的单元，然后把发生冲突的元素存入到该单元。**开放寻址法需要的表长度要大于等于所需要存放的元素数量**

开放寻址法有多种方式：线性探测法、平方探测法、随机探测法和双重哈希法。这里以线性探测法来帮助读者理解开放寻址法思想

**线性探测法**

设 `Hash(key)` 表示关键字 `key` 的哈希值， 表示哈希表的槽位数（哈希表的大小）。

线性探测法则可以表示为：

如果 `Hash(x) % M` 已经有数据，则尝试 `(Hash(x) + 1) % M` ;

如果 `(Hash(x) + 1) % M` 也有数据了，则尝试 `(Hash(x) + 2) % M` ;

如果 `(Hash(x) + 2) % M` 也有数据了，则尝试 `(Hash(x) + 3) % M` ;

**两种解决方案比较**

对于链地址法，基于数组 + 链表进行存储，链表节点可以在需要时再创建，不必像开放寻址法那样事先申请好足够内存，因此链地址法对于内存的利用率会比开方寻址法高。链地址法对装载因子的容忍度会更高，并且适合存储大对象、大数据量的哈希表。而且相较于开放寻址法，它更加灵活，支持更多的优化策略，比如可采用红黑树代替链表。但是链地址法需要额外的空间来存储指针。

对于开放寻址法，它只有数组一种数据结构就可完成存储，继承了数组的优点，对CPU缓存友好，易于序列化操作。但是它对内存的利用率不如链地址法，且发生冲突时代价更高。**当数据量明确、装载因子小，适合采用开放寻址法。**

**总结**

在发生哈希冲突时，Python中dict采用的开放寻址法，Java的HashMap采用的是链地址法，而Go map也采用链地址法解决冲突，具体就是**插入key到map中时**，当key定位的桶**填满8个元素后**（这里的单元就是桶，不是元素），将会创建一个溢出桶，并且将溢出桶插入当前桶所在链表尾部。

```
if inserti == nil {
// all current buckets are full, allocate a new one.
newb := h.newoverflow(t, b)
// 创建一个新的溢出桶
inserti = &newb.tophash[0]
insertk = add(unsafe.Pointer(newb), dataOffset)
elem = add(insertk, bucketCnt*uintptr(t.keysize))
}
```