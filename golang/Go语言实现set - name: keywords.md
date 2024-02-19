> **题目序号：**(3248)
> **题目来源：** 腾讯
> **频次:** 2

## 答案：重拾

```go
package set

import (
	"bytes"
	"fmt"
)

func main() {

}

type HashSet struct {
	m map[interface{}]bool
}

func NewHashSet() *HashSet {
	return &HashSet{m: make(map[interface{}]bool)}
}

//添加    true 添加成功 false 添加失败
func (set *HashSet) Add(e interface{}) (b bool) {
	if !set.m[e] {
		set.m[e] = true
		return true
	}
	return false
}

//删除
func (set *HashSet) Remove(e interface{}) {
	delete(set.m, e)
}

//清除
func (set *HashSet) Clear() {
	set.m = make(map[interface{}]bool)
}

//是否包含
func (set *HashSet) Contains(e interface{}) bool {
	return set.m[e]
}

//获取元素数量
func (set *HashSet) Len() int {
	return len(set.m)
}

//判断两个set时候相同
//true 相同 false 不相同
func (set *HashSet) Same(other *HashSet) bool {
	if other == nil {
		return false
	}

	if set.Len() != other.Len() {
		return false
	}

	for k, v := range set.m {
		if !other.Contains(k) {
			return false
		}
	}

	return true
}

//迭代
func (set *HashSet) Elements() []interface{} {
	initlen := len(set.m)

	snaphot := make([]interface{}, initlen)

	actuallen := 0

	for k, v := range set.m {
		if actuallen < initlen {
			snaphot[actuallen] = k
		} else {
			snaphot = append(snaphot, k)
		}
		actuallen++
	}

	if actuallen < initlen {
		snaphot = snaphot[:actuallen]
	}

	return snaphot
}

//获取自身字符串
func (set *HashSet) String() string {
	var buf bytes.Buffer

	buf.WriteString("set{")

	first := true

	for k, v := range set.m {
		if first {
			first = false
		} else {
			buf.WriteString(" ")
		}

		buf.WriteString(fmt.Sprintf("%v", k))
	}

	buf.WriteString("}")

	return buf.String()
}
```

