> 题目来源：百度

答案：斯鱼

使用切片slice，存储key值，sort排序，按key值访问map中的值；

```go
import "sort"

var m map[string]string
var keys []string
for k := range m {
    keys = append(keys, k)
}
sort.Strings(keys)
for _, k := range keys {
    fmt.Println("Key:", k, "Value:", m[k]
}
```
