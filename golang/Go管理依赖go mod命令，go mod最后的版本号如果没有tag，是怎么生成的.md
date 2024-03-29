>  题目序号：(2768)
>
>  **题目来源**：字节  
>
>  **频次**：1

## 答案：peace

如果没有 tag，就会去拉取最新一次 commit。
也可以直接去拉取某个指定的分支，下面的命令会拉取分支 v1.0.1 的代码：

```shell
go get github.com/rayjun/go-mod-demo@v1.0.1
```

如果 tag 和分支同名时，会优先去拉取 tag 下的代码，一般情况下，不要让 tag 和分支同名，而且同名会导致推送代码到分支失败。

在一些情况下，我们可以需要删除 tag 名，更新代码后再使用这个相同的 tag 名，这样就会导致之前已经下载了这些代码的人无法获取到最新的代码，所以需要先清空本地的缓存，然后再重新下载依赖：

```go
go clean --modcache # 将本地缓存的所有依赖都清空
go get -u github.com/rayjun/go-mod-demo@v1.0.1 # 获取最新的代码
```

最后采用go mod tidy 整理依赖。