在 `if-then` 中使用测试命令（ `-gt` 等）来比较两个数字。例如：

```
#!/bin/bash
x=10
y=20
if [ $x -gt $y ]
then
echo “x is greater than y”
else
echo “y is greater than x”
fi
```