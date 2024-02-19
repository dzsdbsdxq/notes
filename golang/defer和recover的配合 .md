> 题目序号：4562
>
> 题目来源：京东
>
> 频次：3

## 答案：陆户习习门

defer内建函数,延迟调用，所在函数退出时调用，一个方法里若有多个defer语句，则先声明的后被调用，一般与recover()函数一起配合使用，recover()一般用于捕捉panic抛出的异常，比如：panic(11), 捕捉到的就是11 

```go
func main() {

	defer func() {
		if v := recover();v == 11 {
			fmt.Printf("v: %#v
",v)
		}
		fmt.Printf("defer1...
")
	}()

	defer func() {
		fmt.Printf("defer2...
")
	}()

	array := [2]int{1,2}
	fmt.Println("array: ",array[1])
	panic(11)

	/*输出：
	array:  2
	defer2...
	v: 11
	defer1...
	*/
}
```

