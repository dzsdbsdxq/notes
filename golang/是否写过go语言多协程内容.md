> 题目来源：米哈游

作者：ORVR

是否写过go语言多协程内容

协程池用法简单举例

```go
var (
    ctx = gctx.New()
)

func main() {
    wg := sync.WaitGroup{}
    for i := 0; i < 10; i++ {
        wg.Add(1)
        v := i
        grpool.Add(ctx, func(ctx context.Context) {
            fmt.Println(v)
            wg.Done()
        })
    }
    wg.Wait()
}
```
