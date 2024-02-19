> 题目来源：微步

答案：行飞子

golang是通过 channel 进行协程(goroutine)之间的通信来实现数据共享，这种方式的优点是通过提供原子的通信原语，避免了竞态情形(race condition)下复杂的锁机制。