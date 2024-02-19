> 题目来源：知乎

答案：fly

Go 的内存分配借鉴了 Google 的 TCMalloc 分配算法，其核心思想是内存池 + 多级对象管理。内存池主要是预先分配内存，减少向系统申请的频率；多级对象有：mheap、mspan、arenas、mcentral、mcache。它们以 mspan 作为基本分配单位。具体的分配逻辑如下：
当要分配大于 32K 的对象时，从 mheap 分配。
当要分配的对象小于等于 32K 大于 16B 时，从 P 上的 mcache 分配，如果 mcache 没有内存，则从 mcentral 获取，如果 mcentral 也没有，则向 mheap 申请，如果 mheap 也没有，则从操作系统申请内存。
当要分配的对象小于等于 16B 时，从 mcache 上的微型分配器上分配。