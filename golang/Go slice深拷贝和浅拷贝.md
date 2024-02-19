
深拷贝：拷贝的是数据本身，创造一个新对象，新创建的对象与原对象不共享内存，新创建的对象在内存中开辟一个新的内存地址，新对象值修改时不会影响原对象值

实现深拷贝的方式：

1. copy(slice2, slice1) 

2. 遍历append赋值

```go
func main() {
slice1 := []int{1, 2, 3, 4, 5}
slice2 := make([]int, 5, 5)
fmt.Printf("slice1: %v, %p
", slice1, slice1)
copy(slice2, slice1)
fmt.Printf("slice2: %v, %p
", slice2, slice2)
slice3 := make([]int, 0, 5)
for _, v := range slice1 {
slice3 = append(slice3, v)
}
fmt.Printf("slice3: %v, %p
", slice3, slice3)
}

slice1: [1 2 3 4 5], 0xc0000b0030
slice2: [1 2 3 4 5], 0xc0000b0060
slice3: [1 2 3 4 5], 0xc0000b0090
```

浅拷贝：拷贝的是数据地址，只复制指向的对象的指针，此时新对象和老对象指向的内存地址是一样的，新对象值修改时老对象也会变化

实现浅拷贝的方式：

引用类型的变量，默认赋值操作就是浅拷贝

1. slice2 := slice1

```go
func main() {
slice1 := []int{1, 2, 3, 4, 5}
fmt.Printf("slice1: %v, %p
", slice1, slice1)
slice2 := slice1
fmt.Printf("slice2: %v, %p
", slice2, slice2)
}

slice1: [1 2 3 4 5], 0xc00001a120
slice2: [1 2 3 4 5], 0xc00001a120
```