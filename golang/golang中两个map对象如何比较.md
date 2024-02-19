> 题目序号：2093
>
> 题目来源：哔哩哔哩
>
> 频次：1

## 答案：fly

使用reflect.DeepEqual 这个函数进行比较。使用 reflect.DeepEqual 有一点注意：由于使用了反射，所以有性能的损失。如果你多做一些测试，那么你会发现 reflect.DeepEqual 会比 == 慢 100 倍以上。