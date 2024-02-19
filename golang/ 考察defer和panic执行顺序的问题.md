> 题目来源: 小米 
>
> 频次: 1

答案：苦痛律动

**例子**

```go
func main(){
	call()
	fmt.Println("333 Helloworld")
}

func call()  {
	
	defer func(){
		fmt.Println("11111")
	}()
	defer func(){
		fmt.Println("22222")
	}()
	defer func() {
		if r := recover(); r!= nil {
			fmt.Println("Recover from r : ",r)
		}
	}()
	defer func(){
		fmt.Println("33333")
	}()

	fmt.Println("111 Helloworld")
	panic("Panic 1!")
    // 这个panic和后面的代码已经不会执行，ide应该会有提示
	panic("Panic 2!")
	fmt.Println("222 Helloworld")
}

// output
111 Helloworld
33333
Recover from r :  Panic 1!
22222
11111
333 Helloworld
```

**源码实现**

```go
// panic 关键字代码实现. 
func gopanic(e interface{}) {
    gp := getg()    // getg()返回当前协程的 g 结构体指针，g 结构体描述 goroutine
    if gp.m.curg != gp {
        print("panic: ")
        printany(e)
        print("\n")
        throw("panic on system stack")
    }

    //  省略无用代码. 
    var p _panic
    p.arg = e
    p.link = gp._panic
    gp._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

    atomic.Xadd(&runningPanicDefers, 1)

    for {
        
        // 获取当前协程的defer链表.
        // 这里defer链表和在代码定义的顺序是相反的，类似于先进后出的概念. 
        d := gp._defer    
        if d == nil {
            break    // 当前协程的defer都被执行后，defer链表为空，此时退出for循环
        }

        // If defer was started by earlier panic or Goexit (and, since we're back here, that triggered a new panic),
        // take defer off list. The earlier panic or Goexit will not continue running.
        if d.started {    // 发生panic后，在defer中又遇到panic(),则会进入这个代码块
            if d._panic != nil {
                d._panic.aborted = true
            }
            d._panic = nil
            d.fn = nil
            gp._defer = d.link
            freedefer(d)  // defer 已经被执行过，则释放这个defer，继续for循环。
            continue
        }

        // Mark defer as started, but keep on list, so that traceback
        // can find and update the defer's argument frame if stack growth
        // or a garbage collection happens before reflectcall starts executing d.fn.
        d.started = true

        // Record the panic that is running the defer.
        // If there is a new panic during the deferred call, that panic
        // will find d in the list and will mark d._panic (this panic) aborted.
        d._panic = (*_panic)(noescape(unsafe.Pointer(&p)))

        p.argp = unsafe.Pointer(getargp(0))
        reflectcall(nil, unsafe.Pointer(d.fn), deferArgs(d), uint32(d.siz), uint32(d.siz))   // 执行当前协程defer链表头的defer
        p.argp = nil

        // reflectcall did not panic. Remove d.
        if gp._defer != d {
            throw("bad defer entry in panic")
        }
        d._panic = nil
        d.fn = nil
        gp._defer = d.link  // 从defer链中移除刚刚执行过的defer

        // trigger shrinkage to test stack copy. See stack_test.go:TestStackPanic
        //GC()

        pc := d.pc
        sp := unsafe.Pointer(d.sp) // must be pointer so it gets adjusted during stack copy
        freedefer(d)   // 释放刚刚执行过的defer
        if p.recovered {    // defer()中遇到recover后进入这个代码块
            atomic.Xadd(&runningPanicDefers, -1)

            gp._panic = p.link
            mcall(recovery)   // 跳转到recover()处，继续往下执行
            throw("recovery failed") // mcall should not return
        }
    }

    // ran out of deferred calls - old-school panic now
    // Because it is unsafe to call arbitrary user code after freezing
    // the world, we call preprintpanics to invoke all necessary Error
    // and String methods to prepare the panic strings before startpanic.
    preprintpanics(gp._panic)
    startpanic()

    // startpanic set panicking, which will block main from exiting,
    // so now OK to decrement runningPanicDefers.
    atomic.Xadd(&runningPanicDefers, -1)
    printpanics(gp._panic)   // 输出panic信息
    dopanic(0)       // should not return
    *(*int)(nil) = 0 // not reached
}
```

**概括**

大概的流程就是，如果遇见panic关键字的话，那么go执行器就会进入代码gopanic函数中，进入之后会拿到表示当前协程g的指针，然后通过该指针拿到当前协程的defer链表，通过for循环来进行执行defer，如果在defer中又遇见了panic的话，则会释放这个defer，通过continue去执行下一个defer，然后就是一个一个的执行defer了，如果在defer中遇见recover，那么将会通过mcall(recovery)去执行panic.

**参考资料**

https://juejin.cn/post/6974298479065563166