 TCP(Transmission Control Protocol)和 UDP(User Datagram Protocol) 协议属于传输层协议，它们之间的区别包括：

[![TCP 与 UDP 的区别](https://image-1302243118.cos.ap-beijing.myqcloud.com/img/047b5c455b1153e895df36e364134fcd)](http://static.iocoder.cn/047b5c455b1153e895df36e364134fcd)TCP 与 UDP 的区别

- TCP 是面向连接的；UDP 是无连接的。
- TCP 是可靠的；UDP 是不可靠的。
- TCP 只支持点对点通信；UDP 支持一对一、一对多、多对一、多对多的通信模式。
- TCP 是面向字节流的；UDP 是面向报文的。
- TCP 有拥塞控制机制；UDP 没有拥塞控制，适合媒体通信。
- TCP 首部开销(20 个字节)，比 UDP 的首部开销(8 个字节)要大。