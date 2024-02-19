> 题目序号：（5896） 
>
> 题目来源：腾讯音乐
>
> 频次:  1

**答案：**Zbbxd
map是引用类型的, map取一个key，然后修改这个值，原map数据的值也会变化。

元素value类型为int ， string

```go
	// 初始化一个int值的map并赋值
	m2 := map[string]int{}
	m2["A"] = 1
	m2["B"] = 2
	fmt.Println(m2) // map[A:1 B:2]
	m2["A"] = 3
	fmt.Println(m2) // map[A:3 B:2]
 
	// 初始化一个string值的map并赋值
	m3 := map[string]string{}
	m3["a"] = ""
	m3["b"] = "b"
	fmt.Println(m3) // map[a: b:b]
	m3["a"] = "a"
	fmt.Println(m3) // map[a:a b:b]
```

元素value类型为struct

```go
    type data struct {
	    name   string
	    gender string
    }
 
    // 初始化一个struct值的map并赋值
	m := map[string]data{"x": {"value1", "F"}}
	// 当key不存在map中的时候为新增，当key存在于map的时候为修改，覆盖旧值
	m["y"] = data{"value2", "M"} // 增加新键值对
	m["z"] = data{"value3", "M"} // 增加新键值对
	fmt.Println(m)               // map[x:{value1 F} y:{value2 M} z:{value3 M}]

//map元素是无法取址的，不可以m["x"].name来直接修改

// 1，修改key为y的值,只修改gender字段
	m["y"] = data{gender: "F"} // map[x:{value1} y:{new value}]
	// 修改key为z的name,构造新结构体赋值
	m["z"] = data{name: "z's name", gender: "F"}
	fmt.Println(m) // map[x:{value1 F} y:{ F} z:{z's name F}]
 
	// 2，将map的value设为指针类型（结构体较大时最优）
	mm := map[string] *data{}
	mm["a"] = &data{gender:"F",name:"name"}
	fmt.Println(mm["a"])	// &{name F}
	// 修改name字段为name1
	mm["a"].name = "name1"
	fmt.Println(mm["a"])	// &{name1 F}
    // 注意下面一行操作，key为new的元素不存在，此时mm["new"]是空引用，不能.name
    mm["new"].name = "new"	// panic: runtime error: invalid memory address or nil pointer dereference
    // 可以这样来
	mm["new"] = &data{name:"new",gender:"new"}
	for key, value := range mm {fmt.Printf("m[%s] = %s \t", key, value)}	//m[a] = &{name1 F} 	m[new] = &{new new}
 
	// 3，以第三方变量的方式，如想修改key为x的元素值，原元素为"x": {"value1", "F"},修改F为M
	data := m["x"]
	data.gender = "M"
	m["x"] = data
	fmt.Println(m) // map[x:{value1 M} y:{ F} z:{z's name F}]
	
```
