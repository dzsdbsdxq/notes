> **题目序号：**(478)
> **题目来源：** 滴滴
> **频次:** 1

答案：阿纪、

1. **sync.Once源码分析**
   sync.Once是通过一个对象实现的.

   ```go
   type Once struct {
       done unit32
       m Mutex
   }
   ```

   他们分别为标记是否已经执行过的标志(done),以及执行时所用的互斥锁(m)
   除了结构体外，sync.Once还包括了一个公开的方法Do:

   ```go
   func (o *Once) Do(f func()) {
       if atomic.LoadUint32(&o.done) == 0 {
           o.doSlow(f)
       }
   }
   ```

   Once.Do方法的实现非常简单，通过atomic.LoadUint32获取Once实例的done属性值。
   若done值为0时，表示函数f未被调用过或正运行中且未结束，则将调用doSlow方法；
   若done值为1时，表示函数f已经调用且完成，则直接返回。
   这里使用了原子操作方法atomic.LoadUint32而不是直接将o.done进行比较，也是为了避免并发状态下错误地判断执行状态，产生不必要的锁操作带来的时间开销。

   ```go
   func (o *Once) doSlow(f func()) {
       o.m.Lock()
       defer o.m.Unlock()
       if o.done == 0 {
           defer atomic.StoreUint32(&o.done, 1)
           f()
       }
   }
   ```

   Once.doSlow方法的实现使用了传统的互斥锁Mutex操作，在执行时即调用o.m.Lock方法获得锁，然后再继续判断是否已经完成并调用f函数。
   可以看到，在获得锁后还需要对o.done的值进行一次判断，避免了f函数被重复调用。
   最后，在退出doSlow方法时还需要对获取的锁进行释放，若进入到f函数的调用则需要更改o.done属性值。