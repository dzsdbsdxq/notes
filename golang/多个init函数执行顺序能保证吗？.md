> **题目序号：**823
>
> **题目来源**：高德  
>
> **频次**：1 

 **答案1：**（peace）

**go中不同包中init函数的执行顺序是根据包的导入关系决定的。**
嵌套最深的包内的init函数最先执行。
如下图：
 ![avatar](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/webp.webp)