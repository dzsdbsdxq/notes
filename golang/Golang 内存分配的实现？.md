> **题目序号：**（2372，2707，**4871**，5118，5243，5713，6408，5429）
> **题目来源：** 小米、shopee、腾讯、阿里、知乎、地平线
> **频次:**  9

## 答案：小强

Golang内存分配和TCMalloc差不多，都是把内存提前划分成不同大小的块，其核心思想是把内存分为多级管理，从而降低锁的粒度。

**先了解下内存管理每一级的概念：**
**mspan**
mspan跟tcmalloc中的span相似，它是golang内存管理中的基本单位，也是由页组成的，每个页大小为8KB，与tcmalloc中span组成的默认基本内存单位页大小相同。mspan里面按照8*2n大小(8b，16b，32b .... )，每一个mspan又分为多个object。


**mcache**
mcache跟tcmalloc中的ThreadCache相似，ThreadCache为每个线程的cache，同理，mcache可以为golang中每个Processor提供内存cache使用，每一个mcache的组成单位也是mspan。

**mcentral**
mcentral跟tcmalloc中的CentralCache相似，当mcache中空间不够用，可以向mcentral申请内存。可以理解为mcentral为mcache的一个“缓存库”，供mcaceh使用。它的内存组成单位也是mspan。

mcentral里有两个双向链表，一个链表表示还有空闲的mspan待分配，一个表示链表里的mspan都被分配了。

**mheap**
mheap跟tcmalloc中的PageHeap相似，负责大内存的分配。当mcentral内存不够时，可以向mheap申请。那mheap没有内存资源呢?跟tcmalloc一样，向OS操作系统申请。

还有，大于32KB的内存，也是直接向mheap申请。

golang 分配内存具体过程如下：

1. 程序启动时申请一大块内存，并划分成spans、bitmap、arena区域
2. arena区域按页划分成一个个小块
3. span管理一个或多个页
4. mcentral管理多个span供线程申请使用
5. mcache作为线程私有资源，资源来源于mcentral