**题目大意** 

实现一个查找 substring 的函数。如果在母串中找到了子串，返回子串在母串中出现的下标，如果没有找到，返回 -1，如果子串是空串，则返回 0 。

**解题思路** 

这一题比较简单，直接写即可。

**代码** 

```go
package leetcode

import ""strings""

// 解法一
func strStr(haystack string, needle string) int {
 for i := 0; ; i++ {
  for j := 0; ; j++ {
   if j == len(needle) {
    return i
   }
   if i+j == len(haystack) {
    return -1
   }
   if needle[j] != haystack[i+j] {
    break
   }
  }
 }
}

// 解法二
func strStr1(haystack string, needle string) int {
 return strings.Index(haystack, needle)
}
```