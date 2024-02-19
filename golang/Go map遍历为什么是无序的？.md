使用 range 多次遍历 map 时输出的 key 和 value 的顺序可能不同。这是 Go 语言的设计者们**有意为之**，旨在提示开发者们，Go 底层实现并不保证 map 遍历顺序稳定，请大家不要依赖 range 遍历结果顺序

主要原因有2点：

- map在遍历时，并不是从固定的0号bucket开始遍历的，每次遍历，都会从一个**随机值序号的bucket**，再从其中**随机的cell**开始遍历
- map遍历时，是按序遍历bucket，同时按需遍历bucket中和其overflow bucket中的cell。但是map在扩容后，会发生key的搬迁，这造成原来落在一个bucket中的key，搬迁后，有可能会落到其他bucket中了，从这个角度看，遍历map的结果就不可能是按照原来的顺序了

map 本身是无序的，且遍历时顺序还会被随机化，如果想顺序遍历 map，需要对 map key 先排序，再按照 key 的顺序遍历 map。

```go
func TestMapRange(t *testing.T) {
m := map[int]string{1: "a", 2: "b", 3: "c"}
t.Log("first range:")
for i, v := range m {
t.Logf("m[%v]=%v ", i, v)
}
t.Log("
second range:")
for i, v := range m {
t.Logf("m[%v]=%v ", i, v)
}

// 实现有序遍历
var sl []int
// 把 key 单独取出放到切片
for k := range m {
sl = append(sl, k)
}
// 排序切片
sort.Ints(sl)
// 以切片中的 key 顺序遍历 map 就是有序的了
for _, k := range sl {
t.Log(k, m[k])
}
}
```