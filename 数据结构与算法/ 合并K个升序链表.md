**题目大意** 

合并 K 个有序链表

**解题思路** 

借助分治的思想，把 K 个有序链表两两合并即可。相当于是第 21 题的加强版。

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
func mergeKLists(lists []*ListNode) *ListNode {
 length := len(lists)
 if length < 1 {
  return nil
 }
 if length == 1 {
  return lists[0]
 }
 num := length / 2
 left := mergeKLists(lists[:num])
 right := mergeKLists(lists[num:])
 return mergeTwoLists1(left, right)
}

func mergeTwoLists1(l1 *ListNode, l2 *ListNode) *ListNode {
 if l1 == nil {
  return l2
 }
 if l2 == nil {
  return l1
 }
 if l1.Val < l2.Val {
  l1.Next = mergeTwoLists1(l1.Next, l2)
  return l1
 }
 l2.Next = mergeTwoLists1(l1, l2.Next)
 return l2
}
```