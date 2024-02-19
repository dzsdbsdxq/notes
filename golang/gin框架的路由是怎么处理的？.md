答案：呼哈

实现原理：采用的是前缀树的方式实现的动态路由。
代码实现：

```go
r := gin.New()
r.GET("/user/:name", routeUser)
func routeUser(c *gin.Context){
    //todo something
}
```

r := gin.New()生成了一个Engine对象，Engine对象是整个框架的核心，也包含了对路由的操作和许多成员变量，其中包括路由要执行的任务链Handlers HandlersChain，方法树trees methodTrees等。
r.GET("/user/:name", routeUser)定义一个GET请求，模糊匹配/user/:name。
访问路由：localhost:8080/user/abc。