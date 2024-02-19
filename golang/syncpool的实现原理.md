> **题目序号：**361
>
> **题目来源：**
>
> **频次：**1

 **答案1：**（趁醉独饮痛）

**Pool是什么：**

Go标准库中提供的一个通用的Pool数据结构，可以使用它创建池化的对象。sync.Pool数据类型的对象用来保存一组可独立访问的临时对象，注意它是临时的，也就是说sync.Pool中保存的对象会在将来的某个时间从sync.Pool中移除掉，如果也没有被其他对象引用的话，该对象会被垃圾回收掉。

Go是一个自带GC的语言，使用起来没有心智负担，我们想使用一个对象就创建一个对象，想使用多个就创建多个，不用关心对象的释放问题。因为有GC程序会帮助我们自动将不再使用的对象的内存回收掉。但是在使用Go开发高性能的应用程序的时候，需要考虑GC执行垃圾回收给我们带来的影响。在GC执行垃圾回收的过程中需要STW(stop-the-world)时间，如果创建有大量的对象，GC程序在检查这些对象是不是垃圾的时候需要花费很多时间。进而对业务功能造成性能影响。

**优点：**

* 减轻GC负担
* 减少新对象的申请 复用对象，避免每次使用对象都要创建对象

1. sync.Pool 本身就是线程安全的，多个 goroutine 可以并发地调用它的方法存取对象；
2. sync.Pool 不可在使用之后再复制使用（struct 中有 **Nocopy 静态检查是否复制使用**）

**三个方法：**

**New：**当调用 Pool 的 **Get**方法从池中获取元素，

            1. 没有更多的空闲元素可返回时，就会调用这个 New 方法来创建新的元素。
            2. 没有设置 New 字段，没有更多的空闲元素可返回时，Get 方法将返回 nil，表明当前没有可用的元素。
            3. **Get：从Pool 移除一个元素，这个元素会从 Pool 中移除，返回给调用者，返回值可能是 nil**

**Put：将一个元素返还给 Pool，Pool 会把这个元素保存到池中，并且可以复用。但如果 Put 一个 nil 值，Pool 就会忽略这个值**

**原理**：

(sync.Pool结构图)

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/sync.pool.png)


**1.13< 版本之前的两个问题：**

* 每次 GC 都会回收创建的对象。
      1. **元素过多会导致 SWT 耗时变长**
      2. **缓存元素都被回收后，会导致Get 命中率下降，不得不创建心的对象**
* 底层实现使用了 Mutex，对这个锁并发请求竞争激烈的时候，会导致性能的下降。
      3. **Go 对 Pool 的优化就是避免使用锁，同时将加锁的 queue 改成 lock-free 的 queue 的实现，给即将移除的元素再多一次“复活”的机会。**

**Pool 最重要的两个字段 （local** 和 **victim**）

1. **local：**

local是一个poolLocal数组的指针，localSize代表这个数组的大小，它的大小是goroutine中P的数量。每个P都有一个ID,这个ID的值对应着poolLoca数组的下标索引。所以poolLocal数组大小最大为runtime.GOMAXPROCS(0)。victim对应local,它也是一个poolLocal数组的指针，victimSize是victim数组的大小。每次垃圾回收的时候，Pool会把victim中的对象清理掉，然后把local的数据赋值给victim，并把local设置为nil,victim像一个中转站，里面的数据可以被捡回来重新使用，也可能被GC回收掉。

2. **Victim:** 

 victim 中的元素如果被 Get 取走，那么这个元素就很幸运，因为它又“活”过来了。但是，如果这个时候 Get 的并发不是很大，元素没有被 Get 取走，那么就会被移除掉，因为没有别人引用它的话，就会被垃圾回收掉。

```go
type Pool struct {
 noCopy noCopy
 // local是一个指向[p]poolLocal的指针，它指向的是一个数组，此数组中的元素是poolLocal类型
 // 数组的大小是固定的，数量与p的数量是相同的，也是每个p对应数组中的1个元素
 local unsafe.Pointer 
 // local数组的大小
 localSize uintptr 
 // victim类型与local是一样的，可以理解为local的中转站，过渡存储local用的
 victim unsafe.Pointer 
 // victim数组的大小
 victimSize uintptr 
 // New方法，返回的空接口类型，该字段也是Pool唯一暴露给外界的字段
 // New方法可以赋值一个能够产生值的函数，在调用Get方法的时候可以用
 // New方法来产生一个value，如果没有给New赋值，调用Get时将返回nil.
 New func() interface{}
}
```

**poolLocal**：

poolLocal是sync.Pool中local或victim指向数组中的元素类型，里面有一个pad数组填充，使得poolLocal的大小构成128字节的整数倍，是为了防止在cache line上分配多个poolLocalInternal从而造成伪共享。

```go
type poolLocal struct {
 poolLocalInternal
 // pad是填充用的，防止伪共享，让整个poolLocal构成128字节的整数倍
 pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

**poolLocalInternal**：

它包含private和shared两个字段，private是一个私有对象，只能被拥有它的P使用，而一个P同时只能执行一个G, 所以G访问private不需要加锁，不存在并发冲突。shared是一个双端队列，它既能被与之绑定的P访问和修改，也能被其他的P访问和修改，绑定的P可以从队头加入和出队元素，其他的P只能从队尾出队元素

```go
// poolLocal的内部结构，Pool中的poolLocal是对poolLocalInternal的简单包装
type poolLocalInternal struct {
 // private是一个私有对象，从名字可以看出来，它只能被拥有它的P使用
 // Pool中也说明了，每个P有1个poolLocal，也即有一个poolLocalInternal
 // 这里的private只能被他属于的P使用
 private interface{} // Can be used only by the respective P.
 // shared是一个双端队列，它既能被与之绑定的P访问和修改，也能被其他的P访问和修改
 // 绑定的P可以从对头入队加和出队元素，其他的P只能从队尾出队元素。
 shared poolChain 
}
```

**poolChain：**

poolChain是一个双向链表的结构体头，它保存着双向链表的头节点和尾节点指针，链表中节点元素类型是poolChainElt。这里需要注意一下，按照我们常规数据结构的理解，head指针指向的位置在链表最左边，tail指针指向的位置在最右边，但是这里刚好是相反的，这里的head指针指向链表的最右边，tail指向最左边。每添加一个元素的时候，添加在head节点的下一个节点，然后将head指向刚添加的节点。

```go
// poolChain是一个双向链表的结构体头，它保存着双向链表的头节点和尾节点指针，poolChainElt
// 是链表中每个节点的类型结构，该结构类型中有2个字段，headTail uint64和 vals []eface，
// headTail作用后面有描述，vals就是存储元素的切片。vals的大小在每个poolChainElt是不同的，
// 相邻的2个直接大小存在着2倍的关系。
type poolChain struct {
 // head只会被1个生产者调用，即Put操作，只在头部添加元素，不需要强同步
 head *poolChainElt
 // tail被消费者调用，从尾部获取元素，可能有多个消费者同时获取，所以操作必须是原子的
 tail *poolChainElt
}
```

**poolChainElt：**

双向链表中的节点结构，它有一个存储数据的poolDequeue队列，next和prev分别是指向前后节点的指针。poolDequeue中vals字段存放元素，vals是一个固定大小的切片，相邻的两个poolChainElt中的vals大小存在2倍关系，第一个poolChainElt中vals大小是8，可以结合上面pool结构图看，第二个poolChainElt中的size是16，最大值为2^30。

```go
type poolChainElt struct {
 // 存数据的队列，是一个固定大小的切片
 poolDequeue
 // next和prev是双向链表的指针，next被生产者写，消费者读，它的值只会从
 // nil变成非nil, prev被消费者写，生产者读，它的值只会从非nil变成nil
 next, prev *poolChainElt
}
```

**poolDequeue：**

poolDequeue是一个无锁的、固定大小的，单个生产者、多个消费者的队列。poolDequeue无锁实现是Pool的精髓，巧妙采用CAS去掉了1.13版本之前的有锁实现。它有headTail和vals两个字段，headTail占8个字节，高4字节表示的是head在vals中的下标位置，低4字节表示的是tail在vals中的下标位置。生产者每生产1个数据，head+1，消费者每取走队列中1个数据，tail+1.当tail与head相等的时候，说明队列中没有元素了，当tail+len(vals)=head的时候，说明队列满了。

```go
// poolDequeue是一个无锁的、固定大小的、单个生产者、多个消费者的队列
// 生产者只能从队头向队列添加数据或者从队头取走数据，单个或多个消费者可以
// 可用队尾取走数据
type poolDequeue struct {
 // headTail占8个字节，分为两部分，高位4字节和低位4字节，高4字节表示的
 // 32位int数据描述的是head下标，低4字节表示的是32位int数据描述的是tail
 // 下标，这里的下标说的都是下面vals数组的下标，vals[tail,head)下标范围
 // 内是有数据的，可以被消费者消费。生产者每生产1个数据，head会+1， 消费者
 // 每取走队列中1个数据，tail+1, 当tail赶上head的时候，说明队列中没有数
 // 据了,头部索引存储在最高有效位中，因此我们可以原子地添加它，溢出并不会产生
 // 别的影响。
 headTail uint64
 // vals是一个存储数据的环形队列，数据可以是任何类型，因为定义的类型是interface{}
 // 它的大小必须是2整数次幂。vals[i].typ为nil,表示该下标位置的没有放数据，typ不是
 // nil表示该位置已放有数据。在tail还未设置为i+1和vals[i].typ为非nil前，vals中的
 // 位置i视为还在占用状态。当前消费者取走数据或是生产者读取走数据后，vals[i]将自动被
 // 它们设置为nil.
 vals []eface
}
type eface struct {
 typ, val unsafe.Pointer
}
```

**Get实现原理:**

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/sync.pool2.png)


Get先从本地 local 的private中获取元素，此private只能被当前的P访问,一个P同时只能运行一个G， 所以直接访问l.privated不需要加锁。如果local.private中没有元素了，尝试从local.shared队列的头部弹出一个元素，如果local.shared中也没有元素了，则从其他P对应的shared中偷取一个，如果其他P对应的shared中也没有元素了，再检查victim中是否有元素，如果还是没有，设置了New方法，将调用New方法产生一个。如果没有设置New方法，将返回nil。

```go
// Get方法从Pool中返回一个元素，并将它从Pool中移除，调用者不要假定放入（Put方法）Pool中元素和从里面
// 取（Get）出来的元素有任何关系,如果Pool中没有缓存的元素了并且P.New没有赋值将会返回nil.否则调用New
// 方法产生一个对象返回
func (p *Pool) Get() interface{} {
 if race.Enabled {
  race.Disable()
 }
 // 把当前的goroutine固定在当前的P上，返回当前的P上的*poolLocal
 l, pid := p.pin()
 // 先从本地local的private字段获取元素，此private只能被当前的P访问
 // 其他P是不能访问的，一个P同一时间只能有1个goroutine在运行，所以
 // 直接访问l.private并不需要加锁
 x := l.private
 l.private = nil
 // private中没有元素
 if x == nil {
  // 尝试从当前的本地local.shared中出队一个元素，x是从队头弹出的。
  x, _ = l.shared.popHead()
  if x == nil {
   // 如果从local.shared也没有拿到，则从其他P中的l.shared中偷1个
            // 如果shared中也没有，产生从victim中获取
   x = p.getSlow(pid)
  }
 }
 runtime_procUnpin()
 if race.Enabled {
  race.Enable()
  if x != nil {
   race.Acquire(poolRaceAddr(x))
  }
 }
 // 走到这里说明，private, local.shared, 其他P中的local.shared, 本地local.victim，
 // 其他P中的local.victim都没有缓存了，如果设置New函数，调用New函数创建一个元素返回
 if x == nil && p.New != nil {
  x = p.New()
 }
 // 走到这里说明没有设置New函数，返回nil
 return x
}
```

pin方法会调用 **runtime_procPin** 方法获取当前运行的G绑定到当前的P上，禁止抢占，返回关联P的pid. 然后原子操作取出p.localSize值与Pid进行比较，正常情况下pid是不会比localSize 大，如果不在 ocalSize 范围内，说明程序修改过P的大小，会走到pinSlow处理逻辑。
pin方法会调用**runtime_procPin**方法获取当前运行的G绑定到当前的P上，禁止抢占，返回关联P的pid. 然后原子操作取出p.localSize值与Pid进行比较，正常情况下pid是不会比localSize大，如果不在localSize范围内，说明程序修改过P的大小，会走到pinSlow处理逻辑。

```go
// 将当前的G定在与它关联的P上，禁止抢占，返回关联P对应的poolLocal对象和P的id
// 调用者必须执行runtime_procUnpin操作当结束对pool操作的时候
func (p *Pool) pin() (*poolLocal, int) {
 // 获取运行当前G,与之关联的P的id
 pid := runtime_procPin()

 // 检查localSize是不是比pid大，正常情况下p.localSize=max(pid)+1
 // 因为pid是P的id,如果P的数量为n,则pid的范围为[0,n), 如果pid的值不在
 // localSize范围内，说明程序修改过P的数量，这里就会走到pinSlow逻辑
 // pinSlow逻辑主要是重新分配p的local和localSize
 s := atomic.LoadUintptr(&p.localSize) // load-acquire
 l := p.local                          // load-consume
 if uintptr(pid) < s {
  return indexLocal(l, pid), pid
 }
 return p.pinSlow()
}
```

pinSlow会再次检查pid是否在localSize范围内，如果不在，将会重新分配poolLocal数组给local, 初始化localSize大小，并将p加入到全局的allPools切片中，allPools维护了程序中所有的sync.Pool对象。

```go
func (p *Pool) pinSlow() (*poolLocal, int) {
 runtime_procUnpin()
 allPoolsMu.Lock()
 defer allPoolsMu.Unlock()
 pid := runtime_procPin()
 // 已经加了allPoolsMu全局锁，这里可以直接读取localSize和local，不存在data race
 s := p.localSize
 l := p.local
 // 再次判断pid的值是否在正常的范围内，刚才前面解除了禁止抢占，等现在再拿到pid的时候
 // 可能不是之前的pid了
 if uintptr(pid) < s {
  return indexLocal(l, pid), pid
 }
 // 将p加入到全局pool allPools中
 if p.local == nil {
  allPools = append(allPools, p)
 }

 // GOMAXPROCS传0返回值是当前可以并发运行的CPU数
 size := runtime.GOMAXPROCS(0)
 // 创建一个新的poolLocal切片，并复制给当前的p
 local := make([]poolLocal, size)
 atomic.StorePointer(&p.local, unsafe.Pointer(&local[0])) // store-release
 atomic.StoreUintptr(&p.localSize, uintptr(size))         // store-release
 // 返回当前的P对应的localPool和pid的值
 return &local[pid], pid
}

// l存储的是local的起始地址，local是一个poolLocal数组，所以根据起始位置+poolLocal的偏移量
// 可以获取到第i个索引中的poolLocal值
func indexLocal(l unsafe.Pointer, i int) *poolLocal {
 lp := unsafe.Pointer(uintptr(l) + uintptr(i)*unsafe.Sizeof(poolLocal{}))
 return (*poolLocal)(lp)
}
```

分析Get中的popHead操作，popHead从poolChain中获取一个数据，优先从头节点head指向的环形队列中获取，如果没有获取到，再从head前一个节点指向的环形队列中获取，直到查询完所有的队列都没有，返回失败。整个处理过程是不断的从链表的head节点向尾节点遍历，对遍历到的每个节点执行popHead, 这个节点类型是poolDequeue, poolDequeue的popHead。

```go
// popHead从poolChain获取一个数据，优先从头部的环形队列中获取，
// 如果没有获取到，在从head前一个节点指向的环形队列中获取，直到
// 查询完所有的队列都没有，返回获取失败false
func (c *poolChain) popHead() (interface{}, bool) {
 d := c.head
 // 从head节点开始查找
 for d != nil {
  if val, ok := d.popHead(); ok {
   return val, ok
  }

  // 继续查找前一个队列，只要d非空即没有查到tail位置，一直
  // 查找，获取到了数据就提前返回
  d = loadPoolChainElt(&d.prev)
 }
 // 走到这里表示没有获取数据，返回失败
 return nil, false
}



// popHead从队列的头部返回一个元素，如果队列为空，第2个返回值将返回false, 注意该函数
// 只能被单个生产者调用，多个生产者即G同时调用，会引发数据竞争
func (d *poolDequeue) popHead() (interface{}, bool) {
 var slot *eface
 for {
  ptrs := atomic.LoadUint64(&d.headTail)
  head, tail := d.unpack(ptrs)
  // 下标tail和head相等了，说明tail追赶上了head,即将vals中的元素取完了
  if tail == head {
   // Queue is empty.
   return nil, false
  }

  head--
  // 构建一个新的uint64,head是-1后的值
  ptrs2 := d.pack(head, tail)
  // 进一步比较headTail和ptrs，如果相同说明没有修改它，将其设置为ptrs2
  // 这时可以放心大胆的对head进行操作，获取里面的元素了，如果这段时间headTail
  // 有人修改，继续进行下一轮循环处理
  if atomic.CompareAndSwapUint64(&d.headTail, ptrs, ptrs2) {
   // 这里可以放心大胆的读取vals中head下标处的元素了
   slot = &d.vals[head&uint32(len(d.vals)-1)]
   break
  }
 }

 // 这里判断slot中的val是不是dequeueNil型，因为在pushHead中有肯能会放入，作占位用
 val := *(*interface{})(unsafe.Pointer(slot))
 if val == dequeueNil(nil) {
  val = nil
 }

 // 将slot填充为nil值，因为popHead和pushHead不会同时被调用，所以这里可以放心大胆对其进行
 // 操作，还有一点是，前面已经重新设置了head值了，popTail已经不可能访问到slot这里，也不用担心
 // popTail与popHead在这里的冲突问题
 *slot = eface{}
 return val, true
}
```

如果shared中的popHead方法也没有获取到值，将会调下面的getSlow方法。先尝试从其他的P对应的poolLocal中偷一个元素，尝试的顺序是从当前pid+1个索引位置开始的，会对sync.local检查一圈。对每个poolLocal检查的是shared中的元素，看尾部有没有元素，如果有从尾部弹出一个元素。如果检查完所有的poolLocal中的shared都没有找到，接下来将在当前本地poolLocal中的victim的private查找，如果还是没有，就检查它的victim的shared是否有，最后检查其他P对应的poolLocal中victim中的shared是否有数据。

```go
func (p *Pool) getSlow(pid int) interface{} {
 // See the comment in pin regarding ordering of the loads.
 size := atomic.LoadUintptr(&p.localSize) // load-acquire
 locals := p.local                        // load-consume
 // Try to steal one element from other procs.
 // 尝试从其他P中偷一个元素，尝试P的顺序是从当前pid+1个索引位置开始对应的
 // poolLocal中的shared开始查找是否有元素，在shared中的查询方式是从
 // 它的尾部弹出一个元素，如果shared队列中没有，继续尝试下一个位置，直到
 // 循环检查一圈都没有，走victim中检查
 for i := 0; i < int(size); i++ {
  l := indexLocal(locals, (pid+i+1)%int(size))
  if x, _ := l.shared.popTail(); x != nil {
   return x
  }
 }

 size = atomic.LoadUintptr(&p.victimSize)
 if uintptr(pid) >= size {
  return nil
 }
 // 下面尝试从受害中缓存victim中查找是否元素，查找的位置是从pid索引位置开始的poolLocal
 // 产生从它的shared尾部弹出一个元素，如果有就返回，如果没有就尝试下一个位置的poolLocal
 locals = p.victim
 l := indexLocal(locals, pid)
 // 检查victim.private是否有元素，有就返回
 if x := l.private; x != nil {
  l.private = nil
  return x
 }
 for i := 0; i < int(size); i++ {
  l := indexLocal(locals, (pid+i)%int(size))
  if x, _ := l.shared.popTail(); x != nil {
   return x
  }
 }

 // 这里将p.victimSize设置为0，是为了减少后序调用getSlow,不用再一次遍历
 // victim数组了，直接返回nil, 加快处理速度
 atomic.StoreUintptr(&p.victimSize, 0)
 // 尝试了所有的local.shared和victim.private, victim.shared都没有找到，返回nil
 return nil
}



// popTail从poolChain中获取一个数据返回，第一个返回值表示返回的数据，第二个
// 返回值表示是否成功获取到了数据，如果没有获取到，将返回false
func (c *poolChain) popTail() (interface{}, bool) {
 d := loadPoolChainElt(&c.tail)
 // d为nil,表示c.tail还未初始化，比如第一次执行Get操作的时候就会发生，这个时候
 // 还没有Put数据进去，所以获取失败
 if d == nil {
  return nil, false
 }

 for {
  
  // 需要注意这里先要进行loadPoolChainElt操作再执行d.popTail操作，
  // 这两条语句不能颠倒顺序，一般来说，d可能短暂为空，如果在执行pop
  // 且执行失败了，但是在执行前d2为非空。这种情况d永久会为空了，需要将
  // d从poolChain中移除掉
  d2 := loadPoolChainElt(&d.next)

  if val, ok := d.popTail(); ok {
   return val, ok
  }

  if d2 == nil {
   // 走完整个链表了，还是没有获取到数据，直接返回nil,false
   return nil, false
  }

  // 当前tail执行的节点中的环形队列中已经没有数据可以获取了，尝试tail的
  // next节点中获取，因为当前的tail节点永远不会再有数据，所以把它从poolChain
  // 移除掉，下次再执行popTail的时候，就不用检查这个空队列了。
  // 这里采用了CAS操作，因为有多个消费者可能并行执行popTail.
  if atomic.CompareAndSwapPointer((*unsafe.Pointer)(unsafe.Pointer(&c.tail)), unsafe.Pointer(d), unsafe.Pointer(d2)) {
   
   // 将d从poolChain中移除，以便在gc的时候可以将d回收掉，
   // 那为啥在上面的popHead中没有将d从poolChain中移除掉呢？
   // 因为popHead中的d可以留着，在后序的pushHead中可以继续
   // 使用，而popTail中的d永远不会再被使用了，因为没有pushTail操作
   storePoolChainElt(&d2.prev, nil)
  }
  d = d2
 }
}
```

上面的**popTail**对d的处理做了优化，将d从poolChain中移除，以便在gc的时候可以将d回收掉，那为啥在上面的popHead中没有将d从poolChain中移除掉呢？因为popHead中的d可以留着，在后序的pushHead中可以继续使用，而popTail中d永远不会再被使用了，因为没有pushTail操作。 

**Put实现原理**

Put操作处理逻辑比较简单，优先将元素放在本地的private,如果private字段已经有值了，会将元素添加到本地的shared队列中。

![synpool321](https://image-1302243118.cos.ap-beijing.myqcloud.com/go/synpool321.png)

**GC操作：**

sync.Pool中的对象并不是一直保持不释放的。否则对象太多了，会引起内存溢出。Go中的GC对sync.Pool做了专门的处理。在pool.go的init函数里，向GC注册了清理机制，源码如下.poolCleanup 会在 STW 阶段被调用。主要是将 local 和 victim 作交换，那么不至于GC 把所有的 Pool 都清空了，而是需要两个 GC 周期才会被释放


```go
// 注册pool清理回调，在gc的时候，会调用poolCleanup
func init() {
 runtime_registerPoolCleanup(poolCleanup)
}
// 清理pool函数，对oldPools和allPools的操作都是没有加锁的，因为这里执行的时候
// 已经STW了，不存在竞争
func poolCleanup() {
 // oldPools是一个全局变量，是一个保存*sync.Pool的切片，这些
 // 切片元素都是等待被gc清理的，当执行完下面的循环之后，就被清理了
 for _, p := range oldPools {
  p.victim = nil
  p.victimSize = 0
 }
 // allPools是当前新产生的*sync.Pool元素集合，里面存储的元素跟前面的
 // oldPools是不同的，当该轮gc执行的时候，会将allPools中所有的Pool中的
 // local元素移动到victim受害者缓存中，并将local缓存清空
 for _, p := range allPools {
  p.victim = p.local
  p.victimSize = p.localSize
  p.local = nil
  p.localSize = 0
 }
 // 交换oldPools和allPools，处理的很精炼，为什么要这么做呢？
 // 进程起来，运行过程中申请了sync.Pool创建的变量vp，然后执行gc,此时oldPools是空的
 // allPools是非空的，里面装的是vp, 当该函数被调用的时候，将vp中的元素从local移动到victim
 // 即在第一次gc的时候，vp中的元素没有被垃圾回收，接下来执行下面这个语句之后，allPools被清空
 // oldPools里面保存的是vp, 当下次执行gc的时候，oldPools中的内容被清理掉，所以vp中元素生命
 // 周期在两次gc之间都是存活的
 oldPools, allPools = allPools, nil
}
```

