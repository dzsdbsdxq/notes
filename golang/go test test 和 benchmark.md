> 题目序号：1713
>
> 题目来源：字节跳动
>
> 频次：1

**答案1：**（小小）

Go语言中自带有一个轻量级的测试框架`testing`和自带的`go test`命令来实现单元测试和性能测试。

**go test**

由于`go test`命令只能在一个相应的目录下执行所有文件，例如，新建一个项目目录`gotest`,在这个目录下进测试。

比如在该目录下面创建两个文件：gotest.go 和 gotest_test.go

1. gotest.go: 里面有一个函数，实现了一个功能。

2. gotest_test.go: 作为单元测试文件。

   gotest_test.go 有以下原则：

   - 文件名必须是`_test.go`结尾的，这样在执行`go test`的时候才会执行到相应的代码
   - 你必须import `testing`这个包
   - 所有的测试用例函数必须是`Test`开头
   - 测试用例会按照源代码中写的顺序依次执行
   - 测试函数`TestXxx()`的参数是`testing.T`，我们可以使用该类型来记录错误或者是测试状态
   - 测试格式：`func TestXxx (t *testing.T)`,`Xxx`部分可以为任意的字母数字的组合，但是首字母不能是小写字母[a-z]，例如`Testintdiv`是错误的函数名。
   - 函数中通过调用`testing.T`的`Error`, `Errorf`, `FailNow`, `Fatal`, `FatalIf`方法，说明测试不通过，调用`Log`方法用来记录测试的信息。

   编写完成测试用例的代码之后：我们在项目目录下面执行`go test`,就会显示测试结果

**编写压力测试**

- 压力测试用例必须遵循如下格式，其中XXX可以是任意字母数字的组合，但是首字母不能是小写字母

  ```
  func BenchmarkXXX(b *testing.B) { ... }
  ```

- `go test`不会默认执行压力测试的函数，如果要执行压力测试需要带上参数`-test.bench`，语法:`-test.bench="test_name_regex"`,例如`go test -test.bench=".*"`表示测试全部的压力测试函数

- 在压力测试用例中,请记得在循环体内使用`testing.B.N`,以使测试可以正常的运行

- 文件名必须以`_test.go`结尾