> **题目序号：**（6628）
>
> **题目来源**：Klook 
>
> **频次**：1

**答案1：**（peace）

```go
//一个队列
type MyStack struct {
    queue []int
}

func Constructor() MyStack {
    return MyStack{
        queue: make([]int, 0), 
    }
}

func (this *MyStack) Push(x int)  {
    this.queue = append(this.queue, x)
}

func (this *MyStack) Pop() int {//出队操作
    n := len(this.queue)-1 
    for n!=0{ ////除了最后一个，其余的都重新添加到队列里
        val := this.queue[0]
        this.queue = this.queue[1:]
        this.queue = append(this.queue, val)
        n--
    }//新入队元素x到了最前面，出队时直接出-》后入先出
    val := this.queue[0]
    this.queue = this.queue[1:]
    return val
}

func (this *MyStack) Top() int {
    val := this.Pop()
    this.queue = append(this.queue, val)
    return val
}

func (this *MyStack) Empty() bool {
    if len(this.queue)==0{
        return true
    }
    return false
}
```
