> 题目来源：虾皮

答案：小禾先生

想要启动一个新的 Goroutine 来执行任务时，我们需要使用 Go 语言的 `go` 关键字，编译器会通过 [`cmd/compile/internal/gc.state.stmt`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmt) 和 [`cmd/compile/internal/gc.state.call`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.call) 两个方法将该关键字转换成 [`runtime.newproc`](https://draveness.me/golang/tree/runtime.newproc) 函数调用：

```go
func (s *state) call(n *Node, k callKind) *ssa.Value {
	if k == callDeferStack {
		...
	} else {
		switch {
		case k == callGo:
			call = s.newValue1A(ssa.OpStaticCall, types.TypeMem, newproc, s.mem())
		default:
		}
	}
	...
}
```

[`runtime.newproc`](https://draveness.me/golang/tree/runtime.newproc) 的入参是参数大小和表示函数的指针 `funcval`，它会获取 Goroutine 以及调用方的程序计数器，然后调用 [`runtime.newproc1`](https://draveness.me/golang/tree/runtime.newproc1) 函数获取新的 Goroutine 结构体、将其加入处理器的运行队列并在满足条件时调用 [`runtime.wakep`](https://draveness.me/golang/tree/runtime.wakep) 唤醒新的处理执行 Goroutine：

```go
func newproc(siz int32, fn *funcval) {
	argp := add(unsafe.Pointer(&fn), sys.PtrSize)
	gp := getg()
	pc := getcallerpc()
	systemstack(func() {
		newg := newproc1(fn, argp, siz, gp, pc)

		_p_ := getg().m.p.ptr()
		runqput(_p_, newg, true)

		if mainStarted {
			wakep()
		}
	})
}
```

[`runtime.newproc1`](https://draveness.me/golang/tree/runtime.newproc1) 会根据传入参数初始化一个 `g` 结构体，我们可以将该函数分成以下几个部分介绍它的实现：

1. 获取或者创建新的 Goroutine 结构体；
2. 将传入的参数移到 Goroutine 的栈上；
3. 更新 Goroutine 调度相关的属性；

首先是 Goroutine 结构体的创建过程：

```go
func newproc1(fn *funcval, argp unsafe.Pointer, narg int32, callergp *g, callerpc uintptr) *g {
	_g_ := getg()
	siz := narg
	siz = (siz + 7) &^ 7

	_p_ := _g_.m.p.ptr()
	newg := gfget(_p_)
	if newg == nil {
		newg = malg(_StackMin)
		casgstatus(newg, _Gidle, _Gdead)
		allgadd(newg)
	}
	...
```

上述代码会先从处理器的 `gFree` 列表中查找空闲的 Goroutine，如果不存在空闲的 Goroutine，会通过 [`runtime.malg`](https://draveness.me/golang/tree/runtime.malg) 创建一个栈大小足够的新结构体。

接下来，我们会调用 [`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 将 `fn` 函数的所有参数拷贝到栈上，`argp` 和 `narg` 分别是参数的内存空间和大小，我们在该方法中会将参数对应的内存空间整块拷贝到栈上：

```go
	...
	totalSize := 4*sys.RegSize + uintptr(siz) + sys.MinFrameSize
	totalSize += -totalSize & (sys.SpAlign - 1)
	sp := newg.stack.hi - totalSize
	spArg := sp
	if narg > 0 {
		memmove(unsafe.Pointer(spArg), argp, uintptr(narg))
	}
	...
```

拷贝了栈上的参数之后，[`runtime.newproc1`](https://draveness.me/golang/tree/runtime.newproc1) 会设置新的 Goroutine 结构体的参数，包括栈指针、程序计数器并更新其状态到 `_Grunnable` 并返回：

```go
	...
	memclrNoHeapPointers(unsafe.Pointer(&newg.sched), unsafe.Sizeof(newg.sched))
	newg.sched.sp = sp
	newg.stktopsp = sp
	newg.sched.pc = funcPC(goexit) + sys.PCQuantum
	newg.sched.g = guintptr(unsafe.Pointer(newg))
	gostartcallfn(&newg.sched, fn)
	newg.gopc = callerpc
	newg.startpc = fn.fn
	casgstatus(newg, _Gdead, _Grunnable)
	newg.goid = int64(_p_.goidcache)
	_p_.goidcache++
	return newg
}
```

我们在分析 [`runtime.newproc`](https://draveness.me/golang/tree/runtime.newproc) 的过程中，保留了主干省略了用于获取结构体的 [`runtime.gfget`](https://draveness.me/golang/tree/runtime.gfget)、[`runtime.malg`](https://draveness.me/golang/tree/runtime.malg)、将 Goroutine 加入运行队列的 [`runtime.runqput`](https://draveness.me/golang/tree/runtime.runqput) 以及设置调度信息的过程，下面会依次分析这些函数。

**初始化结构体**

[`runtime.gfget`](https://draveness.me/golang/tree/runtime.gfget) 通过两种不同的方式获取新的 [`runtime.g`](https://draveness.me/golang/tree/runtime.g)：

1. 从 Goroutine 所在处理器的 `gFree` 列表或者调度器的 `sched.gFree` 列表中获取 [`runtime.g`](https://draveness.me/golang/tree/runtime.g)；
2. 调用 [`runtime.malg`](https://draveness.me/golang/tree/runtime.malg) 生成一个新的 [`runtime.g`](https://draveness.me/golang/tree/runtime.g) 并将结构体追加到全局的 Goroutine 列表 `allgs` 中。

![golang-newproc-get-goroutine](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/golang-newproc-get-goroutine.png)

[`runtime.gfget`](https://draveness.me/golang/tree/runtime.gfget) 中包含两部分逻辑，它会根据处理器中 `gFree` 列表中 Goroutine 的数量做出不同的决策：

1. 当处理器的 Goroutine 列表为空时，会将调度器持有的空闲 Goroutine 转移到当前处理器上，直到 `gFree` 列表中的 Goroutine 数量达到 32；
2. 当处理器的 Goroutine 数量充足时，会从列表头部返回一个新的 Goroutine；

```go
func gfget(_p_ *p) *g {
retry:
	if _p_.gFree.empty() && (!sched.gFree.stack.empty() || !sched.gFree.noStack.empty()) {
		for _p_.gFree.n < 32 {
			gp := sched.gFree.stack.pop()
			if gp == nil {
				gp = sched.gFree.noStack.pop()
				if gp == nil {
					break
				}
			}
			_p_.gFree.push(gp)
		}
		goto retry
	}
	gp := _p_.gFree.pop()
	if gp == nil {
		return nil
	}
	return gp
}
```

当调度器的 `gFree` 和处理器的 `gFree` 列表都不存在结构体时，运行时会调用 [`runtime.malg`](https://draveness.me/golang/tree/runtime.malg) 初始化新的 [`runtime.g`](https://draveness.me/golang/tree/runtime.g) 结构，如果申请的堆栈大小大于 0，这里会通过 [`runtime.stackalloc`](https://draveness.me/golang/tree/runtime.stackalloc) 分配 2KB 的栈空间：

```go
func malg(stacksize int32) *g {
	newg := new(g)
	if stacksize >= 0 {
		stacksize = round2(_StackSystem + stacksize)
		newg.stack = stackalloc(uint32(stacksize))
		newg.stackguard0 = newg.stack.lo + _StackGuard
		newg.stackguard1 = ^uintptr(0)
	}
	return newg
}
```

[`runtime.malg`](https://draveness.me/golang/tree/runtime.malg) 返回的 Goroutine 会存储到全局变量 `allgs` 中。

简单总结一下，[`runtime.newproc1`](https://draveness.me/golang/tree/runtime.newproc1) 会从处理器或者调度器的缓存中获取新的结构体，也可以调用 [`runtime.malg`](https://draveness.me/golang/tree/runtime.malg) 函数创建。

**运行队列**

[`runtime.runqput`](https://draveness.me/golang/tree/runtime.runqput) 会将 Goroutine 放到运行队列上，这既可能是全局的运行队列，也可能是处理器本地的运行队列：

```go
func runqput(_p_ *p, gp *g, next bool) {
	if next {
	retryNext:
		oldnext := _p_.runnext
		if !_p_.runnext.cas(oldnext, guintptr(unsafe.Pointer(gp))) {
			goto retryNext
		}
		if oldnext == 0 {
			return
		}
		gp = oldnext.ptr()
	}
retry:
	h := atomic.LoadAcq(&_p_.runqhead)
	t := _p_.runqtail
	if t-h < uint32(len(_p_.runq)) {
		_p_.runq[t%uint32(len(_p_.runq))].set(gp)
		atomic.StoreRel(&_p_.runqtail, t+1)
		return
	}
	if runqputslow(_p_, gp, h, t) {
		return
	}
	goto retry
}
```

1. 当 `next` 为 `true` 时，将 Goroutine 设置到处理器的 `runnext` 作为下一个处理器执行的任务；
2. 当 `next` 为 `false` 并且本地运行队列还有剩余空间时，将 Goroutine 加入处理器持有的本地运行队列；
3. 当处理器的本地运行队列已经没有剩余空间时就会把本地队列中的一部分 Goroutine 和待加入的 Goroutine 通过 [`runtime.runqputslow`](https://draveness.me/golang/tree/runtime.runqputslow) 添加到调度器持有的全局运行队列上；

处理器本地的运行队列是一个使用数组构成的环形链表，它最多可以存储 256 个待执行任务。

![golang-runnable-queue](https://image-1302243118.cos.ap-beijing.myqcloud.com/golang.assets/2020-02-05-15808864354654-golang-runnable-queue.png)

简单总结一下，Go 语言有两个运行队列，其中一个是处理器本地的运行队列，另一个是调度器持有的全局运行队列，只有在本地运行队列没有剩余空间时才会使用全局队列。

**调度信息**

运行时创建 Goroutine 时会通过下面的代码设置调度相关的信息，前两行代码会分别将程序计数器和 Goroutine 设置成 [`runtime.goexit`](https://draveness.me/golang/tree/runtime.goexit) 和新创建 Goroutine 运行的函数：

```go
	...
	newg.sched.pc = funcPC(goexit) + sys.PCQuantum
	newg.sched.g = guintptr(unsafe.Pointer(newg))
	gostartcallfn(&newg.sched, fn)
	...
```

上述调度信息 `sched` 不是初始化后的 Goroutine 的最终结果，它还需要经过 [`runtime.gostartcallfn`](https://draveness.me/golang/tree/runtime.gostartcallfn) 和 [`runtime.gostartcall`](https://draveness.me/golang/tree/runtime.gostartcall) 的处理：

```go
func gostartcallfn(gobuf *gobuf, fv *funcval) {
	gostartcall(gobuf, unsafe.Pointer(fv.fn), unsafe.Pointer(fv))
}

func gostartcall(buf *gobuf, fn, ctxt unsafe.Pointer) {
	sp := buf.sp
	if sys.RegSize > sys.PtrSize {
		sp -= sys.PtrSize
		*(*uintptr)(unsafe.Pointer(sp)) = 0
	}
	sp -= sys.PtrSize
	*(*uintptr)(unsafe.Pointer(sp)) = buf.pc
	buf.sp = sp
	buf.pc = uintptr(fn)
	buf.ctxt = ctxt
}
```

**参考资料**

- https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/#654-%e5%88%9b%e5%bb%ba-goroutine