扩容会发生在slice append的时候，当slice的cap不足以容纳新元素，就会进行扩容，扩容规则如下

- 如果新申请容量比两倍原有容量大，那么扩容后容量大小 为 新申请容量
- 如果原有 slice 长度小于 1024， 那么每次就扩容为原来的 2 倍
- 如果原 slice 长度大于等于 1024， 那么每次扩容就扩为原来的 1.25 倍

```go
func main() {
slice1 := []int{1, 2, 3}
for i := 0; i < 16; i++ {
slice1 = append(slice1, 1)
fmt.Printf("addr: %p, len: %v, cap: %v
", slice1, len(slice1), cap(slice1))
}
}
```

```
addr: 0xc00001a120, len: 4, cap: 6
addr: 0xc00001a120, len: 5, cap: 6
addr: 0xc00001a120, len: 6, cap: 6
addr: 0xc000060060, len: 7, cap: 12
addr: 0xc000060060, len: 8, cap: 12
addr: 0xc000060060, len: 9, cap: 12
addr: 0xc000060060, len: 10, cap: 12
addr: 0xc000060060, len: 11, cap: 12
addr: 0xc000060060, len: 12, cap: 12
addr: 0xc00007c000, len: 13, cap: 24
addr: 0xc00007c000, len: 14, cap: 24
addr: 0xc00007c000, len: 15, cap: 24
addr: 0xc00007c000, len: 16, cap: 24
addr: 0xc00007c000, len: 17, cap: 24
addr: 0xc00007c000, len: 18, cap: 24
addr: 0xc00007c000, len: 19, cap: 24
```