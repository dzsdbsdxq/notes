> 题目来源：疯狂猜图

答案：fly

Go语言中的反射是由 reflect 包提供支持的，它定义了两个重要的类型 Type 和 Value 任意接口值在反射中都可以理解为由 reflect.Type 和 reflect.Value 两部分组成，并且 reflect 包提供了 reflect.TypeOf 和 reflect.ValueOf 两个函数来获取任意对象的 Value 和 Type。