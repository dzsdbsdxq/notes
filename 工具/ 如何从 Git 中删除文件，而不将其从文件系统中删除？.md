如果你在 `git add` 过程中误操作，你最终会添加不想提交的文件。但是，`git rm` 则会把你的文件从你暂存区（索引）和文件系统（工作树）中删除，这可能不是你想要的。

换成 `git reset` 操作：

```
git reset filename          # or
echo filename >> .gitingore # add it to .gitignore to avoid re-adding it
```

- 上面意思是，`git reset <file>` 是 `git add <file>` 的逆操作。