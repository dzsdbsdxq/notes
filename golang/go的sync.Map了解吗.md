> 题目序号：89
>
> 题目来源：好未来
>
> 频次：1

答案：咸鱼没有早餐

**总体概述**

sync.Map 采用读写分离和用空间换时间的策略保证 Map 的读写安全

**Map 的基本结构**

```go
type Map struct {
    mu Mutex
    read atomic.Value
    dirty map[ant]*entry
    misses int
}
```

![](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/image-20220410161919256.png)

**read** 

read 使用 `map[any]*entry` 存储数据，本身支持无锁的并发读

read 可以在无锁的状态下支持 CAS 更新，但如果更新的值是之前**已经删除过的** entry 则需要加锁操作

由于 read 只负责读取，dirty 负责写入，因此使用 `amended` 来标记 dirty 中是否包含 read 没有的字段

**dirty**

dirty 本身就是一个原生 map，需要加锁保证并发写入

**entry**

read 和 dirty 都是用到 entry 结构

entry 内部只有一个 unsafe.Pointer 指针 p 指向 entry 实际存储的值

指针 p 有三种状态

- p == nil 

  在此状态下对应： entry 已经被删除 或 map.dirty == nil 或 map.dirty 中有 key 指向 e ==此处不明==

- p == expunged

  在此状态下对应：entry 已经被删除 或 map.dirty != nil 同时该 entry 无法在 dirty 中找到

- 其他情况

  entry 都是有效状态并被记录在 read 中，如果 dirty 不为空则也可以在 dirty 中找到

**Load**

```go
func (m *Map) Load(key any) (value any, ok bool) {
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
    // 如果 read 中读取不到，则尝试从 dirty 中读取
	if !ok && read.amended {
		m.mu.Lock()
        // 双重判定，防止枷锁过程中 dirty 被提升为 read
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
			e, ok = m.dirty[key]
            // 尽管可以从 dirty 中获取数据，但被认为是 read 未击中且低效
			m.missLocked()
		}
		m.mu.Unlock()
	}
    // read 中读取不到且此时 dirty 包含的数据 <= read，直接返回
    // 此时 dirty 只有两种情况 要么为 nil，要么就是 read 的拷贝
	if !ok {
		return nil, false
	}
	return e.load()
}
```

 **missLocked**

```go
func (m *Map) missLocked() {
	m.misses++
    // 用长度来判断性能，如果 misses 大于 dirty 的长度，就认为继续拓展 dirty 不如重新 copy 一份
	if m.misses < len(m.dirty) {
		return
	}
	m.read.Store(readOnly{m: m.dirty})
    // 将 dirty 拓展为 read，原 dirty 设置为 nil(此处应该是与前文相对)
    // 如果 dirty 为 nil，则下次写入时会将 dirty 初始化为一份 read 的浅拷贝
	m.dirty = nil
	m.misses = 0
}
```

**Store**

```go
func (m *Map) Store(key, value any) {
    // 如果存储的 key 和 value 已经存在直接返回即可
	read, _ := m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok && e.tryStore(&value) {
		return
	}

	m.mu.Lock()
	read, _ = m.read.Load().(readOnly)
	if e, ok := read.m[key]; ok {
        // 当前值没有被删除过写入 dirty
		if e.unexpungeLocked() {
			m.dirty[key] = e
		}
		e.storeLocked(&value)
	} else if e, ok := m.dirty[key]; ok {
        // 当前 key 在 dirty 中存在值，直接更新
		e.storeLocked(&value)
	} else {
        // key-value 不在 read 也不在 dirty 中，修改 read.amdended，并将 k-v 存入 dirty
		if !read.amended {
			m.dirtyLocked()
			m.read.Store(readOnly{m: read.m, amended: true})
		}
		m.dirty[key] = newEntry(value)
	}
	m.mu.Unlock()
}
```

**Delete**

```go
func (m *Map) LoadAndDelete(key any) (value any, loaded bool) {
	read, _ := m.read.Load().(readOnly)
	e, ok := read.m[key]
    // 要删除的值不在 read 中
	if !ok && read.amended {
		m.mu.Lock()
        // 双重判断，
		read, _ = m.read.Load().(readOnly)
		e, ok = read.m[key]
		if !ok && read.amended {
			e, ok = m.dirty[key]
            // 直接删除 dirty 中的键值
			delete(m.dirty, key)
            // 虽然此处时删除操作，但是还是通过抵消(dirty)途径访问到的，因此对 misses++
			m.missLocked()
		}
		m.mu.Unlock()
	}
    // 值在 read 可以直接找到，则直接删除
	if ok {
		return e.delete()
	}
	return nil, false
}
```

**Range**

```go
func (m *Map) Range(f func(key, value any) bool) {
    // 如果 read 中 amended 字段为 false 则直接遍历 read 就可以获取到所有数据
	read, _ := m.read.Load().(readOnly)
	if read.amended {
		
		m.mu.Lock()
		read, _ = m.read.Load().(readOnly)
		if read.amended {
            // 由于 dirty 存储了部分 read 中没有的数据，因此在遍历时直接将 dirty 提升为 read
			read = readOnly{m: m.dirty}
			m.read.Store(read)
			m.dirty = nil
			m.misses = 0
		}
		m.mu.Unlock()
	}

	for k, e := range read.m {
		v, ok := e.load()
		if !ok {
			continue
		}
		if !f(k, v) {
			break
		}
	}
}
```
