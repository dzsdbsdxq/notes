> **题目序号：**314
>
> **题目来源**：
>
> **频次**：1

**答案1：**（树枝）

**go map 并发为什么不是安全的？**

熟悉Go语言的人或多或少都听过Rob Pike的这句话` Do not communicate by sharing memory; instead, share memory by communicating.`这句话的意思是：` 不要以共享内存的方式来通讯，相反，要经过通讯来共享内存 `这也很好的解释了为什么Go不把map设计为并发安全的。

但是在生产环境中遇到需要对map大量的并发写入与读取，如何保证map的并发安全呢？

~~~ go
func main() {
	testMap := make(map[int]int)
	for i := 0; i < 100; i++ {
		go func(i int) {
			testMap[i+1] = i
		}(i)
	}
	for i := 0; i < 100; i++ {
		go func(i int) {
			fmt.Println("对map取值：",testMap[i+1])
		}(i)
	}
	time.Sleep(time.Second)
}
~~~

直接对map进行并发读写会painc：concurrent map writes

保证map并发安全的两种方法

1. **使用读写锁**

   ~~~ go
   func main() {
   	var lock sync.RWMutex
   	testMap := make(map[int]int)
   	for i := 0; i < 100; i++ {
   		go func(i int) {
   			lock.Lock()
   			testMap[i+1] = i
   			lock.Unlock()
   		}(i)
   	}
   	for i := 0; i < 100; i++ {
   		go func(i int) {
   			lock.RLock()
   			fmt.Println("对map取值：",testMap[i+1])
   			lock.RUnlock()
   		}(i)
   	}
   	time.Sleep(time.Second)
   }
   
   ~~~

2. **使用sync.Map**

   ~~~ go
   func main() {
   	var m sync.Map
   	for i := 0; i < 100; i++ {
   		go func(i int) {
   			m.Store(i+1, i)
   		}(i)
   	}
   	for i := 0; i < 100; i++ {
   		go func(i int) {
   			v, _ := m.Load(i + 1)
   			fmt.Printf("使用map的key：%d,取到map的value:%d
", i+1, v)
   		}(i + 1)
   	}
   	time.Sleep(time.Second)
   }
   
   ~~~

