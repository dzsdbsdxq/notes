> 题目序号：（3240）
>
> 题目来源：腾讯
>
> 频次：1

答案1：（One）

1. 不同类型的 struct 之间不能进行比较，编译期就会报错（GoLand 会直接提示）

2. 同类型的 struct 也分为两种情况，

   - struct 的所有成员都是可以比较的，则该 strcut 的不同实例可以比较

   - struct 中含有不可比较的成员（如 Slice），则该 struct 不可以比较

   **同类型 struct 比较**

```go
import "fmt"

type A struct {
	age  int
	name string
}

func StructCompare1() {
	aObj1 := A{
		age:  13,
		name: "张三",
	}
	aObj2 := A{
		age:  13,
		name: "张三",
	}
	fmt.Println(aObj1 == aObj2) // true

	aObj3 := &A{
		age:  13,
		name: "张三",
	}
	aObj4 := &A{
		age:  13,
		name: "张三",
	}

	fmt.Println(aObj3 == aObj4) // false

	var aObj5 A
	fmt.Println(aObj5) //{0 } ，未明确初始化时，struct 实例的成员取各自的零值
	//fmt.Println( aObj5 == nil)  // 报错，无法将 nil 转换为类型 A

	var aObj6 *A
	fmt.Println(aObj6)        // <nil> ，指针类型数据的零值为 nil
	fmt.Println(aObj6 == nil) //  true，指针类型的数据可以和 nil 比较
}

```

**struct 包含不可比较的成员,如map，slice**

![image-20220417213758596](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417213758596.png)

**不同类型 struct 不能比较**

![image-20220417213820375](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220417213820375.png)

