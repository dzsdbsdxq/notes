> **题目序号：**（2001）
>
> **题目来源**：shein
>
> **频次**：1

**答案1：**（peace）

通过make创建对象 make只能创建slice  、channel、 map。

- new和make对比：
  1）make 只能用来分配及初始化类型为 slice、map、chan 的数据。new 可以分配任意类型的数据；
  2）new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type；
  3）new 分配的空间被清零。make 分配空间后，会进行初始化；
  4）make 函数只用于 map，slice 和 channel，并且不返回指针