> 题目来源：字节跳动
>
> 频次：高频

## 答案：Evan.C

执行顺序应该为panic、defer、recover

- 发生panic的函数并不会立刻返回，而是先层层函数执行defer，再返回。如果有办法将panic捕获到panic，就正常处理（若是外部函数捕获到，则外部函数只执行defer），如果没有没有捕获，程序直接异常终止。
- Go语言提供了recover内置函数。前面提到，一旦panic逻辑就会走到defer（**defer必须在panic的前面！**)。调用recover函数将会捕获到当前的panic，被捕获到的panic就不会向上传递了
- 在panic发生时，**在前面的defer中通过recover捕获这个panic，转化为错误通过返回值告诉方法调用者**。  

