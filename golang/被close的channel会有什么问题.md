> 题目序号：(658)
> 题目来源：网易
> 频次：1

**答案1：**（呼哈）
nil的channel：关闭会panic;
非空的channel：关闭成功，读完数据后返回零值;
空的channel：关闭成功，读完数据后返回零值;
满了的channel：关闭成功，读完数据后返回零值;
没满的channel：关闭成功，读完数据后返回零值;
向关闭的channel写数据，会panic。