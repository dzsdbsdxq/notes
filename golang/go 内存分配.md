> 题目序号：428
>
> 题目来源：腾讯
>
> 整理人：lws

1. Go在程序启动时，会向操作系统申请一大块内存，之后自行管理。
2. Go内存管理的基本单元是mspan，它由若干个页组成，每种mspan可以分配特定大小的object。
3. mcache, mcentral, mheap是Go内存管理的三大组件，层层递进。mcache管理线程在本地缓存的mspan；mcentral管理全局的mspan供所有线程使用；mheap管理Go的所有动态分配内存。
4. 极小对象会分配在一个object中，以节省资源，使用tiny分配器分配内存；一般小对象通过mspan分配内存；大对象则直接由mheap分配内存。

详细参考：https://juejin.cn/post/6844903795739082760#heading-2