> **题目序号：**(483)
> **题目来源：** 阿里
> **频次:** 1

答案：阿纪、

1. **从源代码文件到可执行文件过程发生了哪些事情**
   前端编译
       1.根据架构初始化不同的链接器Link结构体
       2.根据一些参数 比如，go compile后用户输入的参数初始化Link结构体里面的一些字段
       3.词法分析、语法分析 生成ast抽象语法树，类型检查。一些关键字转换为runtime里的函数
       4.逃逸分析
   后端编译
       1.初始化生成中间代码的配置。ssaconfig
       2.编译顶层函数，生成、优化ssa。
       3.汇编代码生成机器码