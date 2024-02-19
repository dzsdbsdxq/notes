如果就以 Partition 为最小存储单位，可以想象，当 Kafka Producer 不断发送消息，必然会引起 Partition 文件的无限扩张，将对消息文件的维护以及已消费的消息的清理带来严重的影响，因此，需以 segment 为单位将 Partition 进一步细分。

每个 Partition（目录）相当于一个巨型文件，被平均分配到多个大小相等的 segment（段）数据文件中（每个 segment 文件中消息数量不一定相等），这种特性也方便 old segment 的删除，即方便已被消费的消息的清理，提高磁盘的利用率。每个 Partition 只需要支持顺序读写就行，segment 的文件生命周期由服务端配置参数（`log.segment.bytes`，`log.roll.{ms,hours}` 等若干参数）决定。