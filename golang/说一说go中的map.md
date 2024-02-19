> 题目来源：微步

## 答案：重拾

一.map引用类型

1.1使用make定义map

```go
var m1 map[string]string
m1 = make(map[string]string,10)
```

1.2直接赋值的方式定义map

```go
var m4 = map[string]string{"a":"aaa"}
```

2.map的嵌套结构

```go
//方式1
students := make(map[int]map[string]string,10)
students[1] = map[string]string{
  "姓名":"张三",
}
student[2] = map[string]string{
  "姓名":"json",
}
//方式二
type s map[int]map[string]string
ss := s{
  1:{
    "姓名":"张三",
  },
  2:{
    "姓名":"json",
  },
}
```

3.map切片:make([]map[int]int,2,4)

```go
a:=make([]map[int]int,2,4)
a[0] = make(map[int]int)
a[0][1] =1
```

4.map遍历和排序

```go
map1 := map[int]string{
  l1:"测试1",
  l2:"测试2",
}
for key,item := range map1{
  fmt.Println(key,item)
}
```

#### 