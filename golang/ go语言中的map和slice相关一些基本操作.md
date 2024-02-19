> 题目序号：（2338）
>
> 题目来源：奇安信
>
> 频次：1

## 答案1：（ORVR）

**slice 使用**

```go
package main

import "fmt"

func main() {

    //在go语言中没有所谓的动态数组，所以就有了切片
    //切片使用的三中方式
    //第一种
    var intArray [5]int = [...]int{1, 2, 3, 4, 5}
    slice := intArray[2:4]
    fmt.Println(slice)
    fmt.Println(intArray)
    fmt.Println(len(slice))

    //第二种
    //var Array[]int(类型）=make([](类型）,len,cap)
    var Array []int = make([]int, 4, 8)
    Array[2] = 33 //没有定义的都为0
    fmt.Printf("%v\n", Array[2])
    fmt.Println(Array)

    //第三种
    var strSlice []string = []string{"tom", "alice", "jake"}
    fmt.Println(strSlice)

    //切片的遍历
    for i, v := range strSlice {
        fmt.Printf("第%v个元素：%v\n", i, v)
    }

    strSlice1 := strSlice[1:2]
    fmt.Println(strSlice1)
    strSlice1[0] = "alex" //主要因为strSlice1是从strSlice中切出来的时，改变strSlice1也会改变strSlice(引用）
    fmt.Println(strSlice)

    //append可以给切片加上元素
    strSlice = append(strSlice, "bier")
    fmt.Println(strSlice)

    //加上切片，注意加上...
    strSlice = append(strSlice, strSlice...)
    fmt.Println(strSlice)

    //切片的拷贝
    var strSlice3 []string = make([]string, 10)
    copy(strSlice3, strSlice)
    fmt.Println(strSlice3)

    //拷贝后，修改原来的值不会对拷贝的结果产生影响
    strSlice[3] = "小明"
    fmt.Println(strSlice)
    fmt.Println(strSlice3)

    //string进行切片处理
    str := "198abcdefg"
    slice0 := str[3:]
    fmt.Println(slice0)

    //注意字符串是不能修改的，但是可以通过切片方式改变字符串
    str1 := []byte(str)
    str1[0] = '8'
    str = string(str1)
    fmt.Println(str)

    //有汉字时

    str2 := []rune(str)
    str2[0] = '好'
    str = string(str2)
    fmt.Println(str)

}
```

**map使用**

```go
func main() {

	//map的使用，注意map声明时没有分配空间，所以要用make进行分配
	var a map[string]string
	a = make(map[string]string, 10)
	a["No.1"] = "A+"
	a["No.2"] = "B"
	a["No.3"] = "C"
	a["No.1"] = "A"
	fmt.Println(a)
	//map声明的其他方式

	//二
	b := make(map[string]string)
	b["一"] = "太阳"
	b["二"] = "月亮"
	b["三"] = "星星"
	fmt.Println(b)

	//三
	c := map[string]string{
		"one": "广东",
		"two": "北京",
		"three": "上海",
	}
	fmt.Println(c)
	name := make(map[string]string)
	name["1"] = "小明"
	name["2"] = "小张" //这里相当于在增加了
	name["3"] = "小王"
	fmt.Println(name)

	//如果key没有的话，就是增加，如果有的话就是修改
	name["1"] = "小林"
	fmt.Println(name)
	//对map进行删除
	delete(name, "2")
	fmt.Println(name)
	//map的查询
	val, ok := name["3"]
	//如果有的话，ok值为true，没有为false
	if ok {
		fmt.Printf("存在 %v\n", val)
	} else {
		fmt.Println("不存在")
	}
}
```

