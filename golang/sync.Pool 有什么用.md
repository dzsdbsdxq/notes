
对于很多需要重复分配、回收内存的地方，`sync.Pool` 是一个很好的选择。频 繁地分配、回收内存会给 GC 带来一定的负担，严重的时候会引起 CPU 的毛 刺。而 `sync.Pool` 可以将暂时将不用的对象缓存起来，待下次需要的时候直 接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系 统的性能。