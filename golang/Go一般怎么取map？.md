> 题目来源：京东

答案：古尔班通

Go语言的map底层使用Hash表实现，map分别支持字面量初始化和内置函数make()初始化。
获取map中不存在键的值不会发生异常，而是会返回值类型的零值，如果想确定map中是否存在key，则可以使用获取map值的comma,ok表达式语法。

```go
import "fmt"
func main()  {
   m := make(map[string]string)

   v,ok := m["test"]
   //通过ok进行判断
   if !ok{
      fmt.Println("m[test] is nil")
   }else {
      fmt.Println("m[test] =",v)
   }
}
```

map操作不是原子的，所以避免并发读写map。若需要并发读写，则可以使用额外的锁(互斥锁、读写锁)，也可以考虑使用标准库sync包中的sync.map。