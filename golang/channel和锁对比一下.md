> 题目来源：斗鱼

答案：flare

1.）长度为1有缓存channel可以实现互斥锁

	缓存满时<=>上锁

​	缓存空 <=> 解锁

2.）channel实现结构体包含sync.Mteux

3.) channel的性能比锁代价要大很多