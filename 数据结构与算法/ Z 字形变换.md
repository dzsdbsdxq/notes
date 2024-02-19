**题目大意** 

将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `""PAYPALISHIRING""` 行数为 3 时，排列如下：

```go
P   A   H   N
A P L S I I G
Y   I   R
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`""PAHNAPLSIIGYIR""`。

请你实现这个将字符串进行指定行数变换的函数：

```go
string convert(string s, int numRows);
```

**解题思路** 

- 这一题没有什么算法思想，考察的是对程序控制的能力。用 2 个变量保存方向，当垂直输出的行数达到了规定的目标行数以后，需要从下往上转折到第一行，循环中控制好方向ji

**代码** 

```go
package leetcode

func convert(s string, numRows int) string {
 matrix, down, up := make([][]byte, numRows, numRows), 0, numRows-2
 for i := 0; i != len(s); {
  if down != numRows {
   matrix[down] = append(matrix[down], byte(s[i]))
   down++
   i++
  } else if up > 0 {
   matrix[up] = append(matrix[up], byte(s[i]))
   up--
   i++
  } else {
   up = numRows - 2
   down = 0
  }
 }
 solution := make([]byte, 0, len(s))
 for _, row := range matrix {
  for _, item := range row {
   solution = append(solution, item)
  }
 }
 return string(solution)
}
```