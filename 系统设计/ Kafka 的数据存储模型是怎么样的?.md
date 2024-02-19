Kafka 每个 Topic 下面的所有消息都是以 Partition 的方式分布式的存储在多个节点上。同时在 Kafka 的机器上，每个 Partition 其实都会对应一个日志目录，在目录下面会对应多个日志分段（LogSegment）。

```
MacBook-Pro-5:test-0 yunai$ ls
00000000000000000000.index 00000000000000000000.timeindex leader-epoch-checkpoint
00000000000000000000.log 00000000000000000004.snapshot
```

- Topic 为 `test` ，Partition 为 0 ，所以文件目录是 `test-0` 。

LogSegment 文件由两部分组成，分别为 `.index` 文件和 `.log` 文件，分别表示为 segment **索引**文件和**数据**文件。这两个文件的命令规则为：Partition 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个 segment 文件最后一条消息的 offset 值，数值大小为 64 位，20 位数字字符长度，没有数字用 0 填充，如下，假设有 1000 条消息，每个 LogSegment 大小为 100 ，下面展现了 900-1000 的 `.index` 和 `.log` 文件：

[`.index` 和 `.log` 文件](https://user-gold-cdn.xitu.io/2018/7/24/164ca4f7e778be7c?imageView2/0/w/1280/h/960/format/jpeg/ignore-error/1)

- 由于 Kafka 消息数据太大，如果全部建立索引，即占了空间又增加了耗时，所以 Kafka 选择了稀疏索引的方式（通过 `.index` 索引 `.log` 文件），这样的话索引可以直接进入内存，加快偏查询速度。

