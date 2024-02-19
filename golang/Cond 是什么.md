
Cond 实现了一种条件变量，可以使用在多个 Reader 等待共享资源 ready 的场 景（如果只有一读一写，一个锁或者 channel 就搞定了）

每个 Cond 都会关联一个 Lock（*sync.Mutex or *sync.RWMutex），当修改条 件或者调用 Wait 方法时，必须加锁，保护 condition。