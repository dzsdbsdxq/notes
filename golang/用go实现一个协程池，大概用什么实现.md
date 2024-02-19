> 题目序号：773
>
> 题目来源：网易
>
> 频次：1

## 答案：Carpe-Wang

定义一个task 的结构体 标示具体要执行的任务格式

```Go
type Job func([]interface{})type taskWork struct {
	    Run       Job	
        startBool bool	
        params    []interface{}
}
```

定义一个worker 池，控制协程相关信息

```go
type WorkPool struct {
	taskPool  chan taskWork
	workNum   int
	maxNum    int
	stopTopic bool
	//考虑后期 作为冗余队列使用
	taskQue chan taskWork
}
```

实现协程池相关启动，停止，扩容策略，缩减策略，备用队列启用等 逻辑

```go
//得到一个线程池并返回 句柄
func (p *WorkPool) InitPool() {
	*p = WorkPool{workNum: workerNumDefault,
		maxNum: workerNumMax, stopTopic: false,
		taskPool: make(chan taskWork, workerNumDefault*2), taskQue: nil}
 
	(p).start()
	go (p).workerRemoveConf()
}
 
//开始work
func (p *WorkPool) start() {
	for i := 0; i < workerNumDefault; i++ {
		p.workInit(i)
		fmt.Println("start pool task:", i)
	}
}
 
//初始化 work池 后期应该考虑如何 自动 增减协程数，以达到最优
func (p *WorkPool) workInit(id int) {
	go func(idNum int) {
		//var i int = 0
		for {
			select {
			case task := <-p.taskPool:
				if task.startBool == true && task.Run != nil {
					//fmt.Print("this is pool ", idNum, "---")
					task.Run(task.params)
				}
				//单个结束任务
				if task.startBool == false {
					//fmt.Print("this is pool -- ", idNum, "---")
					return
				}
				//防止从channal 中读取数据超时
			case <-time.After(time.Millisecond * 1000):
				//fmt.Println("time out init")
				if p.stopTopic == true && len(p.taskPool) == 0 {
					fmt.Println("topic=", p.stopTopic)
					//work数递减
					p.workNum--
					return
				}
				//从备用队列读取数据
			case queTask := <-p.taskQue:
				if queTask.startBool == true && queTask.Run != nil {
					//fmt.Print("this is que ", idNum, "---")
					queTask.Run(queTask.params)
				}
			}
 
		}
	}(id)
 
}
 
//停止一个workPool
func (p *WorkPool) Stop() {
	p.stopTopic = true
}
 
//普通运行实例，非自动扩充
func (p *WorkPool) Run(funcJob Job, params ...interface{}) {
	p.taskPool <- taskWork{funcJob, true, params}
}
 
//用select 去做 实现 自动扩充 协程个数 启用备用队列等特性
func (p *WorkPool) RunAuto(funcJob Job, params ...interface{}) {
	task := taskWork{funcJob, true, params}
	select {
	//正常写入
	case p.taskPool <- task:
		//写入超时 说明队列满了 写入备用队列
	case <-time.After(time.Millisecond * 1000):
		p.taskQueInit()
		p.workerAddConf()
		//task 入备用队列
		p.taskQue <- task
	}
}
 
//自动初始化备用队列
func (p *WorkPool) taskQueInit() {
	//扩充队列
	if p.taskQue == nil {
		p.taskQue = make(chan taskWork, p.maxNum*2)
	}
}
 
//自动扩充协程 简单的自动扩充策略
func (p *WorkPool) workerAddConf() {
	//说明需要扩充进程  协程数量小于 1000 协程数量成倍增长
	if p.workNum < 1000 {
		p.workerAdd(p.workNum)
	} else if p.workNum < p.maxNum {
		tmpNum := p.maxNum - p.workNum
		tmpNum = tmpNum / 10
		if tmpNum == 0 {
			tmpNum = 1
		}
		p.workerAdd(1)
	}
}
 
//自动缩减协程 实现比较粗糙，可以考虑后续精细实现一些策略
func (p *WorkPool) workerRemoveConf() {
	for {
		select {
		case <-time.After(time.Millisecond * 1000 * 600):
			if p.workNum > workerNumDefault && len(p.taskPool) == 0 && len(p.taskQue) == 0 {
				rmNum := (p.workNum - workerNumDefault) / 5
				if rmNum == 0 {
					rmNum = 1
				}
				p.workerRemove(rmNum)
			}
		}
	}
 
}
func (p *WorkPool) workerAdd(num int) {
	for i := 0; i < num; i++ {
		p.workNum++
		p.workInit(p.workNum)
	}
}
func (p *WorkPool) workerRemove(num int) {
	for i := 0; i < num; i++ {
		task := taskWork{startBool: false}
		p.taskPool <- task
		p.workNum--
	}
}
```

