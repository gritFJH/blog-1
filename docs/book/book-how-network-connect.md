# 《网络是怎么连接的》读书笔记

书中从输入网址到显示页面信息这个一个流程做了很细节的分析，先列一个提纲：

- 客户端
  - 浏览器
  - socket 库
  - tcp 协议栈
  - ip 协议栈
  - 网卡
- 客户端局域网
  - 双绞线
  - 集线器
  - 交换机
  - 路由器
- 运营商网络
  - 待总结
- 服务器
  - 防火墙
  - 缓存服务器
  - 网卡
  - ip 协议栈
  - tcp 协议栈
  - socket 库
  - web 服务器

## 客户端

客户端发起一个请求，会经过一下几个过程：

- 解析 url ，获取需要请求的文件。
- dns 解析域名，获取 ip 地址（从跟域名开始查找，例如：a.b.com，先找 com，再找 b，再找 a）。
- 浏览器生成 http 消息，并转交给 socket 库。
- **进入 tcp 协议栈**
  - 创建 tcp 头（包括发送方，接收方端口号，ip 等信息）。
  - 根据 tcp 头部信息，找到需要连接的套接字。
  - 连接套接字（3 次握手）。
  - 将大的数据包拆分成小的数据包。
  - 通过传递 ACK 和序号，验证双方收到的信息是否可靠。
  - **滑动窗口**，管理 ACK，协商服务器端处理能力。（每一次请求都会返回服务器端剩余的带宽，当服务器带宽很小时，发送方也会减慢上传数据）。
  - 每一个请求响应都会返回服务带宽和 ACK，如果请求很快，则会传输大量的验证信息，tcp 会进行合并处理（即：每次发送带宽信息时，会等待一段时间，如果时间内有其他的链接占用了带宽，也会合并后，发送给客户端）。
  - **拥塞处理**，慢开始算法，拥塞避免算法等，在传输开始时将发送窗口慢慢指数级扩大。（避免网络负载过大）。
- **进入 ip 协议栈**
  - 根据服务器 ip，包裹 ip 头部。
  - 根据网卡的唯一 mac 地址，包裹 mac 头部。
- **进入网卡**
  - 收到 ip 包裹的包，检查以太网可发送状态。
  - 将数据信号的包转换成电信号，通过双绞线发送出去。

## 客户端局域网

- 信号通过**双绞线**，到达集线器。
  - 双绞线的作用是为了抑制噪音，防止其他信号干扰。
- **集线器**将信号广播到所有端口，这样信号便能到达交换机。
  - 集线器：将一些机器连接起来组成一个局域网。（OSI 第一层，物理层）
- **交换机**
  - 交换机又叫交换式集线器：根据 mac 地址表，查找 mac 地址，然后将信号发送到 mac 地址上（**mac 地址对应的就是路由器**）。
- **路由器**
  - 根据收到的包的接收方 ip 地址查询自身的路由表找到输出端口，并将包转发到输出端口（可能是目的地，也可能是下一个路由器）。
  - 地址转换，可以在转发时，对 ip 和端口号进行改写（解决公网 ip 有限问题）
  - 包过滤，可以根据 mac 头，ip 头等，选择转发 or 丢弃包。

::: tip 路由器和交换机的关系

路由器：将 ip 转发到目标地址的过程。

交换机：将包传给下一个路由器时，需要交换机转发。

路由器是在 ip 层，而交换机在 mac 层。当进行数据传递时，会经过 mac 层，在到 ip 层，如果当前 ip 没有找到目标地址，则会被转发到下一个路由器，仍然是先通过 mac 层，再到 ip 层，直到找到为止。

:::

## 运营商网络

这一部分设计到了很多硬件知识，我也没有完全理解，以后待完善。

:::tip 我们在访问一个 IP 时，可能访问到的是一个集群，怎么能够指定某一台机器呢？

这里就用到了子网掩码，子网掩码：将 IP 地址划分成网络地址和主机地址两个部分。子网掩码与 IP 地址都是由 4 个数段组成，每个数段的取值范围是 0-255。如果将每一个数段用二进制表示，可以表示成连续的 1 和 0 组成，连续的 1 表示网络地址，连续的 0 表示主机地址，通过 0 的个数可以计算出子网的容量。

- 主机号：全 0 表示访问整个子网，并不会访问单个机器。
- 主机号：全 1 表示访问所有子网下的机器，类似于广播。

:::

## 服务器

- **防火墙**，对进入的包进行检查，判断是否允许通过。
  - 通过接收方 ip 地址和发送方 ip 地址，进行过滤。
  - 通过接收方端口号和发送方的端口号，进行过滤。
  - 通过三次握手的方向，控制单向连接，例如只允许 web 端->服务器，或者只允许服务器->web 端发起请求。
  - 控制内网 ip，禁止指定 ip 不能访问外网。
  - 控制外网 ip，禁止指定 ip 外网不能访问内网。
- **缓存服务器**，如果用户请求的页面已经缓存在服务器上，则代替服务器想用户返回页面数据。
- **网卡**将电信号，转换成数字信息，交给协议栈。
- **ip 协议栈**
  - 判断是不是发给自己的包。
  - 判断网络包是否经过分片。
  - 检查 ip 头部，取出对应的 tcp 数据包。并转交给 tcp 模块。
- **tcp 协议栈**
  - 根据收到的包的发送方 ip 地址，发送方端口号，接收方 ip 地址，接收方端口号找到相应的套接字。
  - 根据 tcp 头部，取出对应的 http 数据包。
  - 将数据拼合起来并保存在接收缓冲区中。
- 通过 socket 库将原始的 http 数据包转交给 web 服务器。
- web 服务器分析 http 消息的内容。
  - 读取 url，转换为实际文件名。
  - 检查文件访问控制，确保有权限访问文件。
  - 将文件经过处理后，返回给客户端。
  - 客户端根据 content-type 字段来渲染不同的内容。
