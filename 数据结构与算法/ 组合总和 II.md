**题目大意** 

给定一个数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的每个数字在每个组合中只能使用一次。

**解题思路** 

- 题目要求出总和为 sum 的所有组合，组合需要去重。这一题是第 39 题的加强版，第 39 题中元素可以重复利用(重复元素可无限次使用)，这一题中元素只能有限次数的利用，因为存在重复元素，并且每个元素只能用一次(重复元素只能使用有限次)
- 这一题和第 47 题类似，只不过元素可以反复使用。

**代码**

```go
package leetcode

import (
 ""sort""
)

func combinationSum2(candidates []int, target int) [][]int {
 if len(candidates) == 0 {
  return [][]int{}
 }
 c, res := []int{}, [][]int{}
 sort.Ints(candidates) // 这里是去重的关键逻辑
 findcombinationSum2(candidates, target, 0, c, &res)
 return res
}

func findcombinationSum2(nums []int, target, index int, c []int, res *[][]int) {
 if target == 0 {
  b := make([]int, len(c))
  copy(b, c)
  *res = append(*res, b)
  return
 }
 for i := index; i < len(nums); i++ {
  if i > index && nums[i] == nums[i-1] { // 这里是去重的关键逻辑,本次不取重复数字，下次循环可能会取重复数字
   continue
  }
  if target >= nums[i] {
   c = append(c, nums[i])
   findcombinationSum2(nums, target-nums[i], i+1, c, res)
   c = c[:len(c)-1]
  }
 }
}
```