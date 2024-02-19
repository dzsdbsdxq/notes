将整数转换为浮点数。 Go 支持显式类型转换以满足其严格的类型要求。 

```go
i := 55 //int  
j := 67.8 //float64  
sum := i + int(j) //j is converted to int
```