**缓存算法**

缓存算法，比较常见的是三种：

- LRU（least recently used ，最近最少使用)
- LFU（Least Frequently used ，最不经常使用)
- FIFO（first in first out ，先进先出)

🦅 **手写 LRU 代码的实现**

手写 LRU 代码的实现，有多种方式。其中，最简单的是基于 LinkedHashMap 来实现，代码如下：

```
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;

    /**
     * 传递进来最多能缓存多少数据
     *
     * @param cacheSize 缓存大小
     */
    public LRUCache(int cacheSize) {
        // true 表示让 LinkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map 中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。
        return size() > CACHE_SIZE;
    }

}
```

其它更复杂，更能体现个人编码能力的 LRU 实现方式，可以看看如下两篇文章：

- [《动手实现一个 LRU Cache》](https://crossoverjie.top/2018/04/07/algorithm/LRU-cache/)
- [《缓存、缓存算法和缓存框架简介》](http://blog.jobbole.com/30940/) 文末，并且还提供了 FIFO、LFU 的代码实现。