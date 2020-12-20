# HTTPS

> **参考  https原理和流程: https://juejin.cn/post/6844903522333376525   这个链接很棒**
>
> https://blog.csdn.net/kefengwang/article/details/81219121  可以看一些概念的东西

> 参考 RSA非对称加密算法: http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html





## HTTP

#### 三次握手

1. client 发送 SYN(seq=x) 到 server;   `client进入SYN_SEND状态`
2. server 收到并回应 SYN(seq=y), ACK(seq=x+1);   `server进入SYN_RECV状态`
3. client 收到 server 的SYN报文, 回应一个 ACK(seq=y+1) `client发送完进入ESTABLISHED状态, server收到也进入ESTABLISHED状态`

> seq number (x, y) 均为随机数, 抓包看到的一般为相对的数字, 即 0 和 1

#### 四次挥手

1. client 发送 FIN(seq=x) 到 server;   `client只是不再发送数据, 但还可以接受数据`   `client进入FIN_WAIT_1状态`
2. server 回应 ACK(seq=x+1);   `server可能还需要发送数据到client, 所以不能一起发送FIN`   `server进入CLOSE_WAIT,client进入FIN_WAIT_2状态`
3. server 没有要发送的数据之后, 向 client 发送 FIN(seq=y);   `server进入LAST_ACK状态`
4. client 回应 ACK(seq=y+1)   `client发送完成进图established状态, server收到后进入CLOSED状态`   `client在等待固定时间(两个最大生命周期)后, 若没有接受到服务的ACK包, 则进入CLOSED状态`



