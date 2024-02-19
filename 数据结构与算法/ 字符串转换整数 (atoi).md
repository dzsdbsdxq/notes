**题目大意** 

请你来实现一个 myAtoi(string s) 函数，使其能将字符串转换成一个 32 位有符号整数（类似 C/C++ 中的 atoi 函数）。

函数 myAtoi(string s) 的算法如下：

- 读入字符串并丢弃无用的前导空格
- 检查下一个字符（假设还未到字符末尾）为正还是负号，读取该字符（如果有）。 确定最终结果是负数还是正数。 如果两者都不存在，则假定结果为正。
- 读入下一个字符，直到到达下一个非数字字符或到达输入的结尾。字符串的其余部分将被忽略。
- 将前面步骤读入的这些数字转换为整数（即，“123” -> 123， “0032” -> 32）。如果没有读入数字，则整数为 0 。必要时更改符号（从步骤 2 开始）。
- 如果整数数超过 32 位有符号整数范围 [−231, 231 − 1] ，需要截断这个整数，使其保持在这个范围内。具体来说，小于 −231 的整数应该被固定为 −231 ，大于 231 − 1 的整数应该被固定为 231 − 1 。
- 返回整数作为最终结果。

注意：

- 本题中的空白字符只包括空格字符 ' ' 。
- 除前导空格或数字后的其余字符串外，请勿忽略 任何其他字符。

**解题思路** 

- 这题是简单题。题目要求实现类似 `C++` 中 `atoi` 函数的功能。这个函数功能是将字符串类型的数字转成 `int` 类型数字。先去除字符串中的前导空格，并判断记录数字的符号。数字需要去掉前导 `0` 。最后将数字转换成数字类型，判断是否超过 `int` 类型的上限 `[-2^31, 2^31 - 1]`，如果超过上限，需要输出边界，即 `-2^31`，或者 `2^31 - 1`。

**代码** 

```go
package leetcode

func myAtoi(s string) int {
 maxInt, signAllowed, whitespaceAllowed, sign, digits := int64(2<<30), true, true, 1, []int{}
 for _, c := range s {
  if c == ' ' && whitespaceAllowed {
   continue
  }
  if signAllowed {
   if c == '+' {
    signAllowed = false
    whitespaceAllowed = false
    continue
   } else if c == '-' {
    sign = -1
    signAllowed = false
    whitespaceAllowed = false
    continue
   }
  }
  if c < '0' || c > '9' {
   break
  }
  whitespaceAllowed, signAllowed = false, false
  digits = append(digits, int(c-48))
 }
 var num, place int64
 place, num = 1, 0
 lastLeading0Index := -1
 for i, d := range digits {
  if d == 0 {
   lastLeading0Index = i
  } else {
   break
  }
 }
 if lastLeading0Index > -1 {
  digits = digits[lastLeading0Index+1:]
 }
 var rtnMax int64
 if sign > 0 {
  rtnMax = maxInt - 1
 } else {
  rtnMax = maxInt
 }
 digitsCount := len(digits)
 for i := digitsCount - 1; i >= 0; i-- {
  num += int64(digits[i]) * place
  place *= 10
  if digitsCount-i > 10 || num > rtnMax {
   return int(int64(sign) * rtnMax)
  }
 }
 num *= int64(sign)
 return int(num)
}
```