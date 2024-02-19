包 (`pkg`) 是 Go 工作区中包含 Go 源文件或其他包的目录。源文件中的每个函数、变量和类型都存储在链接包中。每个 Go 源文件都属于一个包，该包在文件顶部使用以下命令声明：

```go
package <packagename> 
```

您可以使用以下方法导入和导出包以重用导出的函数或类型： 

```go
import <packagename> 
```

Golang 的标准包是`fmt`，其中包含格式化和打印功能，如`Println()`.