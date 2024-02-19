实际场景下，缓存值可能是一个 POJO 对象，就需要考虑如何 POJO 对象存储的问题。目前有两种方式：

- 方案一，将 POJO 对象

  序列化

  进行存储，适合 Redis 和 Memcached 。

  - 可参考 [《Redis 序列化方式StringRedisSerializer、FastJsonRedisSerializer 和 KryoRedisSerializer》](https://blog.csdn.net/xiaolyuh123/article/details/78682200) 文章。
  - 对于 POJO 对象比较大，可以考虑使用压缩算法，例如说 Snappy、zlib、GZip 等等。

- 方案二，使用 Hash 数据结构，适合 Redis 。

  - 可参考 [《Redis 之序列化 POJO》](https://my.oschina.net/yuyidi/blog/499951) 文章。

不过对于 Redis 来说，大多数情况下，会考虑使用 JSON 序列化的方案。想要深入的胖友，可以看看如下两篇文章，很有趣：

- [《Redis 内存压缩实战》](http://www.iocoder.cn/Fight/Redis-memory-compression-combat/?self) ，Redis HASH 数据结构，可以通过 ziplist 的编码方式，压缩数据。
- [《redis-strings-vs-redis-hashes-to-represent-json-efficiency》](https://stackoverflow.com/questions/16375188/redis-strings-vs-redis-hashes-to-represent-json-efficiency) ，重点看 BMiner 的回答，提供了四种方案，非常有趣。

