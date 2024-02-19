> **题目序号：**（1629）
> **题目来源：**腾讯
> **频次：**1

**答案1：**（自由）

recover 可以中止 panic 造成的程序崩溃，或者说平息运行时恐慌，recover 函数不需要任何参数，并且会返回一个空接口类型的值。需要注意的是 recover 只能在 defer 中发挥作用，在其他作用域中调用不会发挥作用。
编译器会将 recover 转换成 runtime.gorecover，该函数的实现逻辑是如果当前 goroutine 没有调用 panic，那么该函数会直接返回 nil，当前 goroutine 调用 panic 后，会先调用 runtime.gopaic 函数runtime.gopaic 会从 runtime._defer 结构体中取出程序计数器 pc 和栈指针 sp，再调用 runtime.recovery 函数来恢复程序，runtime.recovery 会根据传入的 pc 和 sp 跳转回 runtime.deferproc，编译器自动生成的代码会发现 runtime.deferproc 的返回值不为 0，这时会调回 runtime.deferreturn 并恢复到正常的执行流程。总的来说恢复流程就是通过程序计数器来回跳转。



> **题目序号：**（2006）
> **题目来源：**Shein
> **频次：**1

**答案1：**（自由）

Go 语言核心开发者 Dave 曾说过 "You only need to check the error value if you care about the result"，我理解这句话的意思是，如果你不关心某个方法或者函数返回的 error，那你也不应该去关心它会返回什么结果。在我们不处理错误的时候，我们不应该对它的返回值抱有任何幻想。什么时候可以忽略 error ？你对 value 也不关心。刚关注 go 语言的人经常会吐槽："三分之一的代码都是 if err != nil"，会写大量的 if 处理代码。如果你也有这种不满，读完这段解析，希望能对你有一点小小的帮助。我们先看一下各个语言错误处理演进历史，C：单返回值，一般通过传递指针作为入参，返回值 int 表示成功还是失败；C++：引入了 exception，但是无法知道被调用方会抛出什么异常。Java：引入了 checked exception，方法的所有者必须声明，调用者必须处理。Java 的异常已经不再是异常了，变成一件司空见惯的事情了。从良性到灾难性，都使用它，异常的严重性交给函数的调用者来决定，并且大部分 Java 程序员往往使用根 exception 来处理任何错误，不严谨。Go 语言的异常处理，没有引入 exception，而是使用了多参数返回，在返回中带上错误，由调用者来判定这个错误。Go 语言中还引入了 panic 的机制，panic 中文意思代表恐慌，panic 可以通过 recovery 来捕获，如果你把它理解为 Java 中的 try catch，那你就错了，Go panic 意味着 fatal error（程序直接挂了），不能假设调用者来解决 panic，panic 意味着代码不能继续运行了，而 error 是扔给调用者来处理。对于真正意外的情况，那些表示不可恢复的程序错误，例如索引越界、不可恢复的环境问题、栈溢出，才应该使用 panic。对于其他的错误情况，我们应该期望使用 error 来进行判定。使用多个返回值和一个简单的约定，Go 程序员能够知道什么时候是出了问题，什么时候是异常，并且为真正的异常保留了 panic 进行处理。Go 语言使用 error 的优势有以下几点：

* 简单
* 没有隐藏的控制流
* Error are values
* 考虑失败，而不是成功
* 完全交给你控制 error 在一个互联网的世界里，网络的每一个输入都必须被认为是恶意的。同时我们需要知道，任何一行代码，都可能出错