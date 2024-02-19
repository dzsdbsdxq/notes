> 题目来源：好未来

答案：T

参考：https://cloud.tencent.com/developer/article/1820718

```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"strconv"
)

// 爬取单个页面的函数
func SpiderPage(i int, page chan int) {
	url := "https://tieba.baidu.com/f?kw=%E7%BB%9D%E5%9C%B0%E6%B1%82%E7%94%9F&ie=utf-8&pn=" + strconv.Itoa((i-1)*50)
	result, err := HttpGet(url)
	if err != nil {
		fmt.Println("httpGet err", err)
		return
	}
	//fmt.Println("result=", result)
	f, err := os.Create("第" + strconv.Itoa(i) + "页" + ".html")
	if err != nil {
		fmt.Println("Create err", err)
		return
	}
	f.WriteString(result)
	f.Close() // 保存好一个文件，关闭一个文件,不要用defer

	page <- i //与主go程完成同步
}

func working(start int, end int) {
	fmt.Printf("正在爬取第%d页到%d页", start, end)

	page := make(chan int)
	// 循环读取页数
	for i := start; i <= end; i++ {
		go SpiderPage(i, page)
	}
    // 如果不使用channel会直接让主go程退出，所以使用无缓存的channel进行阻塞
	for i := start; i <= end; i++ {
		fmt.Printf("第%d个页面爬取完成", <-page)
	}

}

func HttpGet(url string) (result string, err error) {
	resp, err1 := http.Get(url)
	if err1 != nil {
		err = err1 // 将封装函数的内部的错误抛给调用者
		return
	}
	defer resp.Body.Close()
	// 循环读取，传出给调用者
	buf := make([]byte, 4096)
	for {
		n, err2 := resp.Body.Read(buf)
		if n == 0 {
			fmt.Println("读取网页完成")
			break
		}
		if err2 != nil && err2 != io.EOF {
			err = err2
			return
		}
		// 累加每一次循环读到的buf数据,存入result一次性读取
		result += string(buf[:n])
	}
	return

}

func main() {
	// 指定爬取起始，终止页
	var start, end int
	fmt.Print("请输入爬取的起始页:")
	fmt.Scan(&start)
	fmt.Println("请输入终止页")
	fmt.Scan(&end)

	working(start, end)
}
```

运行截图：

![image-20220507101227154](C:UserspjAppDataRoamingTypora	ypora-user-imagesimage-20220507101227154.png)