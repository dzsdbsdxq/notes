> 题目来源：微步

## 答案：重拾

1.map类似其他语言中的哈希表或字典，以key-value形式存储数据

2.key必须是支持==或!=比较运算的类型，不可以是函数、map或slice

3.map通过key查找value比线性搜索快很多。

4.map使用make()创建，支持:=这种简写方式

5.超出容量时会自动扩容，

6.当键值对不存在时自动添加，使用delete()删除某键值对

**缺点：**

并发中的map不是安全的

```go
//运行下边代码会报错,原因是并发的去读写map结构的数据了
func main(){
  test := map[int]int{1:1}
  go func(){
    i:=0
    for i<10000{
      test[i] =1
      i++
      
    }()
    go func(){
      i :=0
      for i<10000{
        test[i] = 1
        i++
      }
    }()
    time.Sleep(2*time.Second)
    fmt.Println(test)
  }
}

//解决的方法是加锁

func main(){
  test := map[int]int{1:1}
  var s sync.RWMutex
  go func(){
    i:=0
    for i<10000{
      s.Lock()
      test[i] =1
      s.Unlock()
      i++
      
    }()
    go func(){
      i :=0
      for i<10000{
      s.Lock()
        test[i] = 1
        s.Unlock()
        i++
      }
    }()
    time.Sleep(2*time.Second)
    fmt.Println(test)
  }
}

//另一种方法
func main(){
  test := sync.Map{}
  test.Store(1,1)
  go func(){
    i:=0
    for i<10000{
      test.Store(1,1)
      i++
    }
  }()
  go func(){
    i:=0
    for i<10000{
      test.Store(1,1)
      i++
    }
  }()
  time.Sleep(time.Second)
  fmt.Println(test.Load(1))
}
```

#### 