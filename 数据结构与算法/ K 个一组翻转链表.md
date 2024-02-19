**题目大意** 

按照每 K 个元素翻转的方式翻转链表。如果不满足 K 个元素的就不翻转。

**解题思路** 

这一题是 problem 24 的加强版，problem 24 是两两相邻的元素，翻转链表。而 problem 25 要求的是 k 个相邻的元素，翻转链表，problem 相当于是 k = 2 的特殊情况。

**代码** 

```go
package leetcode

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseKGroup(head *ListNode, k int) *ListNode {
 node := head
 for i := 0; i < k; i++ {
  if node == nil {
   return head
  }
  node = node.Next
 }
 newHead := reverse(head, node)
 head.Next = reverseKGroup(node, k)
 return newHead
}

func reverse(first *ListNode, last *ListNode) *ListNode {
 prev := last
 for first != last {
  tmp := first.Next
  first.Next = prev
  prev = first
  first = tmp
 }
 return prev
}
```