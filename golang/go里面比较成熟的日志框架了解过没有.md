> 题目序号：812
>
> 题目来源：高德
>
> 频次：1

## 答案：Carpe-Wang

golang日志库golang标准库的日志框架非常简单，仅仅提供了print，panic和fatal三个函数，对于更精细的日志级别、日志文件分割以及日志分发等方面并没有提供支持。所以催生了很多第三方的日志库，但是在golang的世界里，没有一个日志库像slf4j那样在Java中具有绝对统治地位。golang中，流行的日志框架包括logrus、zap、zerolog、seelog等。
logrus特性完全兼容golang标准库日志模块：logrus拥有六种日志级别：debug、info、warn、error、fatal和panic，这是golang标准库日志模块的API的超集。如果您的项目使用标准库日志模块，完全可以以最低的代价迁移到logrus上。

```go
import (
  log "github.com/sirupsen/logrus"
)

func main() {
  log.WithFields(log.Fields{
    "animal": "walrus",
  }).Info("A walrus appears")
}
Logger
import (
	"github.com/sirupsen/logrus"
	"os"
)

// logrus提供了New()函数来创建一个logrus的实例。
// 项目中，可以创建任意数量的logrus实例。
var log = logrus.New()

func main() {
    // 为当前logrus实例设置消息的输出，同样地，
    // 可以设置logrus实例的输出到任意io.writer
	log.Out = os.Stdout

    // 为当前logrus实例设置消息输出格式为json格式。
    // 同样地，也可以单独为某个logrus实例设置日志级别和hook，这里不详细叙述。
    log.Formatter = &logrus.JSONFormatter{}

	log.WithFields(logrus.Fields{
		"animal": "walrus",
		"size":   10,
	}).Info("A group of walrus emerges from the ocean")
}
```

