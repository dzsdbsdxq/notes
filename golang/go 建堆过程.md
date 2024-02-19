> 题目来源: 字节跳动 
>
> 频次:1

## 答案：苦痛律动

**堆的概念**

1. 堆是一个完全二叉树 (除了最后一层，其他都是满节点，最后一层先排左节点）
2. 堆中每一个节点的值都必须大于等于（或小于等于）其子树中每个节点的值。
3. 完全二叉树适合用数组存储，因为下标为 i 的元素，它的左子树下标为 2i, 右子树下标为 2i+1。父节点就是 i/2 的 overflow。

大顶堆： 堆中每一个节点的值都必须大于等于其子树中每个节点的值。
小顶堆：堆中每一个节点的值都必须小于等于其子树中每个节点的值。

**建堆**

堆是用数组存储的，而且 0 下标不存，从 1 开始存储，建堆就是在原地通过交换位置，达到建堆的目的。完全二叉树我们知道，如果最后一个元素的下标为 n, 则 1 到 n/2 是非叶子节点，需要自上而下的堆化（和子节点比较），n/2 +1 到 n 是叶子节点，不需要堆化。

```go
type heap struct{
    m []int
    len int //堆中有多少元素
}

func main() {
    m := []int{0,9,3,6,2,1,7} //第0个下标不放目标元素
    h := buildHeap(m) //建堆，返回一个heap结构
    h.Push(50)
    h.Pop()
    fmt.Println(h.m)
}
/**
建堆，就是在原切片上操作，形成堆结构
只要按照顺序，把切片下标为n/2到1的节点依次堆化，最后就会把整个切片堆化
 */
func buildHeap(m []int) *heap{
    n := len(m)-1
    for i:=n/2; i>0; i-- {
        heapf(m, n, i)
    }
    return &heap{m,n}
}
func (h *heap)Push(data int) {
    h.len++
    h.m = append(h.m, data)//向切片尾部插入数据（推断出父节点下标为i/2）
    i := h.len
    for i/2 >0 && h.m[i/2]<h.m[i] { //自下而上的堆化
        h.m[i/2], h.m[i] = h.m[i], h.m[i/2]
        i = i/2
    }
}
/**
弹出堆顶元素，为防止出现数组空洞，需要把最后一个元素放入堆顶，然后从上到下堆化
 */

func (h *heap)Pop() int{
    if h.len < 1 {
        return -1
    }
    out := h.m[1]
    h.m[1] = h.m[h.len] //把最后一个元素给堆顶
    h.len--
    //对堆顶节点进行堆化即可
    heapf(h.m, h.len, 1)
    return out
}
//对下标为i的节点进行堆化， n表示堆的最后一个节点下标
//2i,2i+1
func heapf(m []int, n,i int) {
    for {
        maxPos := i
        if 2*i<= n && m[2*i] > m[i] {
            maxPos = 2*i
        }
        if 2*i+1 <=n && m[2*i+1] > m[maxPos] {
            maxPos = 2*i + 1
        }
        if maxPos == i { //如果i节点位置正确，则退出
            break
        }
        m[i],m[maxPos] = m[maxPos],m[i]
        i = maxPos
    }
}
```

[参考文章](https://learnku.com/articles/45976)

#### 