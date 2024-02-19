> **题目序号：**(266)
> **题目来源：** 大疆
> **频次:** 1

## 答案：阿纪、

1. **方法**
   go语言删除切片元素的方法：
   1、指定删除位置，如【index := 1】;
   2、查看删除位置之前的元素和之后的元素;
   3、将删除点前后的元素连接起来即可。
   Go 语言并没有对删除切片元素提供专用的语法或者接口，需要使用切片本身的特性来删除元素。
   示例代码如下：

   ```go
    str := []string{"a","b","c"}
       // step 1
       index := 1
       // step 2
       fmt.Println(str[:index], str[index+1])
       // step 3
       str = append(str[:index], str[index+1]...)
       // res
       fmt.Println(str)
   ```

   