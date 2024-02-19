**题目大意** 

删除链表中倒数第 n 个结点。

**解题思路**

这道题比较简单，先循环一次拿到链表的总长度，然后循环到要删除的结点的前一个结点开始删除操作。需要注意的一个特例是，有可能要删除头结点，要单独处理。

这道题有一种特别简单的解法。设置 2 个指针，一个指针距离前一个指针 n 个距离。同时移动 2 个指针，2 个指针都移动相同的距离。当一个指针移动到了终点，那么前一个指针就是倒数第 n 个节点了。

**代码** 

```go
package leetcode

import (
 ""github.com/halfrost/LeetCode-Go/structures""
)

// ListNode define
type ListNode = structures.ListNode

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

// 解法一
func removeNthFromEnd(head *ListNode, n int) *ListNode {
 dummyHead := &ListNode{Next: head}
 preSlow, slow, fast := dummyHead, head, head
 for fast != nil {
  if n <= 0 {
   preSlow = slow
   slow = slow.Next
  }
  n--
  fast = fast.Next
 }
 preSlow.Next = slow.Next
 return dummyHead.Next
}

// 解法二
func removeNthFromEnd1(head *ListNode, n int) *ListNode {
 if head == nil {
  return nil
 }
 if n <= 0 {
  return head
 }
 current := head
 len := 0
 for current != nil {
  len++
  current = current.Next
 }
 if n > len {
  return head
 }
 if n == len {
  current := head
  head = head.Next
  current.Next = nil
  return head
 }
 current = head
 i := 0
 for current != nil {
  if i == len-n-1 {
   deleteNode := current.Next
   current.Next = current.Next.Next
   deleteNode.Next = nil
   break
  }
  i++
  current = current.Next
 }
 return head
}
```