> 题目来源：腾讯
>
> 频次：高频

## 答案：Evan.C

- sync.Map底层也是锁，进行了读写分离

```go
type Map struct {
   mu Mutex
   read atomic.Value // readOnly
   dirty map[interface{}]*entry
   misses int
}
```

- `read`包含对并发访问安全的map内容的部分(无论是否持有`mu`)。
- `Dirty`包含map内容中需要保存`mu`的部分。
- `misses`计算自从上次读取map更新后，需要锁定`mu`来确定key是否存在的加载次数。