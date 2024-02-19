> **题目来源**：京东      

## 答案：行飞子

recflect是golang用来检测存储在接口变量内部(值value；类型concrete type) pair对的一种机制。它提供了两种类型（或者说两个方法）让我们可以很容易的访问接口变量内容，分别是reflect.ValueOf() 和 reflect.TypeOf()。

- ValueOf用来获取输入参数接口中的数据的值，如果接口为空则返回0
- TypeOf用来动态获取输入参数接口中的值的类型，如果接口为空则返回nil