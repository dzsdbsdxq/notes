**题目大意** 

给定一个可能包含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。说明：解集不能包含重复的子集。

**解题思路**  

- 这一题是第 78 题的加强版，比第 78 题多了一个条件，数组中的数字会出现重复。
- 解题方法依旧是 DFS，需要在回溯的过程中加上一些判断。
- 这一题和第 78 题，第 491 题类似，可以一起解答和复习。

**代码**  

```go
package leetcode

import (
 ""fmt""
 ""sort""
)

func subsetsWithDup(nums []int) [][]int {
 c, res := []int{}, [][]int{}
 sort.Ints(nums) // 这里是去重的关键逻辑
 for k := 0; k <= len(nums); k++ {
  generateSubsetsWithDup(nums, k, 0, c, &res)
 }
 return res
}

func generateSubsetsWithDup(nums []int, k, start int, c []int, res *[][]int) {
 if len(c) == k {
  b := make([]int, len(c))
  copy(b, c)
  *res = append(*res, b)
  return
 }
 // i will at most be n - (k - c.size()) + 1
 for i := start; i < len(nums)-(k-len(c))+1; i++ {
  fmt.Printf(""i = %v start = %v c = %v
"", i, start, c)
  if i > start && nums[i] == nums[i-1] { // 这里是去重的关键逻辑,本次不取重复数字，下次循环可能会取重复数字
   continue
  }
  c = append(c, nums[i])
  generateSubsetsWithDup(nums, k, i+1, c, res)
  c = c[:len(c)-1]
 }
 return
}
```