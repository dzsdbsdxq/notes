> 题目序号：435
>
> 题目来源：百度
>
> 整理人：lws

1. **切片扩容的策略：** 
   1. 首先判断，如果新申请容量大于 2 倍的旧容量，最终容量就是新申请的容
      量 
   2. 否则判断，如果旧切片的长度小于 1024，则最终容量就是旧容量的两倍

   3. 否则判断，如果旧切片长度大于等于 1024，则最终容量从旧容量开始循环增加原来的 1/4, 直到最终容量大于等于新申请的容量

   4. 如果最终容量计算值溢出，则最终容量就是新申请容量

2. **扩容前后slice是否相同**

**情况一：**  

原数组还有容量可以扩容（实际容量没有填充完），这种情况下，扩容以后的数组还是指向原来的数组，对一个切片的操作可能影响多个指针指向相同地址的 Slice。 

**情况二：**  

原来数组的容量已经达到了最大值，再想扩容， Go 默认会先开一片内存区域，把原来的值拷贝过来，然后再执行 append() 操作。这种情况丝毫不影响原数组。 要复制一个 Slice，最好使用 Copy 函数。