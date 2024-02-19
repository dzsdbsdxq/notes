**题目大意**  

给一个非空的数组，输出前 K 个频率最高的元素。

**解题思路**  

这一题是考察优先队列的题目。把数组构造成一个优先队列，输出前 K 个即可。

**代码**  

```go
package leetcode

import ""container/heap""

func topKFrequent(nums []int, k int) []int {
 m := make(map[int]int)
 for _, n := range nums {
  m[n]++
 }
 q := PriorityQueue{}
 for key, count := range m {
  heap.Push(&q, &Item{key: key, count: count})
 }
 var result []int
 for len(result) < k {
  item := heap.Pop(&q).(*Item)
  result = append(result, item.key)
 }
 return result
}

// Item define
type Item struct {
 key   int
 count int
}

// A PriorityQueue implements heap.Interface and holds Items.
type PriorityQueue []*Item

func (pq PriorityQueue) Len() int {
 return len(pq)
}

func (pq PriorityQueue) Less(i, j int) bool {
 // 注意：因为 golang 中的 heap 默认是按最小堆组织的，所以 count 越大，Less() 越小，越靠近堆顶。这里采用 >，变为最大堆
 return pq[i].count > pq[j].count
}

func (pq PriorityQueue) Swap(i, j int) {
 pq[i], pq[j] = pq[j], pq[i]
}

// Push define
func (pq *PriorityQueue) Push(x interface{}) {
 item := x.(*Item)
 *pq = append(*pq, item)
}

// Pop define
func (pq *PriorityQueue) Pop() interface{} {
 n := len(*pq)
 item := (*pq)[n-1]
 *pq = (*pq)[:n-1]
 return item
}
```