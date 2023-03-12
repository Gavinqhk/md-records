# tcp (Transmission Control Protocol，传输控制协议)

- 面向链接
- 有状态
- 可靠传输
- 流量控制
- 拥塞控制
- 基于字节流的传输层协议

## 三次握手

Slient: SYN=1 seq=q 请求发送给 Server。状态修改Clicent: SYN_SEND，Server: listen
Server: SYN=1 ACK=1 ack=q+1 seq=p 返回给Client。状态修改Client: SYN_SEND,Server:SYN_RCVD
Client: ACK=1, ack=p+1, sep=a+1

## 四次挥手

## 丢包重传（超时重传）

## 流量控制

对于发送端和接收端而言，TCP 需要把发送的数据放到发送缓存区, 将接收的数据放到接收缓存区。

而流量控制索要做的事情，就是在通过接收缓存区的大小，控制发送端的发送。如果对方的接收缓存区满了，就不能再继续发送了。

## 拥塞控制
