```go
func main() {
arr := make([]int, 0)  
for i := 0; i < 2000; i++ { 
fmt.Println("len为", len(arr), "cap为", cap(arr))  
arr = append(arr, i) 
}
}
```

我们可以看下结果 

依次是 0,1,2,4,8,16,32,64,128,256,512,1024 

但到了1024之后,就变成了 1024,1280,1696,2304 

每次都是扩容了四分之一左右