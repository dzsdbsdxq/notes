> **题目序号：**(267)
> **题目来源：** 大疆
> **频次:** 1

## 答案：阿纪、

1. **golang 通过结构体嵌套实现继承的特性**
   在Go语言里，没有面向对象这个概念，自然就没有继承，但它支持结构体组合；
   你可以通过在结构体内嵌套结构体实现组合；

   ```go
   type animal struct {
   	name string
   	age  string
   }
   
   type cat struct {
   	animal
   	sound string
   }
   
   type dog struct {
   	animal
   	color string
   }
   ```

2. **使用**

```go
cat := cat{
    animal: animal{
		name: "xiaomiao",
		age:  "1",
	},
    sound: "miaomiaomiao",
}
dog := dog{
    animal: animal{
		name: "dahuang",
		age:  "2",
	},
    color: "yellow",
}  
```

