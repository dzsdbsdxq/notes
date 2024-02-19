> **题目序号：**（2816）
> **题目来源：** 哔哩哔哩
> **频次:**  1

## 答案: 小强

尾插法 不断的将新节点更新为最后一个节点

 ```go
type LinkNode struct {
	Data int
	Next *LinkNode
}

func CreateList(datas []int) *LinkNode {
	if len(datas) == 0 {
		return nil
	}

	head := new(LinkNode)
	tail := head
	for i := 0; i < len(datas); i++ {
	    tNode := new(LinkNode)
	    tNode.Data = datas[i]
	    tail.Next = tNode
	    tail = tNode
	}
	tail.Next = nil

	return head
}
 ```

#### 