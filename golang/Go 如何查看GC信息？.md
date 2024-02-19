### 1. GODEBUG='gctrace=1'

```
package main
func main() {
for n := 1; n < 100000; n++ {
_ = make([]byte, 1<<20)
}
}
```

```
$ GODEBUG='gctrace=1' go run main.go

gc 1 @0.003s 4%: 0.013+1.7+0.008 ms clock, 0.10+0.67/1.2/0.018+0.064 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 2 @0.006s 2%: 0.006+4.5+0.058 ms clock, 0.048+0.070/0.027/3.6+0.47 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 3 @0.011s 3%: 0.021+1.3+0.009 ms clock, 0.17+0.041/0.41/0.046+0.072 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 4 @0.013s 5%: 0.025+0.38+0.26 ms clock, 0.20+0.054/0.15/0.009+2.1 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 5 @0.014s 5%: 0.021+0.16+0.002 ms clock, 0.17+0.098/0.028/0.001+0.016 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 6 @0.014s 7%: 0.025+1.6+0.003 ms clock, 0.20+0.061/2.9/1.5+0.025 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 7 @0.016s 7%: 0.019+1.0+0.002 ms clock, 0.15+0.053/1.0/0.018+0.017 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 8 @0.017s 7%: 0.029+0.17+0.002 ms clock, 0.23+0.037/0.10/0.063+0.022 ms cpu, 4->4->0 MB, 5 MB goal, 8 P
gc 9 @0.018s 7%: 0.019+0.23+0.002 ms clock, 0.15+0.040/0.16/0.023+0.018 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 10 @0.018s 7%: 0.022+0.23+0.003 ms clock, 0.17+0.061/0.13/0.006+0.024 ms cpu, 4->6->2 MB, 5 MB goal, 8 P
gc 11 @0.018s 7%: 0.019+0.11+0.001 ms clock, 0.15+0.033/0.051/0.013+0.015 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 12 @0.019s 7%: 0.018+0.19+0.001 ms clock, 0.14+0.035/0.10/0.018+0.014 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 13 @0.019s 7%: 0.018+0.35+0.002 ms clock, 0.15+0.21/0.054/0.013+0.016 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 14 @0.019s 8%: 0.024+0.27+0.002 ms clock, 0.19+0.022/0.13/0.014+0.017 ms cpu, 4->5->1 MB, 5 MB goal, 8 P
gc 15 @0.020s 8%: 0.019+0.42+0.038 ms clock, 0.15+0.060/0.28/0.007+0.31 ms cpu, 4->17->13 MB, 5 MB goal, 8 P
gc 16 @0.021s 8%: 0.018+0.53+0.060 ms clock, 0.14+0.045/0.39/0.005+0.48 ms cpu, 21->28->7 MB, 26 MB goal, 8 P
gc 17 @0.021s 10%: 0.020+0.91+0.64 ms clock, 0.16+0.050/0.36/0.027+5.1 ms cpu, 12->16->4 MB, 14 MB goal, 8 P
gc 18 @0.023s 10%: 0.020+0.55+0.002 ms clock, 0.16+0.053/0.50/0.081+0.023 ms cpu, 7->9->2 MB, 8 MB goal, 8 P
```

字段含义由下表所示：

| 字段  | 含义                                           |
| :---- | :--------------------------------------------- |
| gc 2  | 第二个 GC 周期                                 |
| 0.006 | 程序开始后的 0.006 秒                          |
| 2%    | 该 GC 周期中 CPU 的使用率                      |
| 0.006 | 标记开始时， STW 所花费的时间（wall clock）    |
| 4.5   | 标记过程中，并发标记所花费的时间（wall clock） |
| 0.058 | 标记终止时， STW 所花费的时间（wall clock）    |
| 0.048 | 标记开始时， STW 所花费的时间（cpu time）      |
| 0.070 | 标记过程中，标记辅助所花费的时间（cpu time）   |
| 0.027 | 标记过程中，并发标记所花费的时间（cpu time）   |
| 3.6   | 标记过程中，GC 空闲的时间（cpu time）          |
| 0.47  | 标记终止时， STW 所花费的时间（cpu time）      |
| 4     | 标记开始时，堆的大小的实际值                   |
| 5     | 标记结束时，堆的大小的实际值                   |
| 1     | 标记结束时，标记为存活的对象大小               |
| 5     | 标记结束时，堆的大小的预测值                   |
| 8     | P 的数量                                       |

### 2. go tool trace

```
package main

import (
"os"
"runtime/trace"
)

func main() {
f, _ := os.Create("trace.out")
defer f.Close()
trace.Start(f)
defer trace.Stop()
for n := 1; n < 100000; n++ {
_ = make([]byte, 1<<20)
}
}
```

```
$ go run main.go
$ go tool trace trace.out
```

打开浏览器后，可以看到如下统计：

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220504204708533.png)

点击View trace，可以查看当时的trace情况

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/f3d74b546b1e360c9d6946757ada4f64.png)

点击 Minimum mutator utilization，可以查看到赋值器 mutator （用户程序）对 CPU 的利用率 74.1%，接近100%则代表没有针对GC的优化空间了

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/imgcdn/image-20220504204751752.png)

### 3. debug.ReadGCStats

```
package main

import (
"fmt"
"runtime/debug"
"time"
)

func printGCStats() {
t := time.NewTicker(time.Second)
s := debug.GCStats{}
for {
select {
case <-t.C:
debug.ReadGCStats(&s)
fmt.Printf("gc %d last@%v, PauseTotal %v\n", s.NumGC, s.LastGC, s.PauseTotal)
}
}
}
func main() {
go printGCStats()
for n := 1; n < 100000; n++ {
_ = make([]byte, 1<<20)
}
}
```

```
$ go run main.go

gc 3392 last@2022-05-04 19:22:52.877293 +0800 CST, PauseTotal 117.524907ms
gc 6591 last@2022-05-04 19:22:53.876837 +0800 CST, PauseTotal 253.254996ms
gc 10028 last@2022-05-04 19:22:54.87674 +0800 CST, PauseTotal 376.981595ms
gc 13447 last@2022-05-04 19:22:55.87689 +0800 CST, PauseTotal 511.420111ms
gc 16938 last@2022-05-04 19:22:56.876955 +0800 CST, PauseTotal 649.293449ms
gc 20350 last@2022-05-04 19:22:57.876756 +0800 CST, PauseTotal 788.003014ms
```

字段含义由下表所示：

| 字段       | 含义       |
| :--------- | :--------- |
| NumGC      | GC总次数   |
| LastGC     | 上次GC时间 |
| PauseTotal | STW总耗时  |

### 4. runtime.ReadMemStats

```
package main

import (
"fmt"
"runtime"
"time"
)

func printMemStats() {
t := time.NewTicker(time.Second)
s := runtime.MemStats{}
for {
select {
case <-t.C:
runtime.ReadMemStats(&s)
fmt.Printf("gc %d last@%v, heap_object_num: %v, heap_alloc: %vMB, next_heap_size: %vMB\n",
s.NumGC, time.Unix(int64(time.Duration(s.LastGC).Seconds()), 0), s.HeapObjects, s.HeapAlloc/(1<<20), s.NextGC/(1<<20))
}
}
}
func main() {
go printMemStats()
fmt.Println(1 << 20)
for n := 1; n < 100000; n++ {
_ = make([]byte, 1<<20)
}
}
```

```
$ go run main.go

gc 2978 last@2022-05-04 19:38:04 +0800 CST, heap_object_num: 391, heap_alloc: 20MB, next_heap_size: 28MB
gc 5817 last@2022-05-04 19:38:05 +0800 CST, heap_object_num: 370, heap_alloc: 4MB, next_heap_size: 4MB
gc 9415 last@2022-05-04 19:38:06 +0800 CST, heap_object_num: 392, heap_alloc: 7MB, next_heap_size: 8MB
gc 11429 last@2022-05-04 19:38:07 +0800 CST, heap_object_num: 339, heap_alloc: 4MB, next_heap_size: 5MB
gc 14706 last@2022-05-04 19:38:08 +0800 CST, heap_object_num: 436, heap_alloc: 6MB, next_heap_size: 8MB
gc 18253 last@2022-05-04 19:38:09 +0800 CST, heap_object_num: 375, heap_alloc: 4MB, next_heap_size: 6M
```

字段含义由下表所示：

| 字段        | 含义                                                         |
| :---------- | :----------------------------------------------------------- |
| NumGC       | GC总次数                                                     |
| LastGC      | 上次GC时间                                                   |
| HeapObjects | 堆中已经分配的对象总数，GC内存回收后HeapObjects取值相应减小  |
| HeapAlloc   | 堆中已经分配给对象的字节数，GC内存回收后HeapAlloc取值相应减小 |
| NextGC      | 下次GC目标堆的大小                                           |