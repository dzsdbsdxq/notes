> 题目序号：（1666、1707、1667）
>
> 题目来源：字节跳动
>
> 频次：3

## 答案：FH-Bin

源码包 src/runTime/string.go.stringStruct 定义了string的数据结构：

```go
Type stringStruct struct{
    str unsafe.Pointer
    len int
}
```

**数据结构：**

stringStruct.str:字符串的首地址

stringStruct.len:字符串的长度

**声明：**

如下代码所示，可以声明一个string变量赋予初值：

```go
var str string
str = "Hello world"
```

字符串构建过程是现根据字符串构建stringStruct，再转化成string。转换的源码如下：

```go
func gostringnocopy(str *byte) string{       //根据字符串地址构建string
    ss := stringStruct{str:unsafe.Pointer(str),len:findnull(str)}  // 先构造 stringStruct
    s := *(*string)(unsafe.Pointer(&ss))   //再将stringStruct 转换成string
    return s
}
```

