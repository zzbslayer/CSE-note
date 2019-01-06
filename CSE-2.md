# lec16 

## Introduction to Network
As a component and as a system itself

### 网络分层

- 网络分层
![network-layer](./image/network-layer.png)

- 封装
![encapsulation](./image/encapsulation.png)

- 应用层（Application Layer）(**data**)
    - Entities
        - Client & Server
        - End-to-end connection
    - Namespace：URL
    - 协议
        - HTTP、FTP、SMTP 
    - 关注点在于传输的数据
- 传输层（Transport Layer）(**TCP：segment** **UDP: datagram**)
    ![transport-layer](./image/transport-layer.jpg)
    - Entities
        - Sender & Receiver
        - 代理、防火墙等
        - End-to-end connection
    - Namespace：端口号
    - 协议
        - TCP、UDP
    - 关注点
        - TCP： 若丢包则重传
        - UDP： Nothing
- 网络层(Network Layer / IP Layer) (**packet**)
    ![packet](./image/packet.png)
    - Network Entities
        - 网关、网桥
        - 路由器
    - Namespace
        - IP 地址
    - 协议
        - IP ICMP
    - 整个网络视作一个图，网络层关注的是下一个节点哪个（由 Forwarding Table 决定）
- 链路层(Link Layer(& Physical Layer)) (**frame**(&**bit**))
    ![frame](./image/frame.jpg)
    - Network Entities
        - 集线器、交换机
        - 各种网线
    - Namespace
        - 不需要
    - 协议
        - Ethernet ATM
        - ARP RARP
    - 关注物理传输与错误比特的检测

## 链路层 （The Link Layer）

- 目的： 将数据从一个物理位置传输到另一个物理位置

### 同步传输： 使用共享时钟（shared clock）进行物理传输

- Example： 将同一个芯片上的寄存器1的bit 传输给寄存器2
    - 用一根线将寄存器1的输出接到寄存器2的输入
    - 等待寄存器1的输出完毕，信号传给寄存器2 (propagation)
    - 寄存器2在下一次 clock tick 读取输出
    - **前提假设**：propagation 能在一个 clock 中做完

### 不使用共享时钟
- Three-wire ready/acknowledge protocol
    - 1. A 将数据放置在 data line 上
    - 2. A 改变 ready line 上的值
    - 3. B 发现 ready line 的值改变，然后就读取 data line 的数据，并且改变 acknowledge line 的值

    ![without-shared-clock](./image/without-shared-clock.png)

### Parallel Transmission （并行？传输）
- 按照之前的方式，如果传输时间 Δt (propagation time)，那么每次传输 1 bit 需要多于 2Δt 的时间，最大数据传输率(data rate)为 1/(2Δt)

- Parallel Transmission
    - 用 N 根并行的 data line 那么传输率就能达到 N/(2Δt)

- Example：SCSI，打印机

### Serial Transmission （序列化传输）
- Ready/acknowledge protocol 这种方式 Δt 增长的很快，会极大程度上限制数据传输率
- Serial Transmission
    - 一根线中发送 a stream of bits
    - 并不等待 receiver 的回复
    - 期望 receiver 自己恢复这些 bits
    - Higher rates, longer distance, fewer wires
    - Example: USB, SATA

### 信号传输的模拟信号

![analog](./image/analog.png)

- B 并不知道 A 的 clock 是什么样的，因此不知道该如何采样。如上图所示。

- Voltage Controlled Oscillator
    - 这种硬件本质上是作为 receiver 的 clock source。原理略。
    - 总之，使用 VCO 就能够让 B 采集到 A 传输过来的信号
- VCO 的问题：在 stream 中如果没有数据传输（e.g.  传输的 bit 全是 0），VCO 的 phase-locked loop (不用关注这是什么，这就是 VCO 原理中的一步，感兴趣可以自己看 ppt) 将会无法同步。

### 曼彻斯特编码 （Manchester Code）
为了解决 VCO 的这个问题，将每个byte编码，保证传输。

- 0 => 01, 1 => 10
- 最大数据传输率减半，但是这种方法简单有效

### 如何共享连接？
- 同步通信（Telephone communication）
    - 需要交换机先行安排（prior arrangement）
    - 连接：set up & tear down
    - 流式传输（Stream）：continuous bits flows out of a phone
- 异步通信（Asynchronous communication）
    - 消息机制（Message）：突发性的，ill-suited to fixed size and spacing of isochronous frames
    - 无长连接 （Connectionless）

### 同步多路复用（Isochronous Multiplexing）
![isochronous-multiplexing](./image/isochronous-multiplexing.png)

- 电话网络 Telephone Network
    - Leverage "virtual link" for connection （？）
    - 当交换机中没有空闲的位置时，将会进入网络繁忙状态
- 时分多路复用 Time Division Multiplexing (TDM)
    ![tdm](./image/tdm.png)
    - 时间片a给第一个人用，时间片b给第二个人用，……
    - Q：为什么电话声音仍然是连续的？
    - A：时间片分隔足够小

### 数据传输网络 （Data Communication Network）

![data-communication](./image/data-communication.png)

### Frame and Packet: Asynchronous Link （链路层和网络层，异步链路）
![frame](./image/frame.jpg)
- Frame 可以是任意长度，只要链路是空闲的，可以随时被传输
- Packet 是带有 guidance info 的 variable-length 的 frame
- 无长连接（Connectionless transmission）：不需要维护状态
- Segment and reassemble
- Packet voice: replacing many parts of isochronous network


![asynchronous-link](./image/asynchronous-link.png)

### Multiplexing / Demultiplexing

![multiplexing-switch](./image/multiplexing-switch.png)


- 多路复用要维护一个队列，交换机需要内存/缓存
- 解复用需要用到 packet header 里的信息
    - Header 包括目的地
    - 交换机里有一个 forwarding table，根据这个表选择经由哪个链路到目的地

### 错误处理（Error Handling）
- 错误检测的代码
    - 冗余数据
    - Example：checksum
- 检测到错误以后做什么？
    - 错误纠正：需要足够的额外数据（redundancy）
        - 噪声需要被充分理解（比如磁盘）
    - 要求发送者重传：发送者在 buffer 里还有 frame
    - 接收者放弃数据
    - 综合以上所有方法

### 实现：更多的冗余信息（Incremental Redundancy）

- 前向传播错误纠正
    - 存储、传输之前，首先进行编码（encoding）
    - 之后由接收者自行解码（decoding）
- 海明距离（Hamming Distance）
    - 定义：A 异或 B 的结果中 1 的个数，即为 A 与 B 的海明距离
- If H-distance between every legitimate pair is 2
    - 000101, can only detect 1-bit flip
- If H-distance between every legitimate pair is 3
    - Can only correct 1 bit flip
- If H-distance between every legitimate pair is 4
    - Can detect 2-bit flip, correct 1-bit flip


- Example 1： Simple Parity Check
    - 2 bits -> 3 bits
        - 发现 1-bit 的错误
        - 8 patterns total
        ![parity-check](./image/parity-check.png)

    - 只有 4 个正确的 patterns
        - 00 -> 000
        - 11 -> 110
        - 10 -> 101
        - 01 -> 011
    - 海明距离为 2
        - 1 个比特反转会导致错误的模式
- Example 2： 4->bit -> 7->bit
    - 编码规则：4 bits -> 7 bits
        - 3 extra bits 可以区分 8 种情况
        - e.g. [1101] -> 10 [1] 0 [101]
        -  4 bits 中的第一位位于 7 bits 的第三位；4 bits 中的第二到四位位于 7 bits 的第五到七位。（方括号中为原始信息）
        - 额外的 3 个 bit 由下图得出
        ![hamming1](./image/hamming1.png)
    - 如何纠正 1-bit error
        - 检测策略如下表所示
        ![hamming2](./image/hamming2.png)
        - e.g. 如果 P3 错了，计算 P1，P2，P4 发现其中 P1 & P2 不匹配，那么可以确定是 P3 错了。再由 P1 ⊕ P7 ⊕ P5 即可还原 P3 

# lec 17

## Network Layer
All about routing

![network-layer](./image/network-layer.png)

### IP：Best-effort Network
- 尽力服务模式 (Best-effort network)
    - 网络发生拥塞时，不顾及用户，直接丢弃数据包，直至流量减少
- 保证传输模式 (Guaranteed-delivery network)?
    - 也称 store-and-forward network，不丢包
    - 面向完整的信息而不是数据包 (packets)
    - 使用磁盘作缓存，来处理高峰流量
    - 跟踪每条消息，保证没有一条丢失
- 现实世界中
    - 没有绝对的 guarantee
    - High layer: Guaranteed-delivery
    - Lower layer: Best-effort

### Forwarding Table：Routing
![ip-route-table](./image/ip-route-table.jpg)
- 不可能人工建立 Forwarding Table
    - 决定最短路径需要计算
    - 链路改变时需要重新计算 Forwarding Table
    - 链路传输失败时需要重新计算 Forwarding Table
    - 需要根据网络拥塞情况调整
- Stating routing vs adaptive routing
    - Adaptive routing 需要一些信息的交换

### Control-plane VS Data-place
- Control-plane - Learning what we will do
    - 通过定义规则，来控制数据流
    - e.g. 路由算法
- Data-plane - Actualy moving the packets based on what we learned.
    - The data-path which copies data according to the rules
    - 性能很关键
    - e.g. IP 前向传播过程
- 路由表
    - Control-plane 写这个表
    - Data-plane 读这个表

## Routing

### 路由协议的目的
- 1. 使每个交换机都知道如何到达网络中任意一个节点 dst
- 2. 使每个交换机都知道，到达 dst 的最短路径
- 3. 在每个交换机中建立路由表，比如 routing-table[dst] 返回到 dst 的最短路径

### 分布式路由：三步骤
- 节点通过 HELLO protocol 知道他们的邻居
- 节点通过广播(advertisements)知道其他所有能够到达的节点
- 节点决定最短路由

### 两种路由协议
- Protocol 1： Link-state
    - 节点 A 向所有节点广播 A 到 A 的邻居的 cost
        - via flooding
    - 结合使用 Dijkstra's
- Protocal 2： Distance-vector
    - 节点 A 向 A 的邻居广播 A 到所有它已知节点的 cost
    - 通过 Bellman-Ford 更新路由表

### Link-state Routing

![link-state](./image/link-state.png)

- Example: 顶点A的更新过程
    ![link-state-process](./image/link-state-process.png)

### Distance-vector Routing

![distance-vector](./image/distance-vector.png)

### Problem: Do not scale to the Internet
- Link-State
    - Pro: 建表很快 (Fast convergence(n. 会聚))
    - Cons: High overhead because of flooding
    - 适用于 SJTU 规模的网络，而不适用于 Internet
- Distance Vector
    - Pro：Low overhead
    - Cons：建表很慢，与最长的路由成正相关 (Convergence time is proportional to the longest path)
    - 适用于小型网络

### 3 ways to scale
![route-scale](./image/route-scale.png)
- Path-vector Routing
    - 广播中同时包含路径，以检测 loops
- Hierarchy of Routing
    - Route between regions and then within a region
- Topological Addressing （设置地址）
    - 为连续的 block 赋予地址，使 advertisement 更小

### Path Vector Exchange
- 每个节点维护一个 path vector
    - 它是到某个目的地的完整路径
    - 逐渐知道其他的路径
    - 从其他新的 path vector 建立一个新的 forwarding table
- 算法
    - Advertising
    - Path selection

- Example
![path-vector](./image/path-vector.png)

下图的 path-vector & forwarding table 都是节点 G 的
![path-vector-process](./image/path-vector-process.png)

- Better for scaling
    - 类似于 Distance-Vector 但是在广播信息中包含了完整的路由
    - Overhead 更多（advertisement 变大），但是 convergence time 减少（避免了无穷大的最糟糕情况）
    - Overhead 比 Link-State 更低

### Q&A of Path-Vector
- 如何避免 loops
    - 当节点更新 path 时，不会选择含有它自己的 path
- 一个节点接收到同一个目的地多条不同的路径时会怎么做？
    - 取最短
- 图改变了会怎么样？
    - 新的节点加入仍照常计算。
    - 如果某节点从图中删去（该节点停止广播），那么它的邻居就会丢弃跟他有关的路径

### Hierarchy Address Assignment & Routing
- Path Vector 实现的两个问题
    - 每个 attachment point 必须有一个唯一的地址
    - 随着节点 (attachment points) 数量增多，path vector 也会变大
- Region 也可以称为 Autonomous System

![hierarchical-address](./image/hierarchical-address.png)

- 但是引入了新的问题：复杂性
    - 需要将 address 与 location 绑定
    - Paths 可能不再是最短
        - 由于 hierarchy 导致少了一些节点之间的信息
- More about hierarchy
    - 可以拓展至更多的 levels
    - 不同地方各自由不同的 levels

### Routing Hierachy
- 跨地区 （Across Region）
![across-region](./image/across-region.png)
    - 使用一个路由协议来做跨地区的路由，地区中采用另一个协议
    - 在地区边缘的设备需要能够 "translate" or "speak" botn protocols
- BGP: path-vector protocol used across region
    - Border Gateway Protocol

### Topological Addressing
- 减小路由表规模
    - 虽然有了 Hierarchy，但是 BGP 仍然是对 IP 地址计算路由 （e.g. to 19.0.0.1 but not to region-3)
    - 每个区域内的 IP 地址都是连续的，因此他们可以通过特定的表示方式表示。e.g. CIDR
    - 使 advertisements 更小

## Data Plane：Packet Forwarding

### 网络层接口
```C
structure packet{
    bit_string source
    bit_string destination
    bit_string end_protocol
    bit_string payload
}
```
```python
procedure NETWORK_SEND (segment_buffer, destination, net_protocol, end_protocol):
    packet instance outgoing_packet
    outgoing_packet.payload ← segment_buffer 
    outgoing_packet.end_protocol ← end_protocol 
    outgoing_packet.source ← MY_NETWORK_ADDRESS 
    outgoing_packet.destination ← destination 
    NETWORK_HANDLE (outgoing_packet, net_protocol)

procedure NETWORK_HANDLE (net_packet, net_protocol):
    packet instance net_packet 
    if net_packet.destination != MY_NETWORK_ADDRESS:
        next_hop ← LOOKUP (net_packet.destination, forwarding_table)
        LINK_SEND (net_packet, next_hop, link_protocol, net_protocol)
    else
        GIVE_TO_END_LAYER (net_packet.payload, 
        net_packet.end_protocol, net_packet.source)
```

### 前向传播 IP packet
- 在 Forwarding Table 中查找数据包的目的地（DST）
    - 若找到，则找到对应的链路（link）
    - 否则 drop packet
- Decrement TTL(Time To Live)
    - 当 TTL 为零时，drop packet
    - 防止网络中出现无限循环的 packet
- 更新 header 的 checksum
- 将 packet 发送至对应端口 （物理端口，而非我们通常所说所说的 TCP/IP 协议中的逻辑端口）
- 将 packet 传输至链路

### Data-plane Case Study: Intel's DPDK
- DPDK: Data Plane Development Kit

![dpdk](./image/dpdk.png)
- 网卡
    - 有一些物理端口
    - 每个物理端口都有有 RX / TX （输入输出口）
- 处理器
    - 从 RX 读数据包  
        - 轮询(polling)
    - 找到输出端口
    - 将数据包写至 TX

## NAT (Network Address Translation)
- 私网
    - Public routes don't accept routes to network 10
- NAT router: 桥接私网
![NAT](./image/NAT.jpg)
    - 本质上是公网与私网之间的路由器
    - 发送: 将源地址(source address)修改为临时的公网地址(public address)
    - 接受: 同理，查找 mapping table 做类似修改
- 限制
    - 一些端到端的协议会把地址放在原始数据(payload)中
    - NAT route 可能成为性能瓶颈
    - 如果两个私网合并该怎么办？
- Example
![NAT-example](./image/NAT-example.png)

## Case: Ethernet Mapping

### Mapping Internet to Ethernet
- Listen-before-sending rule, collision
- Ethernet: CSMA/CD
    - 当 A、B 两站同时检测到信道空闲时，会同时传输数据，但是就会发生冲突。
    - 为了解决该问题而提出该协议
    -  发送数据前 先侦听信道是否空闲 ,若空闲，则立即发送数据。若信道忙碌，则等待一段时间至信道中的信息传输结束后再发送数据；若在上一段信息发送结束后，同时有两个或两个以上的节点都提出发送请求，则判定为冲突。若侦听到冲突,则立即停止发送数据，等待一段随机时间,再重新尝试。

### Overview of Ethernet
- A half duplex(adj. 两部分的) Ethernet
    - The max propagation time is less than the 576 bit times, the shortest allowable packet （？）
    - 两个发送者能同时检测到冲突
    - 如果冲突，等待随机时间
- A full duplex Ethernet & point-to-point Ethernet
    - No collisioins
    - the max length of the link is determined by the physical medium

![ethernet-packet](./image/ethernet-packet.png)

### 交换机与集线器的区别
- 集线器
    - 每个 frame 被广播(broadcast)到它所有物理端口上
    - 一个 10/100Mbps 的集线器，每个端口共享总带宽
- 交换机
    - 记录每个设备的 MAC 地址
    - 一个 10/100Mbps 的交换就，为每个端口分配 full 10/100Mbps

### Broadcast Aspects of Ethernet
- Broadcast Network (集线器)
![broadcast](./image/broadcast.png)
    - 每个 frame 都会被发送至每个 station
    - **ETHERNET_SEND**
        - Pass the call along to the link layer
    - **ETHERNET_HANDLE**
        - 十分简单，甚至可以在硬件中来实现

```python
# No need to do any forwarding
procedure ETHERNET_HANDLE (net_packet, length):
    destination ← net_packet.target_id
    if (destination == my_station_id or destination == BROADCAST_ID ):
        GIVE_TO_END_LAYER (net_packet.data, 
                        net_packet.end_protocol, 
                        net_packet.source_id)
    else
        ignore packet
```

### Layer Mapping: Attach Ethernet to Forwarding Network
![layer-mapping](./image/layer-mapping.png)
- L 给 server N 发 RPC 请求，直接通过 link 1 给 station 18 发数据包即可
- 但是如果 L 要给 E 发 RPC 请求

### Layer Mapping
- The Internet network layer
    - **NETWORK_SEND**(data, length, RPC, INTERNET, N)
    - **NETWORK_SEND**(data, length, RPC, ENET, 18)
- L 必须维护一个表
    - 访问 E 必须要经过 enet/19 的 K
![layer-mapping-table](./image/layer-mapping-table.png)

### ARP (Address Resolution Protocol)
- **NETWORK_SEND**("where is M?", 11, ARP, ENET, BROADCAST)
- **NETWORK_SEND**("M is at station 15", 18, ARP, ENET, BROADCAST)
- L 如果询问 E 的以太网地址，但是 E 在 Internet 中，接收不到 Ethernet 内部的广播，但是位于 station 19 的 router K 可以接收到，K 会回复一个 ARP response
- Manage forwarding table as a cache

### ARP & RARP Protocol
![arp&rarp](./image/arp&rarp.png)
- Name mapping: IP address <-> MAC address

### 网络拓扑结构 (Network Topology) 
- Example: SJTU network
    - 子网(Subnet): usually like 192.168.0.2 or 10.0.0.2
    - 网关(Gateway): usually like 192.168.0.1
        - get the global IP address: 202.120.40.82
        - 网关往往有两个甚至多个 IP 地址
    - 代理(Proxy): get proxy's address
        - e.g. 106.185.46.164 (Japan)

![network-topology](./image/network-topology.png)

### 如何使用套接字(socket)访问 www.baidu.com
![connect-baidu](./image/connect-baidu.png)
- 写代码的时候，类似于直接连接到 baidu 的
    - Call connect() with Baidu's IP address
- 但是系统如何找到下一个链路节点 (next hop) 

### Putting All Together
- App: 想要给 Baidu 发一个数据包. 数据包里是 百度的 IP （target IP），自己的 IP （source IP）
![app-request](./image/app-request.png)
- OS: OS并不知道如何抵达 Baidu 这个网络节点。OS 会将数据包发送给 路由器（网关） (router(gateway)). 但是 OS 并不能改变 source IP， 所以 OS 仅仅把 Target MAC 改成路由器（网关）的 MAC 地址
![os-request](./image/os-request.png)
- Router-1 (gateway): router-1 收到了一个 Target MAC 是自己 MAC 地址的数据包，但是 IP 地址并不是自己的。那么 router-1 就会把 Target MAC 改成下一个节点的 MAC 地址(NAT: 同时改变 source IP 和 source port)，最后把数据包发向下一个节点(next hop)
![router1-request](./image/router1-request.png)
- Router-2: 如果 Router-2 能够访问到 Baidu，那么将 Target MAC 改为 Baidu 的
![router2-request](./image/router2-request.png)

### ARP Spoofing

![arp-spoofing1](./image/arp-spoofing1.png)
![arp-spoofing2](./image/arp-spoofing2.png)

- 然后如果 A 想给 B 发包，那么全都会发到 Hacker 的机器上。
- Cache entry 会过期，因此需要再次发送 ARP 来更新 Cache
    - How often ? depends on the particular system
    - usually every 40s 
- 如果 Hacker 不希望他的 Ethernet driver 一直在运行
    - 用 ifconfig -arp 实现

### 中间人攻击 Man-in-the-Middle Attack
- Hacker 将他的节点插入到两个通信节点之间
- 在同一个交换机子网下，中间人攻击利用 ARP Spoofing 进行:
    - Suppose X is the hacker's computer
    - T1 and T2 are the targets
    - X poisons the ARP cache of T1 and T2
    - T1 associates T2's IP with X's MAC
    - T2 associates T1's IP with X's MAC
    - All of T1 and T2's traffic will then go to X first, instead of directly to each other

![middle-man-attack1](./image/middle-man-attack1.png)
![middle-man-attack2](./image/middle-man-attack2.png)

### 防御 ARP Spoofing
- No universal defense
- 使用静态的 ARP entries
    - 不能被更新
    - Spoofed ARP replies are ignored
    - ARP table needs a static entry for each machine on the network
    - Large overhead
        - Deploying these tables
        - Keep the table up-to-date

### Arpwatch
- 用来监听网络上 ARP replies 的免费 UNIX 程序
- 它会建立 IP/MAC 的表，并存在文件里
- 当某个 IP/MAC 改变(flip-flop)时，自动给网管发邮件提醒
- 但是这并不能完全解决问题，比如 Ettercap 这种攻击方式，它只会引起一些 flip-flop，在 DHCP-enabled network 中 flip-flop 会频繁发生，因此很难被发现

# lec 18
Best-effort is not enough
![network-layer](./image/network-layer.png)

## 端到端层(End-to-End Layer)

### 端到端层
- 网络层并没有保证
    - Delay
    - Certainty of arrival
    - Right place to deliver
    - Order of arrival
    - Accuracy of content
- 端到端层
    - 没有一个设计是能够满足上述全部要求的
    - 每个协议都为一种应用设计
    - Example
        - UDP (User Datagram Protocol)
            - 在一些简单应用中被直接使用
            - 也被使用于其他协议中
        - TCP (Transimission Control Protocol)
            - 保证顺序，没有丢失，没有冗余
            - Provision for flow control
        - RTP (Real-time Transport Protocol)
            - Built on UDP
            - 使用于流式数据如视频、音频
        - 没有一个能够满足所有应用场景

### Assurance of End-to-end Protocol
- At-least-once delivery
- At-most-once delivery
- Data Integrity
- Stream order & closing of connections
- Jitter control （网络波动控制？）
- Authenticity and privacy
- End-to-end performance

### At-least-once delivery
- RTT (Round-trip time) 往返时间
    - to_time + process_time + back_time(ack)
- At least once on best effort network
    - 发数据包时，连带发一个 nonce （类似于一个随机数）
    - 发送者保留 packet 的一份拷贝
    - 如果在收到 ACK 之前超时了，就重发
    - 接收者回复一个 ACK 以及 nonce
- 在接收者返回 error 之前，重发有限次
- Dilemma
![at-least-once](./image/at-least-once.png)
    - 1. 接收者没有收到数据包
    - 2. 接收者收到数据包，但是发送者没有收到 ACK
    - 没有方式能够知道究竟是 1 还是 2
#### 如何决定超时时间
- Fixed Timer 
- Adaptive Timer
- NAK(Negative Acknowledgment)

##### Fixed Timer
- Dilemma
    - Too short: 引起不必要的重传
    - Too long: 发现丢包所需要的时间过长
- Problem
    - NFS 中，固定时间会导致 congestion collapse （拥塞崩溃）
    - Emergent phase synchronization of periodic protocols
        - 周期性轮询 (Periodic polling)
            -  比如发个邮件，"are-you-there"
            - 某个 workstation 会每 5min 广播一次数据包
            - 所有的 workstation 都是在同一时间尝试广播
        - Each workstation
            - Send a broadcast
            - Set a fixed timer
        - Lesson: Fixed timers have many evils. Do not assume that unsynchronized periodic activities will stay that way

    - Wisconsin time server meltdown
        - NETGEAR 为无线路由器添加了一个 feature
            - Logging packets -> timestamp -> time server (SNTP) -> name discovery -> 128.105.39.11
            - Once per second (!) until receive a response
            - Once per minute or per day after that
        - Wisconsin Univ.
            - From 20,000 to 60,000 requests per second, filtering 23457
            - After one week, 270,000 requests per second, 150Mbps
        - Lesson(s)
            - Fixed timers, again
            - Fixed Internet address
            - The client implements only part of a protocol
                - There is a reason for features likes the "go away" response in SNTP

##### Adaptive Timer
- Example 
    - 根据 RTT 调整 
    - Exponential back-off
- 但是 RTT 变化很大
![rtt](./image/rtt.png)

- 计算 RTT 和 Timeout (in TCP)
- Exponentially weighted moving average
    - 估计均值 `rtt_avg` 和方差 `rtt_dev`
    ```python
    Procedure calc_rtt(rtt_sample):
        rtt_avg = a*rtt_sample + (1-a)*rtt_avg; ''' a = 1/8 '''
        dev = absolute(rtt_sample – rtt_avg); 
        rtt_dev = b*dev + (1-b)*rtt_dev;  ''' b = 1/4 ''' 

    Procedure calc_timeout(rtt_avg, rtt_dev):
        Timeout = rtt_avg + 4*rtt_dev
    ```

##### NAK(Negative AcKnowledgement)
- 接收者向发送者发一个信息，列出缺少的数据包
- 接收者对收到的数据包计数，而不是计时
- 发送者不需要维护 timer(only once per stream)

### At-most-once Delivery
- At-least-once 
    - 发送者端记录状态
    - 容易产生多余的请求
- At-most-once
    - 接收端维护一个 nonce table
    - 表格可能会无限增长
        - 空间以及搜索时间
        - Tombstones (something that cannot be deleted forever)
    - 另一个方式：让 app 允许多余的请求
        - 回顾 NFS 的崩溃情况：Server 浪费时间在处理多余的请求

#### 减少冗多于请求 Duplicate Suppression
- 单调递增的 sequence number
    - 接收端丢弃较小 nonce 的请求，保留最后一个 nonce，one per sender (tombstone)
- 接受出错的可能性
    - E.g., if sender always gives up after five RTT (cannot ensure at-least-once), then receiver can safely discard nonces that are older than five RTT
    - It is possible that a packet finally shows up after long delay (solution: wait long time)
- 如果接收者崩溃重启了：会丢失 nonce table
    - One solution:每次重启使用新的 port number
    - Another: ignore all packets until the number of RTT has passed since restarting, if sender tries limit times

- 总之，duplicate suppresion 会引入复杂度

### Data Integrity
- 定义：接收者能够得到与发送者一样的数据内容
- 可靠的发送协议
    - 发送者：加一个 checksum
    - 接收者：重新计算 checksum，不匹配就丢弃
- Q: Is it redundant since link layer provides checksum?
    - 链路层保证传输不会出错，这里可以保证其他地方不会出错, e.g., in memory copying
- 这个保证不是绝对的
    - What if a packet is mis-delivered and the receiver ACK?

### Segments and Reassembly of Long Messages
- Bridge the difference between message and MTU
    - 消息长度：由应用决定
    - MTU(最大传输单元)：由网络决定
- Segment contains ID for where it fits
    - E.g., "message 914, segment 3 of 7"
    - Can be used for at-least-once and at-most-once delivery
- Reassembly
    - Out-of-order, mingled with other message's segments
    - Allocating a buffer large enough to hold the message
    - Keeping a checklist for segments not arrived
- 当乱序的时候(Out-of-order)
    - Solution-1：只接受符合顺序的 ACK
        - 浪费带宽
    - Solution-2：ACK every packet，只在buffer里存较早的buffer，当数据组装完毕后释放buffer
        - 如果有bad packet，需要使用一个很大的buffer
    - Solution-3：结合前两者
        - 当 buffer 满了就丢弃
        - 新的问题：多大的buffer？
    - Speedup for common case
        - NAK 避免超时
        - 如果 NAKs 导致 duplicates，停止 NAKs
    - TCP 基于 ACK 而非 NAK

### Closing of Connections
- Open a stream
    - 记录哪个 elements 被发出、收到、ack
- Close a stream
    - 结束时，report an end-of-stream
    - Both ends need to agree last elements is OK and then close
    - 1. Alice sends close request to Bob with stream record ID
    - 2. Bob checks and agrees, sends a close ACK
    - 3. Alice receives ACK, turn off sender, discard record
    - 4. Alice sends "all done" to Bob
    - 5. Bob receives "all done" and discard stream record

### Jitter Control
- Real-time
    - 当时效性比可靠性更重要时
    - 比如视频中有几个像素点出错，其实并不重要
    - Jitter：传输时的变化，网络中的波动
- Strategy
    - Basic：delay all arriving segments
    - 比如视频传输的 HLS 协议。播放10s前收到的视频，就有缓冲时间
- 需要测量网络延迟的分布
- 决定 D_long 让它比 99% 的网络延迟更长
- 决定最短的延迟 D_short
- 计算 number of segment buffer 
    - n = (D_long - D_short)/D_headway
    - D_headway 是每个 arriving segments 的平均 delay

### Authenticity and Privacy
- Key-based 数学转换方式
    - Sign and verify：establish the authenticity of the source and integrity of contents
    - Encrypt and decrypt: maintain privacy of contents
- Consideration
    - False sense of security, worse than no assurance
    - 你以为安全，比不做任何安全措施还要危险

- Security：非对称加密
    - Public key: encrypt to identify reader (only me can read this)
    - Private key: encrypt to identify writer (yes, it's me who wrote this)
    - 性能不好，所以还是用对称加密

### End-to-end Performance

![end-to-end-performance](./image/end-to-end-performance.png)
![overlapping-transmissions](./image/overlapping-transmissions.png)
- Packets or ACK 可能会丢失
    - 发送者持有一个已发送的 segments 的列表，当收到 ACK 时 check it off
    - According to RTT, set timer for last segment
- If list of missing ACK is empty, OK
- If timer expires, resend packets and another timer

#### Fixed Window
![fixed-window](./image/fixed-window.png)
- Receiver tells the sender a window size
- Sender sends window
- Receiver acks each packet as before
- Window advances when all packets in previous window are acked
    - E.g., packets 4-6 sent, after 1-3 ack done
- If a packet times out -> resend packets
- Still much idle time

#### Sliding Window
![sliding-window](./image/sliding-window.png)
- Sender advances the window by 1 for each in-sequence ACK it receives
    - Reduces idle periods
    - Pipelining idea
- But what's the correct value for the window?
    - We'll revisit this question
    - First, we need to understand windows
- 处理丢包
![sliding-window1](./image/sliding-window1.png)

- Recovery
    - When to take action on sender's checklist?
        - Solution-1: Timestamp and timeout
            - Associate each segment in the list with a timestamp
            - After more than one RTT, resend
        - Solution-2: Numbering segments
            - Receiver sends a NAK
- If sender resends a segment, should the available window increase?
    - Presume receiver discard duplicated segments
    - no increase
- What if a permission message is lost?
    - No data will be received
- Resend permission message?
    - May get twice as much
    - Incremental value is fragile -> cumulative total is better
        - E.g., using "send 1-3" instead of "send 3 more"
    - Similar with "in" and "out" pointer in bounded-buffer
- Rate-matching problem
    - Blizzard of packets arising from a newly-opened window
    - Need cooperation between end-to-end protocol & network forwarders

##### 如何选择合适的 window size
- too small
    - 很长的空闲时间
    - underutilized network
- too large
    - congestion
```
window size >= round-trip time * bottleneck data rate
```
- Larger window makes no sense
- Data rate is still bottleneck
- Example
    - Receive 500 KBps
    - Sender 1 MBps
    - RTT 70ms
    - A segment carries 0.5 KB
    - then: Sliding window size = 35KB (70 segment)

## Congestion Control
- 主要是网络层与端到端层负责处理拥塞
- 网络层
    - 直接经历拥塞
    - 最终决定面临过量数据包该怎么做
- 端到端层
    - 控制发送的频率，最有效的方式

![network-congestion](./image/network-congestion.png)
- If all of a sudden, streams of packets begin arriving on three or four input lines and all need the same output line, a queue will build up
- 虽然更多的内存能够缓解，但是 Nagle(1987) 证实如果内存无限大，其实拥塞会更严重
- 因为很多到队首的 packets 很可能已经过期，仍需要重发

### Load Shedding: Setting window size
- For performance
    - window size ≥ round-trip time × bottleneck data rate
- For congestion control
    - window size ≤ min(RTT x bottleneck data rate, Receiver buffer)
    - Congestion window

### Congestion Control
- Basic idea:
    - Increase congestion window slowly
    - If no drops -> no congestion yet
    - If a drop occurs -> decrease congestion window quickly
- Use the idea in a distributed protocol that achieves
    - Efficiency: i.e., uses the bottleneck capacity efficiently
    - Fairness, i.e., senders sharing a bottleneck get equal throughput (if they have demands)
### AIMD (Additive Increase, Multiplicative Decrease)
![AIMD](./image/AIMD.png)
- Every RTT:
    - No drop: cwnd = cwnd + 1
    - A drop: cwnd = cwnd / 2
- Problem
    - Increases very slowly at the beginning 
    - Initial window size is 1
        - Probably too small in practice
- Solution
![retrofitting-tcp](./image/retrofitting-tcp.png)
    - do multiplicative increase at the beginning
        - Congestion_window init = 1
        - Initially, do Congestion_window ← 2 * Congestion_window each RTT until we hit congestion
        - Named "slow start" (even though it's exponentially fast!) 
    - TCP 并不知道 duplicate 是 lost segment 还是 reordering of segments，因此收到一些 duplicate ACKs 时，window size 才减小


1. Slow start: one packet at first, then double until
Sender reaches the window size suggested by the receiver 
All the available data has been dispatched
Sender detects that a packet it sent has been discarded
2. Duplicate ACK
When receiver gets an out-of-order packet, it sends back a duplicate of latest ACK
3. Equilibrium(平衡)
Additive increase & multiplicative decrease
4. Restart, after waiting a short time

- AIMD leads to both efficiency and fairness
![AIMD-fairness](./image/AIMD-fairness.png)
    - cwnd 指拥塞窗口 congestion window
- Q: Why not additive decrease
    - A: from a congested point, (x',y'), reducing each by 1 worsens fairness and takes us away from the "ideal" outcome

### Weakness of TCP
- If routers have too much buffering, causes long delays
- 丢包并不完全是拥塞导致的
    - 比如无线网络。重试更有效
- TCP 在数据中心性能并不好
    - 高带宽，低延迟的情况
- TCP 对于 long RTTs 不友好
    - throughput 与 RTT 成负相关
    - 很远的地方
        - 即使没丢包，也容易超时
    - 很近的地方
        - 偶尔丢包，窗口一砍，网速腰斩
- Assumes cooperating sources, which is not always a good assumption（客户端可能是恶意的，比如流氓TCP，你腰斩，我翻倍，不要怂，就是干）

### Summary of Congestion Window
- Reliability Using Sliding Window
    - Tx Rate = W / RTT （写入速率等于窗口大小 / round-trip time）
- Congestion Control
    - W = min(Receiver_buffer, cwnd)
    - Congestion window is adapted by the congestion control protocol to ensure efficiency and fairness
    - TCP congestion control uses AIMD which provides fairness and efficiency in a distributed way

# lec 19
## Fault Tolerant
Reliable System from Unreliable Components

### War Stroies: MAXC & Alto （可以跳过不看）
略。总之就是 Alto 相比 MAXC 在容错机制上有修改，导致许多 failure。 Alto 能纠错，能检测错误；MAXC 只能检测错误。MAXC相对稳定，而Alto问题很大。

- Lesson-1:
    - 庞大系统中没有所谓的微小的改动
    - 新的软件可能使之前工作正常的硬件出问题
    - 你永远不知道你现在的是否处于悬崖边缘
- Lesson-2：
    - 经验是处理 failure 最主要的方式
    - 几乎不可能，在没有任何经验的情况下，预测将要碰到哪些 failure

- 回到 MAXC：为什么当时几乎没有什么错误？
    - 虽然硬件会 report 纠错与检测到而未被纠正的错误
    - 软件仅仅记录检测到而未被纠正的错误
- Lesson-3：
    - Safety margin principle ? 
    - " It's good to learn from your mistakes. 
   It's better to learn from other people's mistakes." 
- Alto 之后又开发了 Alto-2
    - 使用了新的 4096 bits 的内存条
    - Single-error-correction, double-error-detection，与MAXC相同
    - 但之后同样发现问题，容错机制在 1/4 cards 中不适用
- Lesson-4：
    - 永远不要认为硬件能够一致按照它说明书说的那样完美运行
    - 看上去在正常工作不代表它实际上确实如此

## Fault, Error, Failure
- **Fault** can be latent(adj. 潜在的) or active
    - If active, get wrong data or control signals
- **Error** is the results of active **fault**
    - Discovery of errors is ad hoc (formal specification?)
- **Failure** happens if an **error** is not detected and masked
    - Not producing the intended result at an interface

- 子系统的 failure 可以成为系统的 fault
    - fault 可以导致 error，error 可以进一步引起更大的子系统的 failure
    - 除非上一级子系统处理了这个 error

### Tolerating Active Faults
- Do nothing: 让上一层系统处理
    - 层级越多，这种模式就越复杂
- Be fail-fast: report the problem
    - E.g., Ethernet 中发生冲突时，会停止发送、广播，然后等待随机时间
- Be fail-safe: transfer incorrect values to acceptable ones
    - E.g., blinking red light in all directions
- Be fail-soft: continues to operate with degradation
    - E.g., airplane with three engines continues to fly if one has failure
- Mask the error: makes incorrect value right

## MTTF(Mean Time To Failure) & Availability
- MTTF: mean time to failure
- MTTR: mean time to repair
- MTBF: mean time between failure
- MTBF = MTTF + MTTR

### 测量 MTBF
- Evalutating & Predicting
    - The more reliable, the longer it takes to evaluate 
    - Not measure directly, but use proxies to estimate its value
- One-way to measure MTBF
    - E.g. 3.5-inch disk's MTTF is 700,000 hours (80 years!)
    - Guess: ran 7,000 disks for 1,000 hours and 10 failed
    - If the failure process were memoryless, then OK
    - "expected operational lifetime" is only 5 years

![bathtub-curve](./image/bathtub-curve.png)

## Redundancy

### Systematically Applying Redundancy

- Masking error
    - Analog system designer: margin
    - Digital system designer: redundancy
- Outline
    - Coding: incremental redundancy
        - 链路层中的纠错、检错机制。Hamming distance
    - Replication: massive redundancy
        - 多个实例，挂了一个还有其他的可以工作
        - Mask failures silently
    - Voting (类似于分布式系统中的 voting)
        - NMR：N-modular redundancy(supermodule)
            - 能够在任何级别应用
        - Fail-vote: NMR with a majority voter
            - 如果某个 replica 不同意 majority，rasie an alert
            - 如果没有 majority，signal a failure
            - 如果两个 replica 以不同形式 fail，那么 fail-fast
        - 系统正常工作的概率 R_supermodel = R^3 + 3R^2(1-R)
        - Voter 也会出错
            - R 是单个 module 正常工作的概率
            - 因此系统出错的概率为 1 - R_supermodel
            - Voter 也同样应该多备份(replica)
            - Everything should be replicated
            - Recursive：voters belong to next module
            - 最后的 voters 是客户端
            ![voter](./image/voter.png)
        - TMR(Triple Modular Redundancy) 可以提高可靠性
            - R(T) = 0.999, TMR's R(T) = 0.999997
        - 但是 MTTF 却变小了
            - 其实很显然，多了几个 replica，每个 replica 都可能会出错，只不过被 mask 了
            - If  MTTF is 6,000 hours and fails independently, and the mechanism of engine failure is memoryless
            - 6,000 hours in only 2,000 hours of flying, first fail
            - 3,000 hours next and cause the second fail
            - 5,000 hours < 6,000 hours
        - MTTF-replica and MTTF-system
            - If MTTF-replica = 1, N replicas total
                - Expected time until 1st failure is MTTF-replica/N
                - Expected time from then until the 2nd is MTTF-replica/(N-1)
                - Expected time until the system of N replicas fails is 
                ```
                MTTF_system = 1 + 1/2 + 1/3 + ... + 1/N
                ```
                - For large N is approximately ln(N)
            - If mission time is long compared with MTTF-replica
                - Simple replication escalates the cost while providing little benefit
    - Repair
        - 公式有点麻烦，直接截图
        ![repair](./image/repair.png)
        ![repair2](./image/repair2.png)

### 回顾我们最初乐观的假设
- 错误独立性 fail independently (?)
    - 谁报错，就是谁出错 （Same vendor, same fault）
    - Earthquake, whose MTTF is less than 3650 years
- 错误的无记忆性 memoryless failure (?)
    - Bathtub curve
- 修复的无记忆性 memoryless repair (?)
    - Stock enough spares
- 完美修复 flawless repair (?)
    - Replace the wrong disk, forget to copy data, etc.

## 磁盘的容错 Magnetic Disk Fault Tolerance
- 使用硬盘作持久化存储
    - Low cost, large capacity, non-volatility
    - 断电数据仍保留
- Three/four nested layers
    - Raw storage, fail-fast storage, careful storage
    - Optional: durable storage

### 磁盘的错误种类
- 机械上的磨损
- A bumping may cause a head to hit the surface
    - Head crush may also create cloud of dust
    - Results in several sectors decaying: decay set
- Electronic components in the controller age
    - E.g. clock timing and signal detection circuits
    - Cause previously good data to become unreadable, or bad data to be written
    - Soft or hard errors
- Seek error
    - Arm moves to a wrong track

### ALL_OR_NOTHING_PUT
```python
procedure ALMOST_ALL_OR_NOTHING_PUT (data, all_or_nothing_sector):
    CAREFUL_PUT (data, all_or_nothing_sector.S1)
    CAREFUL_PUT (data, all_or_nothing_sector.S2)	
    CAREFUL_PUT (data, all_or_nothing_sector.S3)

procedure ALL_OR_NOTHING_GET (reference date,all_or_nothing_sector):
    CAREFUL_GET (data1, all_or_nothing_sector.S1)
    CAREFUL_GET (data2, all_or_nothing_sector.S2)
    CAREFUL_GET (data3, all_or_nothing_sector.S3)
    if (data1 = data2) 
        data ← data1
    else  
        data ← data3

procedure ALL_OR_NOTHING_PUT (data, all_or_nothing_sector):
    CHECK_AND_REPAIR (all_or_nothing_sector)
    ALMOST_ALL_OR_NOTHING_PUT (data, all_or_nothing_sector)

procedure CHECK_AND_REPAIR (all_or_nothing_sector):			                            # Ensure copies match
    CAREFUL_GET (data1, all_or_nothing_sector.S1)
    CAREFUL_GET (data2, all_or_nothing_sector.S2)
    CAREFUL_GET (data3, all_or_nothing_sector.S3)

```

![all-or-nothing-put](./image/all-or-nothing-put.png)
```C
// Example
if (data1 = data2) and (data2 = data3) return	// State 1 or 7, no repair
if (data1 = data2)
    CAREFUL_PUT (data1, all_or_nothing_sector.S3) return  // State 5 or 6.
if (data2 = data3)
    CAREFUL_PUT (data2, all_or_nothing_sector.S1) return  // State 2 or 3.
CAREFUL_PUT (data1, all_or_nothing_sector.S2) // State 4, go to state 5
CAREFUL_PUT (data1, all_or_nothing_sector.S3  // State 5, go to state 7

```

### Durable Storage: RAID-1
- 磁盘阵列: 多个磁盘 replica
    - 这里的例子总共两个磁盘
- Tolerated error
    - Hard errors reported by careful layer are 
masked by reading from other replicas
- Untolerated error
    - Decay on the same sector of all the replicas, 
status = BAD
    - OS crashes during a DURABLE_PUT
    - Decay in a way that is undetectable

### Improving on RAID-1
- Clerk：周期性检测 decay
    - 周期 T_d 应足够小
- Tolerated error
    - (Old) Hard errors reported by careful layer are masked by reading from one of the other replicas
    - (New) Data of a single decay set decays, is discovered by the clerk, and is repaired, all within Td of the decay event
- Untolerated error
    - The OS crashes during DURABLE_PUT
    - All decay sets fail with Td
    - The data of some sector decays in a way undetectable

### RAID-4 (Dedicated Parity Disk)
![raid-4](./image/raid-4.png)
- 所有信息异或存放于一个单独的硬盘
-  单独一个 disk 坏了可以恢复
- 需要 N+1 个硬盘
- Performance benefits if stripe a single file across multiple data disks
- All writes hit the parity disk

### RAID-5 5 (Spread Out the Parity)
![raid-5](./image/raid-5.png)
- 所有信息异或后分散存储于每个硬盘
- Writes are spread across disks

### Summary
- 系统会出错，我们需要考虑这些错误，建立可靠的、能够容错的系统。但reliability & cost, reliability & simplicity 之间都有 trade off
- 主要实现 reliability 的方式是 redundancy。而 redundancy 的其中一种形式就是 replication ( e.g. replica 能解决 disk failure)
- RAID-5 即防止 single-disk failures 同时性能也很好

# lec-20

## Outline
- Part 1: Fault Tolerance
  - View Stamp
- Part 2: Consistency
  - Linearizability
  - Paxos
- Part 3: Distributed Transaction

## 分布式系统中的容错

### 分布式系统
- For ease-of-use
    - Handle geographic separation 
    - Provide users (or applications) with location transparency 
- For availability 
    - Build a reliable system out of unreliable parts 
- For scalable capacity 
    - Aggregate resource
- For modular functionality
    - Only need to build one service to accomplish one single task well
        - Authentication server 
        -  Backup serve
- 问题就在于复杂度

- Abstraction & Interface
- System Architecture
- Fault Tolerance
- Consistency 一致性
- Performance
    - Latency & Throughput
    - 更多的资源就能得到更好的 Throughput 
    - 而 Latency 则要看性能瓶颈在哪

- 分布式系统正确性的一些问题
    - 同步性
    - 机器挂了
    - 网络挂了

### Strawman fails under concurrency
![strawman-failure](./image/strawman-failure.png)

### Primary determines order of updates
- 可以通过一个主节点 (primary) 进行更新
    - 引入 sequence number
    - 为了保证 replica 的一致性，必须按照一致的顺序写入。因此单独用一个 server 来决定更新顺序，其他 replica 听从主节点

![primary](./image/primary.png)

- 正常流程
    - Primary 收到来自 client 的 op
    - 给所有 backup 发送 op
    - 等待 backup 回复
    - 执行 op 并返回给 client
- Challenge in handling failure
    - Challenge 1: 如果有一个 backup 的 ack 超时怎么办？
    ![challenge1-1](./image/challenge1-1.png)
        - Solution 1: Primary 重试
            - 但是只对暂时性的错误有效
        - Solution 2: 忽略失败的节点
            - 忽略多少个是安全的？如何让被忽略的节点之后跟上？
    - Challenge1-2: 如果 primary 挂了怎么办
    ![challenge1-2](./image/challenge1-2.png)
        - Solution: 切换到其他 primary
            - 如何保证最后一个 op 切换之后不丢？
            - 是否有可能出现两个 primary？
                ![challenge1-3](./image/challenge1-3.png)
            - 如果上图中的网络问题恢复以后，S3, S1 如何重组？

### Viewstamp replication
![log](./image/log.png)
- 每个节点都将 op 记录为 log
    - server 之间通过 log 可以快速比较状态(state)，过时的 server 可以快速同步
- 正确性：每个节点都应该按照 log 顺序执行 op

### VR overview: 如何切换主节点(primary-backup)
![primary-backup](./image/primary-backup.png)
- 通过 view number 来决定哪个节点是 primary
- primary = view_number % total_servers

### VR 正确性的条件
- op 只有在被大多数节点执行后，才会被提交(commit)
- 一旦提交， op 在 log 中的位置就是固定的
- 因此在 log 中同一个位置，不可能有不同的 op 
- view change 发生时，下一个 primary 应该先更新 log，保证自己的 log 是正确的


- Q: 如何防止两个 primary 在log相同位置提交不同的 op
- A: Primary 在 commit 之前，必须等待规定的节点数
(quorum n.法定人数)返回。
- Example
![vr1](./image/vr1.png)
    1. S1 becomes primary of V1 
    2. S1 commits OP1 at pos=1 on S1, S3 
    3. S1 crash, S2 becomes primary of V2 
    4. S2 tries to commit OP1’ at pos=1 
    5. S3 also will receive request OP1’ at pos=1 

    - 在 S2 成为 primary 的时候，必须知道上一个 view 的 log 是什么样的（问剩下的 majority 要 log）
    - 但ppt这个例子不好，s1 崩了，s2 log不是最新，只有 s3 正常，这个时候正常的机器数量为 1，集群其实已经挂了……

# lec21
## 回顾一下 VR
1. 正确性：每个节点都按照 log 中的顺序执行 op
    - 只有在被大多数节点执行后，才写入 log
    - **加入到 log 不等于 commit**
2. 因此，两个不同的 op 不可能位于 log 中同一位置
3. 所有在 view V-1 中被提交的 op，必须被 view V 的 primary 知道。
    - view change 之后，新的 primary 更新完 log 之后，新的 view 才进入正常状态

## 如果 Primary 没有收到 majority 的答复？
- 共 2F + 1 个节点
- Case 1. 超过半数 (F+1) 节点挂了
    - The system will stuck
- Case 2. 超过半数 (F+1) 节点还活着
    - 发起 viewchange，old primary 在网络恢复后会跟上进度

## Basic VR protocol
![basic-vr](./image/basic-vr.png)
- curV: 当前 view number
- status: NORMAL, VIEW-CHANGE, RECOVERY
- op-num: log 中最后一个 op 的编号
- commit-num: 最后一个提交的 op 的编号

![basic-vr1](./image/basic-vr1.png)
- server 收到 prepare 消息后
    - 如果 `curV != msg.curV or status != NORMAL` 就忽略 req
    -  等待它自己的 log 变为最新之后 (wait until it has entries in its log for all earlier requests )
    - 将该 req 加入到 log 的 msg.op-num 位置
## Normal Process
![vr-normal](./image/vr-normal.png)

## Recovery
- 某个节点收到 prepare 消息后，如果发现它的 curV < msg.curV，那么要给其他节点发 recovery 请求
- 其他节点收到 recovery 请求后，如果该节点 NORMAL，就返回 curV, x, log, op-number, commit-number

![recovery1](./image/recovery1.png)
![recovery2](./image/recovery2.png)
![recovery3](./image/recovery3.png)

## View Change
![viewchange1](./image/viewchange1.png)
![viewchange2](./image/viewchange2.png)
![viewchange3](./image/viewchange3.png)
![viewchange4](./image/viewchange4.png)

- Problem： viewchange 发生时，如何决定新的 primary 采用哪个 log ?
- Solution-1： 最长的 log ?
![longest-log](./image/longest-log.png)
- Solution-2: curV 最大的 log ?
![biggest-curV](./image/biggest-curV.png)

因此引入新的状态变量 `lastV` ：上一个 view-number

- final Solution：
    - Rule 1：采取 lastV 最大的 log
![viewchange5](./image/viewchange5.png)
![viewchange6](./image/viewchange6.png)
![viewchange7](./image/viewchange7.png)
![viewchange8](./image/viewchange8.png)
![viewchange9](./image/viewchange9.png)
![viewchange10](./image/viewchange10.png)
    - Rule 2: 如果根据 Rule 1 得到的 log 有复数个，那么选最长的。

## Summary
![vr-summary](./image/vr-summary.png)
- 最开始，每个 backup 与 primary 是一致的
- 同一个 view 中，每个 op 一定被大多数节点记录在 log 的相同位置
- 同一个 view 中，backup 的 log 一定是 primary 的 log 的 prefix
- 在任何 majority 中，必定有一个 backup 的 log 是正确的（包含所有提交过的 op）
- 切换到新的 view V 的 primary ，一定能够获取到 viw V-1 所有提交过的 op

![vr-summary2](./image/vr-summary2.png)

# lec 22 VR exercise
## Exercise 1
![vr-exercise-1](./image/vr-exercise-1.png)
- 最后 view number 是多少? Primary 的 log 是什么样的？
- 重点在于 S1, S2 什么时候 Recovery
  - 第七步，S1 尝试 commit OP2,3,4，那么就需要大多数节点回复他，这个时候 S1 收到大多数的节点回复发现自己的 view 不是最新的，开始 Recovery。等 S4,S5 view change 做完，进入 NORMAL 状态后，S1 就可以继续 Recovery。(注意：此时 S2 没有收到其他节点的信息，不会发起 Recovery)
![vr-exercise-1-1](./image/vr-exercise-1-1.png)
![vr-exercise-1-2](./image/vr-exercise-1-2.png)
![vr-exercise-1-3](./image/vr-exercise-1-3.png)
![vr-exercise-1-4](./image/vr-exercise-1-4.png)
![vr-exercise-1-5](./image/vr-exercise-1-5.png)
  - S2 应该在 第八步 OP7 被提交时，进行的 Recovery。这时 S4 收到 OP 7 的请求，把 OP 7 发给各个节点，S2 这时才发现自己不是最新的 view，进行 Recovery
![vr-exercise-1-6](./image/vr-exercise-1-6.png)
![vr-exercise-1-7](./image/vr-exercise-1-7.png)

## Exercise 2
![vr-exercise-2](./image/vr-exercise-2.png)

| curV | lastV | status |
|------|-------|--------|
| 1 | 1 | NORMAL |
| 2 | 2 | NORMAL |
| 2 | 2 | NORMAL |

## Exercise 3
![vr-exercise-3](./image/vr-exercise-3.png)

(比如 S1 执行了 op 3，请求没发给 S2)

| curV | lastV | status |
|------|-------|--------|
| 1 | 1 | NORMAL |
| 1 | 1 | NORMAL |
| 5 | 5 | NORMAL |
| 5 | 5 | NORMAL |
| 5 | 5 | NORMAL |

## Exercise 4
![vr-exercis-4](./image/vr-exercise-4.png)
- op 2 & op 3 最终是否会被提交？
- 如果是，给出最小的 commited latency (其实这题上课的时候他自己都说有问题……latency 涉及到具体实现，实现不同 latency 也不同)
- 从 S1 断开连接，发生 viewchange 开始
![vr-exercis-4-1](./image/vr-exercise-4-1.png)
![vr-exercis-4-2](./image/vr-exercise-4-2.png)
![vr-exercis-4-3](./image/vr-exercise-4-3.png)

# lec 23 Linearizability 线性化

- 系统实际实现
  - 并发执行
  - 多份数据
- 使用者眼中的系统
  - 串行执行
  - 只有一份数据

## Strawman key-value store
![key-value-strawman](./image/strawman-key-value.png)

## 如何定义一个健壮的一致性模型(strong consistency model)
- 并不是所有可能的等价的执行顺序都 make sense

- Global issue order
  - 全局上，请求发起的顺序。即下图中 A,B,C,D 四点的先后顺序
  ![global-issue-order](./image/global-issue-order.png)
- Global completion order
  - 全局上，请求处理完毕的顺序。即下图中 A,B,C,D 四点的先后顺序
  ![global-completion-order](./image/global-completion-order.png)
- per-thread issue(or completion)
  - 只要求每个 thread 内部的 issue(or completion) 顺序符合
- Global completion-to-issue order
  - 以下图为例，global completion-to-issue order 只要求头尾完全错开的任务，图中的 Put(x,1) 在 Get(x)=0 前面即可。
  ![global-completion-to-issue](./image/global-completion-to-issue.png)

然而保证 global issue/completion order 很不切实际。

![strong-consistency-model](./image/strong-consistency-model.png)

- Sequential Consistency
  - 必须遵循 per-thread issue/completion order
- Linearizability
  - 必须遵循 global completion-to-issue order

![consistency-order](./image/consistency-order.png)

## Sequential consistency vs Linearizability

- Does the difference between them matter?
  - 只有在线程间有相互通信时才影响 Matters only when external communicating between threads
- 两者都对于 w/o (write only) 很好
  - 多线程程序共享 sequential consistent global memory

## 为什么 Linearizability 是一个健壮的一致性模型
- 能够达到的，实际的，最健壮的模型
- It allows scalable sytstem design
  - 如果每个 object 都是 Linearizable 的，那么整个系统就是 Linearizable 的
  - Scale system -> shared objects across servers

## Linearizability exercise
![linearizability-exercise](./image/linearizability-exercise.png)

## Linearizability 的实现
- Case-study: key-value store
  - N servers: server-i stores keys, k mod N = i
  - 3 servers replicating data using viewstamp replication
  ![linearizability-implementation](./image/linearizability-implementation.png)

## What we've learnt until now?
- 健壮的一致性模型：Linearizability
  - execution is equivalent to a serial history that preserves global completion-to-issue order
- 如何实现 Linearizability
  - No replication
  - viewstamp replication

## Consensus(一致同意) - Consistency Replication(前后一致)

Paxos 解决了 consensus 的问题(其实vr也是)。我们先不看 Paxos 具体是如何实现的，先自己尝试解决这个问题。

## Single Decree Consensus
- Problem setting
  - N 个节点，每个可能的输入都不一样（决策不一样）
  - 很多节点都有可能复杂地崩溃
  - 网络是异步的
- Goal
  - 安全性：最终不同节点选择的值都是一样的
  - Liveness：运行中的节点最终都选了某个值
  
![single-decree-consensus](./image/single-decree-consensus.png)

## Paxos Components
- Roles
  - Proposer: 发送提议的值给 Acceptor
  - Acceptor: 接受/拒绝 Proposer 提出的值
  - Learner: learn about outcome of consensus (chosen value)
  - 为了简单起见，每个节点担任所有角色（虽然我觉得这并不简单）
- Chosen
  - 如果某个值被绝大多数节点接受，那么这个值被 chosen

## Basic idea of Paxos
- Liveness
  - Acceptor 可以接受多个 proposals
- Consensus (Safety)
  - Proposer 只会提议 safe value
    - 如果某个 proposal 的值为 p，这个 proposal 被选择了，那么 p 就是 safe value
    - **如果没有任何一个 proposal 被接受，那么，任何值都是 safe value**

- 问题来了：在异步网络下，怎么才能知道 safe value

## Discover the safe value with an extra round trip(Prepare Phase) in asynchronous network

![discover-safe-value-1](./image/discover-safe-value-1.png)
![discover-safe-value-2](./image/discover-safe-value-2.png)
![discover-safe-value-3](./image/discover-safe-value-3.png)
![discover-safe-value-4](./image/discover-safe-value-4.png)

不过我们还是不知道该选哪个 proposal

## Totally ordered proposals
- 每个 Proposal 都有一个全局唯一的 id  (proposal/ballot number)
  - 比如通过 cascade(local counter, server id) 实现
- 如果 Proposal p 被选择了，那么对于所有 p' > p，都有 p'.value = p.value
  - 保证 proposal p 被接受后，所有之后被接受的 proposal p' 必须有相同的值
- 如果 p' < p (p 是 Acceptor 收到的最新的 proposal)，那么意味着 p' 是过时的 proposal，应该拒绝。
  - Tolerate the asynchronous network

## Basic Paxos - two phase protocol (正式提出 Paxos 算法)
- Phase 1: Prepare
  - Proposer:选择一个 proposal number，并且 (Read Quorum) 找到 safe value
  - Acceptor:记录他接收到的最新的 proposal number

- Phase 2: Accept
  - Proposer: 将 safe value 发给所有 Acceptor，如果大多数界都都同意了就 commit
  - Acceptor: 如果收到的 proposal number 大于等于他记录的 proposal number 那么就接受它

- Safe value
  - 在 read quorum 中，safe value 有着最大的 accepted ballot number
  - read quorum 和 write quorum 的交集不是空集
    - e.g. RQ = WQ = majority

- Example
  - 米妮与其他节点间有 delay
  - ballot number 表示先后顺序
![paxos-1](./image/paxos-1.png)
![paxos-2](./image/paxos-2.png)
  - 由于 delay，米奇没有获得米妮那边的情况。但是米奇+唐老鸭一共两个节点已经是大多数(Read quorum)，所以正常运行
  - 米奇与唐老鸭都没有选择，所以任何值都是 safe value，米奇 propose KFC
![paxos-3](./image/paxos-3.png)
![paxos-4](./image/paxos-4.png)
  - 唐老鸭成功获取了米妮的情况
  - 这里唐老鸭和自己之间少画了一根线
![paxos-5](./image/paxos-5.png)
  - 米妮的 ballot number 较小，所以 safe value 是 KFC
![paxos-6](./image/paxos-6.png)
  
- Basic Algorithm
  ![paxos-basic-alg](./image/paxos-basic-alg.png)

## Multi-Decree Consensus: from Paxos to MultiPaxos

之前的 Paxos 只针对单一任务的一致性：去哪里吃饭。现在要拓展至多任务。

![multi-paxos](./image/multi-paxos.png)

- 最 Naive 的想法：
  - 先把任务 Lunch 使用 basic Paxos(Single-decree)统一一致性；再把任务 Entertain 使用 basic Paxos(Single-decree)统一一致性。这样的话每个 Paxos 节点都有他自己的 h_n, a_n, a_v 状态

- 最终决定
  - 如果我们让其中一个 server 作为 default/distinguished proposer，那么所有的节点，所有的节点都共享一个相同的 h_n （类似于 vr 里的 primary）
  - 并且采用 batch prepare

- Example
  - ballot number
    - 米妮:1
    - 米奇:2
    - 唐老鸭:3
  - 首先某个节点通过 prepare(ballot) 成为 distinguished leader
  ![multi-paxos-1](./image/multi-paxos-1.png)
  - 每个 Acceptor 返回他的 log entries(这里返回所有的 log，但实际实现时，似乎并不是返回所有的)
  - 这里 log 为空，所以先返回 null
  - distinguished leader 根据返回结果，更新自己的 log
  ![multi-paxos-2](./image/multi-paxos-2.png)
  - distinguied leader 通过 accept(ballot, value, index) 向其他节点提议
  ![multi-paxos-3](./image/multi-paxos-3.png)
  - 每个 Acceptor 返回 OK 以及 log entry
  ![multi-paxos-4](./image/multi-paxos-4.png)
  - 此时可以认为，这条 proposal 被 majority 接受。(红色标识为 commit，即被 majority 接受)
  ![multi-paxos-5](./image/multi-paxos-5.png)
  - 之后进行下一个提议。此时唐老鸭发生了某些故障。不过米妮+米奇已经是 majority，最终 Shopping 仍被 commit.
  - **注意**: 目前只发送过一个 prepare 请求，这里采用的是 batch prepare
  ![multi-paxos-6](./image/multi-paxos-6.png)
  ![multi-paxos-7](./image/multi-paxos-7.png)
  ![multi-paxos-8](./image/multi-paxos-8.png)
  - 同理 卡拉OK 被 commit
  ![multi-paxos-9](./image/multi-paxos-9.png)
  - 这时，米奇开始发起 prepare(2)
  ![multi-paxos-10](./image/multi-paxos-10.png)
  - 唐老鸭返回他的 log entry of 0 and 2
  ![multi-paxos-11](./image/multi-paxos-11.png)
  - 米奇根据唐老鸭返回的 log 更新自己的 log
  ![multi-paxos-12](./image/multi-paxos-12.png)
  - 米奇给唐老鸭发 accept 1 and 2
  ![multi-paxos-13](./image/multi-paxos-13.png)
  - 米奇之前受到来自米妮的 h_n=1 的 shopping，仅仅是记录 h_n=1 并没有 commit。此时他自己 propose ，提议h_n=2 的 shopping
  ![multi-paxos-14](./image/multi-paxos-14.png)
  - 唐老鸭返回 OK，米奇这里确认 majority 写入了，因此 commit Shopping & Karaoke
  ![multi-paxos-15](./image/multi-paxos-15.png)
  - 然后是最后一个 McDonald
  ![multi-paxos-16](./image/multi-paxos-16.png)
  ![multi-paxos-17](./image/multi-paxos-17.png)
  ![multi-paxos-18](./image/multi-paxos-18.png)

- Algorithm
  ![multi-paxos-alg](./image/multi-paxos-alg.png)

## Exercise 1
![paxos-ex-1](./image/paxos-ex-1.png)

待定

## 实际应用中的强一致性模型
- Intel CPU 内存模型？
- 主流的 key-value stores
  - S3
  - Dynamo
  - MySQL with async replication

# lec 24 Transaction
New Abstraction: Atomicity & Isolation

原子性与独立性

## Transaction
### 目标
- 目标: 在不可靠的组件上建立可靠的系统（容错）
- 为了实现这个目标，之前我们提出了 Consistency
- 现在提出两个新的 abstraction: Atomicity 和 Isolation

- Example
![transaction-1](./image/transaction-1.png)
- Atomicity: All-or-nothing. T1 T2 要么运行完毕，要么根本没执行
- Isolation: 同步执行时，T1 T2 表现的像串行执行一样
- 这两点能够帮助解释 failures( and concurrency)

### Commit Point
![commit-point](./image/commit-point.png)

- Example
    - Where is the commit point?
    ![commit-point-ex](./image/commit-point-ex.png)
    - between 5 and 6

## Shadow Copy

### Example: Bank Account Transfer
```python
xfer(bank, a, b, amt):
    # sum = 200
    bank[a] = bank[a] – amt
    # Here is the problem
    # sum = 150 
    bank[b] = bank[b] + amt 
    # sum = 200

audit(bank):
    sum = 0
    for acct in bank:
        sum = sum + bank[acct]
    return sum
```

修改 xfer：在当前数据的 copy 上先做修改，修改完后，用 copy 覆写当前数据。（word, vim 等编辑器也采用这种方法）

```python
xfer(bank, a, b, amt):
    bank[a] = read_accounts(bankfile)
    bank[a] = bank[a] – amt
    bank[b] = bank[b] + amt
    write_accounts("#bankfile") 
    rename("#bankfile", bankfile)
```

#### First Try: rename 初步实现

- Step 1:
    - Directory data blocks:
        - filename "bank" → inode 12
        - filename "#bank" → inode 13
    - inode 12:
        - data blocks: 3, 4, 5
        - refcount: 1
    - inode 13:
        - data blocks: 6, 7, 8
        - refcount: 1
- Step 2: 简略起见，只写出有修改的部分
    - Directory data blocks:
        - filename "bank" → inode 13
- Step 3:
    - inode 13:
        - refcount: 2
- Step 4:
    - inode 12:
        - refcount: 0
- Step 5:
    - Directory data blocks:
        - filename "bank" → inode 12
        - ~~filename "#bank" → inode 13~~
- Step 6:
    - inode 13:
        - refcount: 1

问题：如果崩溃了，哪一个 step 会引起问题？
- Step 2 就有问题
    - 两个 file 都指向 inode 13，但是 inode 13 的 refcount = 1，哪一个才是正确的？

#### Second Try: Increase ref-count first
```python
rename(x, y):
   newino = lookup(x)
   oldino = lookup(y)

   incref(newino)
   ... # change y's dirent to newino
   decref(oldino)
   ... # remove x's dirent
   decref(newino)
```

Q：commit point 在哪里 ?
A：修改 y 的 dirent 之后。

#### Recovery After Crash
```python
salvage(disk):
  for inode in disk.inodes:
    inode.refcnt =
      find_all_refs(disk.root_dir, inode)
  if exists("#bank"):
    unlink("#bank")
```

### Shadow Copy
- 写入都发生在数据的拷贝中，写完后将当前数据切换到拷贝中
- 因为切换(switching)是可以用一步 all-or-nothing 的操作(sector write)实现
- 需要底层的一些原子性实现的支持(disk)
- Main rule: 只对一份 copy 作一次写入
    - In our example: serctor write for rename
    - Creates a well-defined commit point

- Shadow Copy 是否能满足各种应用场景？
    - Pros
        - Works well for a single file
    - Cons
        - Hard to generalize to multiple files or directories
            - Might have to place all files in a single directory, or rename subdirs
        - Requires copying the entire file for any (small) change
        - Only one operation can happen at a time
        - Only works for operations that happen on a single computer, single disk

## Logging for all-or-nothing
- Transactions 提供了 atomicity & isolation 两种抽象，能够使得我们的目标更容易实现，但是并不保证性能
![logging-for-atomicity](./image/logging-for-atomicity.png)

- 而且我们希望 transaction-based system 能够在分布式地使用

### Example: Bank Account App (Shadow Copy)
```python
xfer(bank, a, b, amt):
	copy(bank, tmp)
	tmp[a] = tmp[a] – amt
	tmp[b] = tmp[b] + amt
	rename(tmp, bank)
```

### Transaction Terminology
![bank-transaction](./image/bank-transaction.png)


- Two accounts: A and B
    - Accounts start out empty
    - Run these all-or-nothing actions
![bank-crash](./image/bank-crash.png)
- Problem
    - After crash, A=110, but T3 never committed
- Solution
    - Revert to A's previous committed value
    - T1 and T2: "All"
    - T3: "Nothing"


### Log
- A log sample
![log-sample](./image/log-sample.png)

- 共涉及到五个操作
    - Begin
    - Write variable
    - Read variable
    - Commit
    - Abort
1. Begin: allocate a new transaction ID
2. Write variable: append an entry to the log
3. Read
    - As an aside: how to see your own updates?
    - Read uncommitted values from your own TID

```python
read(log, var): 
  commits = {} 
  # scan backwards 
  for record r in log[len(log) - 1] .. log[0]:
    # keep track of commits 
    if r.type == commit: 
      commits.add(r.tid) 
      # find var's last committed value 
      if r.type == update and
         r.tid in commits and 
         r.var == var: 
           return r.new_value 
```

```python
read(log, var): 
  commits = {} 
  # scan backwards 
  for record r in log[len(log) - 1] .. log[0]:
    # keep track of commits 
    if r.type == commit: 
      commits.add(r.tid) 
      # find var's last committed value 
      if r.type == update and
        (r.tid in commits or r.tid == current_tid) and 
         r.var == var: 
           return r.new_value 

```
4. Commit: write a commit record
    - 我们希望每从 commit record 都是这次操作的 Commit Point
    - However, writing log records better be all-or-nothing
        - One approach, from last time: make each record fit within one sector
5. Abort: Do nothing or write a record?
    - 我们目前这种方式，可以写入一个 abort 记录，但是没必要

- Recover from crash: do nothing

### 性能优化
- Keep both a log and cell storage(类似于 cache)
    - Log as before: authoritative, provides all-or-nothing atomicity
    - Cell storage: provides fast reads, but cannot provide all-or-nothing
    - Both are stored on disk

### Terminology
- "log" an update when it's written to the log
- "install" an update when it's written to cell storage

### Cell Storage
![cell-storage](./image/cell-storage.png)

### Read / Write with Cell Storage
- 一个 update 来了以后，如何同时更新 log 和 cell storage ?
- 崩溃以后，如何根据 log 恢复 cell storage
    - 尤其是在 logging 和 installing 之间崩溃

![cell-storage-1](./image/cell-storage-1.png)

```python
read(var):
    return cell_read(var)

write(var, value):
    log.append(cur_tid, update, var, read(var), value)
    cell_write(var, value)
```

### Order Matters
- logging 和 installing 的顺序
    - The logging and installing together do not have all-or-nothing atomicity
    - 很可能发生在两者之间
- 如果我们先 install 后 log
    - 崩溃的话，没法复原 cell storage
- 定义 logging 的规则 "Write-ahead-log protocol" (WAL)
    - 先 log 再 install
- 如果崩溃以后，log 是完好且可靠的，能够用 log 来修复 cell storage

### Recovering Cell Storage
- 现在我们先 log 后 install
    - 如果此时 abort/crash 了，就应该 undo
    - Plan: 扫描整个 log，决定哪些 actions 应该被 abort，把它们 undo
- 从后往前扫描
    - 需要从最新的一直 undo 到最旧的
    - 在我们决定要 undo 之前，需要先知道这些 action 的结果

```python
1   recover(log): 
2       commits = {} 
3       for record r in log[len(log)-1] .. log[0]: 
4           if r.type == commit:
5               commits.add(r.tid) 
6           elif r.type == update and r.tid not in commits:
7               cell_write(r.var, r.old_val) # undo 
```

![recovery-with-cell-storage](./image/recovery-with-cell-storage.png)
执行 4, 6, 7 行。将 A undo 为 80

![recovery-with-cell-storage-1](./image/recovery-with-cell-storage-1.png)
执行 4, 5 行。将 T2 加入 commit

![recovery-with-cell-storage-2](./image/recovery-with-cell-storage-2.png)
同理执行 4, 5 行。将 T1 加入 commit

![recovery-with-cell-storage-3](./image/recovery-with-cell-storage-3.png)

### Performance Now
- **Write**s might still be OK
    - but we do write twice: log & install
- **Read**s are fast
    - look up the cell storage
- **Recovery** requires scanning the entire log
- Remaining performance problems
    - We have to write to disk twice
    - Scanning the whole log will take longer as the log grows

### Optimization 1: Improve Writes
- 将 cell storage 存在 volatile cache, e.g. memory
    - 写入也可以很快：just one write instead of two
    - Hope that variable is modified several times in cache before flush

- Reads 同样更快
    - Reads go through the cache, since cache may contain more up-to-date values

- Read & Write with Cache
![cell-storage-2](./image/cell-storage-2.png)
```python
read(var):
  if var in cache:
    return cache[var]
  else: # may evict others from cache to cell storage
    cache[var] = cell_read(var)
    return cache[var]
write(var, value):
  log.append(current_tid, update, var, read(var), value)
  cache[var] = value
```

- Problem Brought by Cache
- Atomicity problem: cell storage (on disk) may be **out-of-date**
    - 是否有可能，有些本来应该在 cell storage 里的改变，没在里面？
        - Yes: might not have flushed the latest commits
    - 是否有可能，有些本来不应该再 cell storage 里的改变，在里面
        - Yes: flushed some changes that then aborted (same as before)
- Solution: change the recover procedure
    - 除了 undo 还需要 redo
    - undo 倒序；redo 正序
    - Do not treat actions with an abort record as "done"
        - There might be leftover changes from them in cell storage
- Recover with Cache
![recover-with-cache](./image/recover-with-cache.png)
```python
recover(log):
  commits = {}
  for record r in log[len(log)-1] .. log[0]:
    if r.type == commit:
      commits.add(r.tid)
    if r.type == update and r.tid not in commits:
      cell_write(r.var, r.old_val) # undo
  for record r in log[0] .. log[len(log)-1]:
    if r.type == update and r.tid in commits:
      cell_write(r.var, r.new_value) # redo
```

### Optimization 2: 截短日志 Truncate the Log
- 现在 log 会无限增长: not practical
    - 哪部分 log 能够被丢弃？
        - Must know the outcome of every action in that part of log
        - Cell storage must reflect all of those log records (commits, aborts).
    - Truncating mechanism (assuming no pending actions):
        - Flush all cached updates to cell storage
        - Write a **checkpoint** record, to save our place in the log
        - Truncate log prior to checkpoint record
        - (Often log implemented as a series of files, so can delete old log files)
    - With pending actions, delete before checkpoint & earliest undecided record

### Checkpointing
- Do the following:
    - (Prepare) Stop accepting new transactions
    - (Prepare) Wait until all current transactions commit or abort and have written the COMMIT or ABORT to the log
    - (Checkpointing) Flush the log to disk
    - (Checkpointing) Write a log record <CKPT> and flush the log again
    - (End) Resume accepting transactions

- Problem with checkpointing
    - The system must be **stop**ped until operations have committed or aborted
### **Non-quiescent** checkpointing (动态 checkpoint)
- Write a record <START CKPT(T1…Tk)> and flush log (T1,…Tk are the active transactions)
- Wait until all of T1, … Tk commit or abort, but allow new transactions to start
- When all of T1,…Tk have written COMMIT or ABORT, then write <END CKPT> to the log

- If crash, then look backwards for the first <START CKPT(T1…Tk)> or <END CKPT>
    - If see an <END CKPT>, only have to consider after this
    - If see a <START CKPT(T1…Tk)> but no <END CKPT>, then only need to consider after the transactions T1, .. Tk began

### Optimization-3: External Synchronous I/O
- Problem: flush to disk is slow
    - Lazy writing, even for the log file
    - some app, e.g. database, calls syns() to ensure flushing
    - But sync() is slow...
- Solution: External Sync
    - Once the kernel return from sync(), the file may not be flushed. 
    - It will be flushed if something externally visible happens
        - e.g. print to the user, network sending, serial port outcome, etc.

# lec 26
- Serializability
    - Final-state serializability
    - Conflict serializability
    - View serializability

## Conflict serializability
- Two operations conflict if :
    - they operate on the same object, and 
    - at least one of them is write
    - the actions belong to different transactions
![conflict](./image/conflict.png)

## Conflict Graph
- Nodes are transactions, edges are directed
- Edge between Ti and Tj if and only if:
    - Ti and Tj have a conflict between them and
    - the first step in the conflict occurs in T i
- 如果某个排序的 Conflict Graph 是有向无环图，那么它满足 Conflict Serializability

![conflict1](./image/conflict1.png)
![conflict2](./image/conflict2.png)
![conflict3](./image/conflict3.png)
![conflict4](./image/conflict4.png)

### View Serializability
![view-serializability](./image/view-serializability.png)
- 定义
    - A schedule is view serializable if the final written state as well as intermediate reads are the same as in some serial schedule
![serializability](./image/serializability.png)

### 为什么关注 Conflict Serializability
- Conflict serializability is easy to test
    - 检验有向图是否成环很简单
    - 检验 View serializability 是 NP-hard
- Conflict serializable schedules are easy to generate
    - Using 2-phase locking
- Conflict serializable schedules are also view serializable
- Schedules that are view serializable but not conflict serializable involve blind writes: 
    - Blind writes: writes that are ultimately not read, which are not common in practice

## Generate Conflict-Serializable Schedules
### How to generate
- 悲观
    - 假设许多干扰
    - 主动防止冲突产生
    - 2-phase locking
- 乐观
    - 允许任何写入
    - 检查冲突，如果有，则 abort
    - e.g. OCC

### Global Lock
- 全局一把锁
- 略

### Simple Locking
- Simple locking
    - 提前获取每个共享变量的锁
    - commit 或 abort 之后放锁
- Lock point
    - Lock set: locks acquired when reaches lock point
    - ock manager's enforcement
        - Intercept read/write/commit/abort, and check
- Problem
    - How to enumerate all shared object to access?
    - The set of might-access may be larger than does-access

### Two-phase Locking
- 每个共享变量都有锁
- 每个transaction在做操作之前，先把对应的锁那好
- transaction 一旦放锁，就再也不能拿锁

### 优化：读写锁 Read-write locks
- 多个 reader ，单个 writer
    - 适合有大量读操作的应用
    - A writer may be delayed indefinitely
    - Can release reader-locks after lock point (may before commit)

### 2PL 证明
![proof-of-2pl](./image/proof-of-2pl.png)

### 2PL 会导致死锁
- Solution
    - global ordering on locks
    - or just abort one transaction

## OCC
### Concurrency Control(CC)
- CC 保证同步操作的正确性
    - 保证 serializability
    - high throughput and scalability

### OCC Executes a Transaction in Three Phases
- Phase 1:Concurrent local processing  
    - 要读的变量读进 read set
    - 要写的变量写进 write set
- Phase 2:  Validation in critical section
    - 检查 serializability 是否满足:
        - Has any tuple in the read set been modified?
- Phase 3: Commit the results in critical section or abort
    - Aborts: aborts the transaction if validation fails
    - Commits: installs the write set and commits the transaction
- Example
![occ-example](./image/occ-example.png)

- Advantage
    - 竞争较少时，性能很好

### Data Dependencies
- wr: T2 reads a tuple that was earlier written by T1
- ww: T2 writes a tuple that was earlier written by T1
- rw: T1 reads a tuple that was later written by T2

### OCC: Serializable Scope
- Ensures serializability by breaking the incoming RW edge of the earliest committed transaction in a possible cycle
![serializable-scope](./image/serializable-scope.png)

### Problem: False Aborts
- 一些被 OCC abort 的 transaction 可能并没有打破 serializability
![false-abort](./image/false-abort.png)

### Transactional Memory
- 大多都提供
    - Opportunistic concurrency
    - Strong atomicity: read set & write set
    - Semantic of both *all-or-nothing* and *before-or-after*
- Real-world best-effort TM
    - Limited read/write set
    - System events may abort an TX (TX? maybe TM)

### Programming with RTM
- If transaction start successfully
    - Do work protected by RTM, and then try to commit
- Fallback routine to handle abort event
    - If abort, system rollback to _xbegin, return an abort code
- Manually abort inside a transaction

```python
if _xbegin() == _XBEGIN_STARTED: 
    if conditions:
        _xabort()
    critical code
    _xend()
else
    fallback routine
```

# lec 27 Distributed Transaction
Across multiple sites

## multi-site Transaction
![multi-site-transaction-1](./image/multi-site-transaction-1.png)

## Two-phase Commit
- Phase-1: preparation / voting
    - Lower-layer transactions either aborts or tentatively committed
    - Higher-layer transaction evaluate lower situation
- Phase-2：commitment
    - If top-layer, then COMMIT or ABORT
    - If nested itself, then become tentatively committed

![multi-site-transaction-2](./image/multi-site-transaction-2.png)

commit 之前先确认是否 ready

## Multiple-site Atomicity
- Worker: Bob, Charles, Dawn
    - Does three transactions: X, Y, Z
- Coordinator: Alice 
    - Create a higher-layer transaction
    - Send three messages to the three workers
- Challenge: **un-reliable communication**

![multi-site-transaction-3](./image/multi-site-transaction-3.png)

- Coordinator
    - Collect some ABORT or nothing: ABORT or assign the work to another worker
    - Collect all COMMIT: then COMMIT
- Worker
    - PREPARED 发出之后，如果什么都没收到：resend PREPARED
        - Coordinator will send current state if it receives duplicate message
    - 如果收到 COMMIT: then COMMIT

- Failure: lost prepare
![lost-prepare](./image/lost-prepare.png)
    - coordinate 没有到 A-M 回复，由 coordinate 重发 prepare
- Failure: lost ack for prepare
![lost-ack-for-prepare](./image/lost-ack-for-prepare.png)
    - 仍然由 coordinate 重发 prepare
- Failure: worker failure during prepare
![worker-failure](./image/worker-failure.png)
    - 根据心跳，发现 worker 崩溃，coordinate 发 abort
- Failure: lost commit 
![lost-commit](./image/lost-commit.png)
    - coordinate 重发 commit
- Failure: lost ack for commit 
![lost-ack-for-commit](./image/lost-ack-for-commit.png)
    - coordinate 重发 commit
- Failure: worker failure during commit 
![worker-failure-in-commit](./image/worker-failure-in-commit.png)
    - 这时我们不能够简单的 abort
    - 必须将 worker 恢复至 prepared 状态
    - prepared 之后 worker 将 PREPARE 记录写入 log
    - 读取 log
    - 推测哪个 transaction prepare 但是没有 commit
    - 然后 worker 给 coordinate 发消息，让 coordinate 重发 commit
![worker-failure-in-commit2](./image/worker-failure-in-commit2.png)
- Failure: coordinator failure during prepare
![coordinator-failure-during-prepare](./image/coordinator-failure-during-prepare.png)
    - coordinator 恢复以后，给 worker 发 abort
- Failure: coordinator failure during commit 
![coordinator-failure-during-commit](./image/coordinator-failure-during-commit.png)
    - coordinator 恢复以后，给 worker 发 commit

### Summary of 2-phase Commit
- 2-phase Commit 解决了 mult-site atomicity
- 发生在 commit point 之前的 failure 都要 abort；发生在 commit point 之后的，等恢复以后继续
- Availability 和 replication 其实还有问题没解决：通过 replica 的方式提高 availability，必须先解决多份数据的一致性

# lec 27 (2) Security

## 安全系统中两个重要的概念
- Policy
- Threat Model

### Policy: Goals
- Information security goals：
    - Privacy：限制谁能读取数据
    - Integrity：限制谁能写数据
- Liveness goals：
    - Availability：保证服务运行

### Threat Model：Assumptions
- What dose a threat model look like
    - Adversary controls some computers, networks (but not all)
    - Adversary controls some software on computers he doesn't fully control
    - Adversary knows some information, such as passwords or keys (but not all)
    - Adversary knows about bugs in your software?
    - Physical attacks?
    - Social engineering attacks?
    - Resources?  (Can be hard to estimate either resources or requirements!)

## Guard Model

### Guard Model of Security
- Complete mediation 
    - 所有对于资源的访问都要经过 guard
- Only way to access the resource involves the guard
1. Must enforce client-server modularity
    - Adversary should not be able to access server's resources directly
    - E.g., assume OS enforces modularity, or run server on separate machine
2. Must ensure server properly invokes the guard in all the right places

![guard-model](./image/guard-model.png)

- *Authentication*: is the principal who they claim to be?
- *Authorization*: does principal have access to perform request on resource?

### Design the Guard
- Authentication: request -> principal
    - E.g., client's username, verified using password
- Authorization: (request, principal, resource) -> allow?
    - E.g., consult access control list (ACL) for resource
- Simplifies security
    - Can consider the guards under threat model
    - But don't forget about complete mediation

#### Example: Unix FS
- Resource:	files, directories.
- Server:	OS kernel.
- Client:	process.
- Requests:	read, write system calls.
- Mediation:	U/K bit / system call implementation.
- Principal:	user ID.
- Authentication:  kernel keeps track of user ID for each process.
- Authorization:   permission bits & owner uid in each  file's inode.
#### Example: Firewall
- Resource: 	internal servers
- Client: 	any computer sending packets
- Server: 	the entire internal network
- Requests: 	packets
- Mediation:
    - internal network must not be connected to internet in other ways
    - no open wifi access points on internal network for adversary to use
    - no internal computers that might be under control of adversary
- Principal, authentication: none
- Authorization: check for IP address & port in table of allowed connections
### What can go Wrong
1. Bypass complete mediation by software bugs
2. Bypass complete mediation by an adversary
3. Policy vs. mechanism
4. Interactions between layers, components
5. Users make mistakes
6. Cost of security

### Bypass complete mediation
减少代码行数，是代码易于维护，bug更少

# lec 28 
## Authentication
### Principle of Least Privilege
- Deal with bugs / incomplete mediation
    - 任何 component 对于资源的访问都要经过 guard
    - 如果这个 component 有 bug 或者设计有问题，会导致 incomplete mediation
    - 因此减少 component 的数量

### Security Jargon：Trust
- Trusted is bad：如果信任的 component 崩溃了，gg
- Untrusted components are good：崩了也无所谓
- 好的设计中少有 Trusted component

### Policy VS. Mechanism
- 高层次的 policy 是简明、清楚的
    - Security mechanism(e.g. gurads) 提供低层次的保障
    - E.g., policy is that students cannot get a copy of exam.txt
        - What should the permissions on the files be?
        - What should the firewall rules be?
- Good idea: try to line up security mechanisms with desired policies

### Interactions between layers
```sh
cse-srv% cd /2012f/bob/project
cse-srv% cat ideas.txt
Hello world.
…
cse-srv% mail alice@sjtu.edu.cn < ideas.txt
cse-srv% 
```

如果有人将 ideas.txt 变成 exam.txt 的 Symbolic Link

### Users Make Mistakes
- Social engineering, phishing attacks
    - How many of you really read the permission acquired by apps during installing?
- Good idea: threat model should not assume users are perfect

### Cost of Security
- Security VS. Availability
    - E.g., system requires frequent password changes -> users may write them down
- Good idea: cost of security mechanism should be commensurate with value

## Case：Password
### An Example: Guessing Password
```python
def checkpw(user, passwd):
    acct = accounts[user]
    for i in range(0, len(acct.pw)):
        if acct.pw[i] ≠ passwd[i]:
            return False
    return True
```

### Problem: Guess One Character at a Time
- Guess a password
    - Allocate password 1 byte away from a page boundary
    - Page-fault means first char is OK
    - Example of cross-layer interactions
- Also known as "Timing Attack"

### Problem: server has a copy of all passwords
- If adversary exploits buffer overflow, can get a copy of all passwords

### Solution: 存储密码的哈希值
- 同时解决以上两个问题

```python
def check_password(username, inputted_password):
    stored_hash = accounts_table[username]
    inputted_hash = hash(inputted_password) 
    return stored_hash == inputted_hash 
```

### Problem：Rainbow Table
- 如果 1M 的账户信息（密码的哈希值）都泄露了
- 攻击者可以逐个猜测密码，建立 common passwords 的哈希表
- 彩虹表

### Solution
- Numbers + uppercase + lowercase
- 加盐
    - hash(salting(password, random_salt_value))
    - 存储盐值和哈希值

```python
def check_password(username, inputted_password):   
    stored_hash = accounts_table[username] 
    inputted_hash = hash(inputted_password | salt) 
    return stored_hash == inputted_hash 
```

### Bootstrap Authentication
- 不要每个指令都验证密码
- Web
    - cookie
    - jwt

### Session Cookies: Strawman
- 登录时检查用户名和密码，成功后返回
    - {username, expiration, H(server_key | username | expiration)}
- Use the tuple to authenticate user for a period of time
    - 不需要存储密码，或者频繁登录
    - server_key 防止用户自行伪造 session
    - server_key 可以改变
    - 可以检查 username 和 expiration 是否正确

### Problem：同一个哈希值可能用于不同的 username & expiration pair
- E.g., "Ben" and "22-May-2012" may also be "Ben2" and "2-May-2012"
- 加个分隔符就完事了……

### Phishing Attacks 仿冒
- 假网站，让你输用户名和密码
- 之后的某些 Tech 要靠 Server + Browser 一起来验证

### Technique 1：challenge-response scheme
- Server chooses a random value R, sends it to client
- Client computes H(R + password) and sends to server
- Server checks if this matches its computation of hash with expected password
- If the server did not already know password, still does not know (to defend against phishing)

![challenge-response](./image/challenge-response.png)
- Adversary only learns H(valarMorghul1s | 458643); can not recover the password from that 

### Technique 2:Make the server prove it knows your password
- Client chooses Q, sends to server, server computes H(Q + password), replies
    - Server 如果存储的是密码的哈希值，这里应该是 H(Q+H(password))
- Only the authentic server would know your password!

- 如果 Tech1 Tech2 两种形式同时存在，也是有问题的
- Fake server 拿着 Tech1 的 R，给真 server 发 Tech2 的请求，server 返回 H(R + password) 给 Fake server。Fake server 再把 H(R + password) 给 server 发 Tech 1 的请求

### Tech 3: turn offline into online attack
- Turn phishing attacks from offline into online attacks
    - If adversary doesn't have the right image, users will know the site is fake
    - Adversary could talk to real site, fetch image for each user that logs in

- Why is it still useful, then?
    - 攻击者需要花更多的心思
    - 而且被仿冒的网站可以发现有人仿冒它
- Key insight
    - Don't need perfect security, small improvements can help

### Tech 4: Specific password
- Make passwords specific to a site.
    - Instead of sending password, send H(servername + password).
    - Just like a basic password scheme, from the server's point of view.
- Except impersonator on another server gets diff passwd.
- Recommendation
    - E.g., LastPass

### Tech 5: one-time passwords
- Design: construct a long chain of hashes.
    - Start with password and salt, as before.
    - Repeatedly apply hash, n times, to get n passwords.
    - Server stores x = H(H(H(H(...(H(salt+password)))))) = Hn(salt+password)
- To authenticate, send token=H{n-1}(salt+password).
    - Server verifies that x = H(token), then sets x <- token
    - User carries a printout of a few hashes, or uses smartphone to compute them.
- Alternative design: include time in the hash (Google's 2-step verification)
    - Server and user's smartphone share some secret string K.
    - To authenticate, smartphone computes H(K || current time).
    - User sends hash value to server, server can check a few recent time values.

### Tech 6: bind authentication and request authorization
- Session

### Tech 7：FIDO：Replace the password
![fingerprint](./image/fingerprint.png)

## Secure Channel
### Encryption Properties
- Problem: reply attacks
![replay-attack](./image/replay-attack.png)
- Problem: reflection attacks
![reflection-attacks](./image/reflection-attacks.png)

那么A、B使用不同的 key
![encryption](./image/encryption.png)

问题在于A、B如何知道对方使用的 key？

- 质数 p, g
- 有以下性质
    - 若已知 g^r mod p，几乎不可能通过 p, g 求出 r

![diffie-hellman](./image/diffie-hellman.png)

- 仍然有中间人攻击
![man-in-the-middle](./image/man-in-the-middle.png)

### RSA

使用私钥分发消息

使用公钥验证

{public_key, secret_key}

```
sign(secret_key, message) -> sig
verify(public_key, message, sig) -> yes/no
```

### 如何获得公钥
1. Alice remembers the key she used last time
    - Easy to implement
    - Effective against subsequent man-in-the-middle attacks      
    - Doesn't protect against MITM attacks the first time around
    - Doesn't allow parties to change keys   
2. Consult some authority that knows everyone's public key 
    - Does not scale (client asks for a PK for every new name)      
    - Alice needs server's public key beforehand   
3. Authority, but pre-compute responses 
    - Authority creates signed messages: {Bob, PK_bob}_{SK_as}
    - Anyone can verify the authority signed this message, given PK_as
    - When Alice wants to talk to Bob, she needs a signed message from the authority, but it doesn't matter where this message comes from as long as the signature checks out
    - i.e., Alice could retrieve the message from a different server
    - This signed message is a certificate      
    - More scalable

![TLS-handshake](./image/TLS-handshake.png)

### Questions on Certificate Authorities
- Who should run the certificate authority?
- How does the browser get this list of CAs?       
    - Generally they come with the browser     
- How does the CA build its table of names <-> public keys?      
    - Have to agree on how to name principals, and 
    - Need a mechanism to check that a key corresponds to a name    
- What if a CA makes a mistake?       
    - Need a way to revoke certificates: Expiration date?  Not useful for immediate  problems      
    - Publish certificate revocation list?  Works in theory, not as well in practice
    - Query online server to check certificate freshness?  Not a bad idea   
- Alternative: avoid CAs by using public keys as names (protocols: SPKI/SDSI)
    - Works well for names that users do not have to remember/enter
