> 题目来源：字节跳动

## 答案：ORVR

在Go语言中开一个协程非常方便，在需要通过协程来执行的函数时，直接在函数前加go关键字就可以

```go
package main

import (

   "fmt"

)

func A(i int) {

    fmt.Println("我是A")

}

func main() {

    fmt.Println("我是main")

    go A(1)

    fmt.Println("执行完了")

}

//输出

//我是main

//执行完了
```

程序正常执行没有报错，但是没有函数A的输出．

这是因为主协程并不会等待子协程执行完才往下走，执行到go后，语句会继续执行，go后面的函数新开一个协程，各跑各的，所以主协程执行完go语句，就无事可做，就退出了．

那怎么让上面的代码打印出函数A的输出．得让主函数待一会儿协程执行完了再退出，或者让主协程不退出，比如在web程序中，主协程是不退出的．

- 通过**sync. WaitGroup**的三个方法 **Add()**, **Done()**, **Wait()** 来实现协程的控制
- **通过带buffer的channel来控制**
- **通过sync. Cond**

下面演示下等待协程完成的代码

```go
package main

import (

    "fmt"

    "sync"

)

func A(i int) {

    fmt.Println("我是A", i)

}

func main() {

    var wg sync.WaitGroup

    fmt.Println("我是main")

    wg.Add(1)

    go func(i int) {

        defer wg.Done()

        A(i)

    }(1)

    wg.Wait()

    fmt.Println("执行完了")

}

//输出

//我是main

//我是A 1

//执行完了
```

下面演示通过channel来控制协程的流程

```go
package main

import (

    "fmt"

)

func A(i int) {

    fmt.Println("我是A", i)

}

func main() {

    ch := make(chan bool, 1)

    fmt.Println("我是main")

    go func(i int, chp chan<- bool) {

        defer close(chp)

        A(i)

        fmt.Println("finish")

        chp <- true

    }(1, ch)

    fmt.Println("wait")

    <-ch

    fmt.Println("执行完了")

}

//输出

//我是main

//wait

//我是A 1

//finish

//执行完
```

下面演示通过sync. Cond来实现

```go
package main

import (

    "fmt"

    "sync"

)

func A(i int) {

    fmt.Println("我是A", i)

}

func main() {

    var locker = new(sync.Mutex)

    var cond = sync.NewCond(locker)

    var done bool = false

    fmt.Println("我是main")

    cond.L.Lock()

    go func(i int) {

        A(i)

        fmt.Println("finish")

        done = true

        cond.Signal()

    }(1)

    fmt.Println("wait")

    if !done {

        cond.Wait()

        cond.L.Unlock()

    }

    fmt.Println("执行完了")

}

//输出

//我是main

//wait

//我是A 1

//finish

//执行完了
```

