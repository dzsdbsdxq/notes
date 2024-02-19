Go atomic包是最轻量级的锁（也称无锁结构），可以在不形成临界区和创建互斥量的情况下完成并发安全的值替换操作，不过这个包只支持int32/int64/uint32/uint64/uintptr这几种数据类型的一些基础操作（增减、交换、载入、存储等）

**概念：**

原子操作仅会由一个独立的CPU指令代表和完成。原子操作是无锁的，常常直接通过CPU指令直接实现。 事实上，其它同步技术的实现常常依赖于原子操作。

**使用场景：**

当我们想要对**某个变量**并发安全的修改，除了使用官方提供的 `mutex`，还可以使用 sync/atomic 包的原子操作，它能够保证对变量的读取或修改期间不被其他的协程所影响。

atomic 包提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作，善用 atomic 能够避免程序中出现大量的锁操作。

**常见操作：**

-   增减Add
-   载入Load
-   比较并交换CompareAndSwap
-   交换Swap
-   存储Store

atomic 操作的对象是一个地址，你需要把可寻址的变量的地址作为参数传递给方法，而不是把变量的值传递给方法

下面将分别介绍这些操作：

**增减操作**

此类操作的前缀为 `Add`

```
func AddInt32(addr *int32, delta int32) (new int32)

func AddInt64(addr *int64, delta int64) (new int64)

func AddUint32(addr *uint32, delta uint32) (new uint32)

func AddUint64(addr *uint64, delta uint64) (new uint64)

func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

需要注意的是，第一个参数必须是指针类型的值，通过指针变量可以获取被操作数在内存中的地址，从而施加特殊的CPU指令，确保同一时间只有一个goroutine能够进行操作。

使用举例：

```go
func add(addr *int64, delta int64) {
atomic.AddInt64(addr, delta) //加操作
fmt.Println("add opts: ", *addr)
}

```

**载入操作**

此类操作的前缀为 `Load`

```
func LoadInt32(addr *int32) (val int32)

func LoadInt64(addr *int64) (val int64)

func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)

func LoadUint32(addr *uint32) (val uint32)

func LoadUint64(addr *uint64) (val uint64)

func LoadUintptr(addr *uintptr) (val uintptr)

// 特殊类型： Value类型，常用于配置变更
func (v *Value) Load() (x interface{}) {}
```

载入操作能够保证原子的读变量的值，当读取的时候，任何其他CPU操作都无法对该变量进行读写，其实现机制受到底层硬件的支持。

使用示例:

```
func load(addr *int64) {
fmt.Println("load opts: ", atomic.LoadInt64(&opts))
}
```

**比较并交换**

此类操作的前缀为 `CompareAndSwap`, 该操作简称 CAS，可以用来实现乐观锁

```
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)

func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)

func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)

func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)

func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)

func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
```

该操作在进行交换前首先确保变量的值未被更改，即仍然保持参数 `old` 所记录的值，满足此前提下才进行交换操作。CAS的做法类似操作数据库时常见的乐观锁机制。

需要注意的是，当有大量的goroutine 对变量进行读写操作时，可能导致CAS操作无法成功，这时可以利用for循环多次尝试。

使用示例：

```go
func compareAndSwap(addr *int64, oldValue int64, newValue int64) {
if atomic.CompareAndSwapInt64(addr, oldValue, newValue) {
fmt.Println("cas opts: ", *addr)
return
}
}
```

**交换**

此类操作的前缀为 `Swap`：

```
func SwapInt32(addr *int32, new int32) (old int32)

func SwapInt64(addr *int64, new int64) (old int64)

func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)

func SwapUint32(addr *uint32, new uint32) (old uint32)

func SwapUint64(addr *uint64, new uint64) (old uint64)

func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
```

相对于CAS，明显此类操作更为暴力直接，并不管变量的旧值是否被改变，直接赋予新值然后返回背替换的值。  

```
func swap(addr *int64, newValue int64) {
atomic.SwapInt64(addr, newValue)
fmt.Println("swap opts: ", *addr)
}
```

**存储**

此类操作的前缀为 `Store`：

```
func StoreInt32(addr *int32, val int32)

func StoreInt64(addr *int64, val int64)

func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)

func StoreUint32(addr *uint32, val uint32)

func StoreUint64(addr *uint64, val uint64)

func StoreUintptr(addr *uintptr, val uintptr)

// 特殊类型： Value类型，常用于配置变更
func (v *Value) Store(x interface{})

```

此类操作确保了写变量的原子性，避免其他操作读到了修改变量过程中的脏数据。

```
func store(addr *int64, newValue int64) {
atomic.StoreInt64(addr, newValue)
fmt.Println("store opts: ", *addr)
}
```

###