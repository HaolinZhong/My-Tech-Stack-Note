# Introduction, History, and Internet Architecture

## Brief History of Computer Network

- Galactic Network: LickLider, MIT, DARPA, two computer, telephone line
- ARPANET: 4 nodes
- Network Control Protocal: inital ARPANET host to host protocal
- TCP/IP: function of TCP? IP?
- DNS: function of DNS?



## Internet Architecture

- layered atchitecture: advantages(3) & disadvantages(3)
- OSI model & 5 layer model
  - what's are these layers?
  - name of data in these layers?
  - common protocals of each layer
- socket: what is socket



## Layers

- Application layer: 
  - protocal(4) ?
- Presentation layer: 
  - service(1)? 
  - protocal(2)?
- Session layer: 
  - service(2)?
  - protocal(2)?
- **Transport layer** 
  - ⭐service(3)?
  - interface(1)?
  - ⭐protocal(3)?
  - example(2)?

- Network layer
  - service(4)
  - protocal(2)
  - example(1)
- Data Link layer
  - protocal(1)
  - example(3)
  - data link reliable delivery  vs  TCP reliable delivery
- Physical layer



## Layers Encapsulation

- content / function of headers of:

  - segment (2)

  - data gram (1)

    

- intermediate devices
  - layer level of...
    - router
    - switch
    - bridge
    - repeater
    - hub



## End to End Principle

- principle
- violation(2)
  - is WIFI error checking violation or not?





## The Hourglass Shape of Internet Architecture

- understand what hourglass refers to.
- what are the waist of the hourglass (3).
- ramifications of the shape (2)



## Evolutionary Architecture Model

- implications of the model
  - TCP/IP survived for avoid competetion with telephone
  - transport layer acts as shied for IP, as TCP/UDP are competitive and dependent on IP
  - new architecture will still evolve to an hourglass shape



## Learning Bridges

- behavior



## Looping Problem & Spanning Tree Algorithm

- looping problem
- spanning tree algorithm mechanisms
  - In general, spanning tree algorithm chooses certain edges from a graph to turn it into a multiple-node tree. In a tree, there is a certain root/child relationship and no loop is presented, so looping problem can be avoided.
  - A distributed algorithm that let nodes communicate and select which node should be the root, which nodes should be the child node at depth 2, ...







# Transport & Application layers



## Introduction

- why transport layer?
  - problem with network layer

- 2 main protocals
- packet name for protocals?



## Multiplexing

- reason
- implementated with ... ?
- 2 methods



## Connectionless & Connection-Oriented

- 两种方式包含的信息的区别?

  

- TCP vs UDP

  - UDP
    - UDP feature(2)
    - benefit (2)
    - UDP header content (4)
    - 补码机制
  - TCP
    - 建立连接: 三次握手
    - 结束连接: 四次挥手



## Reliability Transmission

- TCP reliability mechanism
- types of ARQ
  - stop & wait
  - go-back-n; selective acking
  - fast retransmit




## Transmission Control

- reason
- 2 part



- flow control
  - formula

- congestion control
  - goals (4)
  - flavors (2)



- signals of congestion



## How TCP limit sending rate

- congestion window, receive window formula





## AIMD

- 算法控制的对象是cwnd size
- Additive Increase
- Multiplicative Decrease
- 两种信号以及反应



## Slow Start in TCP

- slow start phase
  - 指数增长
  - 超过threshold后回到AIMD做法



## TCP fairness

- same RTT
  - AIMD leads to fairness

- unfairness
  - different RTT
  - application using multiple TCP connection



## TCP cubic

- fair regardless of RTT, depends on elapse time from 2 congestion event
- use multiplicative decrease
- use cubed polynomial to try to stay on large cnwd



## TCP throughput

- 计算模型
  - 认为网络稳定后, 图像呈锯齿状, cnwd 从w/2增长到w, 再跌到w/2
  - 一个cycle中:
    - 送出数据量: 
      - (上底 + 下底) * 高(时间轴, x轴) / 2 
      - $(\frac{W}{2} + W) \times \frac{W}{2} \div 2 = \frac{3}{8} W^2 $
    - 认为每发送$\frac{1}{p}$ packet发生一次loss.
      - 则$\frac{1}{p} = \frac{3}{8}W^2$
  - 带宽计算方式: 每个cycle 数据量 / 每个cycle时间
    - 数据量为 3/8W^2  * 单位MSS
    - 时间为 W/2 * 单位RTT	
    - 最后可带入p求得BW表达式









# Intra Domain Routing

## Introduction

- 区分routing / forwarding



## Linkstate Routing Algorithm

- 是一个全局的算法, 一次只有一个node在考虑
- 时间复杂度O(n^2)
- 本质上是dijkstra
- OSPF protocal
  - Hierachy
    - 整个系统分成了很多area, 每个area自己运行一套link state, 并在area内共享算法运行结果保持一致
    - 每个area具有自己的border routers来负责area之间的通信
    - 所有area中, 有且只有一个area会作为整个系统的backbone area, 这个area会负责area之间的routing, 并存储所有border router信息. (也可能存其他router)
    - area间通信时, packet依次经过src, src border, backbone, dest border, dest.
  - Operation
    - 单个area内, 每个router都运行dijkstra, 来获取距离信息
    - 若link state改变, 相关的router会向area内所有router broadcast. 
  - Link State Advertisement: area内, router交流时使用的消息是LSA.
  - Refresh Rate for LSA: 默认30分钟一次.



## Distance Vector Routing

- 可以看做是一个分布式的算法
- count to infinity problem
- poison reverse: z经过y到达x的话, 就告诉y, z到x是无穷大, 来避免count - to - infinity
  - 只能解决2 node的问题

- Protocal: RIP
  - 基于DV protocal但不同
  - RIP advertisement message定期在neighbor间交换





## Processing OSPF Messages in the Router

- How does a router process advertisements?
  - LSA信息进入router, 被存在队列中; 重复的LSA不存, 会立即ACK返回
  - Router CPU依次处理所有LSA, 更新Link State DB, 调度后续SPF算法的执行
  - 没有传入的LSA后, 将之前收到的消息打包成一个LSA packet并flood出去
  - 然后, 执行SPF算法, 更新FIB数据库.





## Hot Potato Routing

- HPR规定了 一个AS在收到跨AS消息后, 如何在多个 networks exit (egress points)来将消息送出
- HPR规定选择内部通信开销(IGP cost)最小的. (从多个AS的全局来看就不知道开销是小是大了)



# Intra Domain Routing

## Introduction

- 理解概念: ISP, IXP, CDN
- ISP的特征
  - 分层
  - 竞争合作: 分层本身就是一种合作, 还有同层的互相开放不收费的合作; 但是同层的会竞争来抢下层的客户.
  - 向平面化发展: 尽管分层, 但随IXP, CDN出现, 越来越向平面发展



## Autonomous System Business Relationships

- Provider - Customer

- Peering

  - 彼此的流量要差不多, 否则有一方就吃亏了

    

## BGP Routing Policies: Importing and Exporting Routes

- BGP: Border Gateway Protocol
  - IGP: AS内部, 包括OSPF, RIP

- import / export的rule 主要基于盈利的动机来设定
- 优先级: customer route (盈利) > peer route(免费) > provider route(收费)



## BGP and Design Goals

- scalability
- express routing policy
- cooperation among ASes
- later: security



## BGP Protocal Basics

- BGP session
  - 发生在一对router间 (BGP peers)
  - 通过半永久的TCP 端口连接, 互相交换routing信息
  - eBGP: 跨AS, iBGP: AS内部



- BGP messages
  - UPDATES消息: 可以是announcement, 也可以是withdraw
  - KEEPALIVE消息: 保持session不断开



- BGP Prefix Reachability
  - BGP协议规定, 目的地用IP前缀表示
  - Gateway router向外推广内部的可及IP前缀, 向内分发到外部目的地的route
  - 内部的router会扩散收到的external route



- Path Attributes and BGP Routes
  - 除了IP前缀之外, 传播的内容还包括一些BGP attrbutes
  - AS-PATH: 
    - route所经过的所有AS的编号
    - 用于防止looping
  - NEXT-HOP
    - 到目的地路上的下一个router的IP地址
    - internal router的NEXT-HOP是border router (往AS外传消息时); 可以在多个border router中存最好的





## iBGP and eBGP

- border router通过eBGP收到的update会通过iBGP传遍AS内部
- 每个border router都会和AS内其他所有活着的router建立iBGP来方便信息传递
- iBGP和IGP有显著差别. iBGP只传递外部的routing信息. IGP则负责配置AS内的routing.





## BGP Decision Process: Selecting Routes at a Router

- BGP按照一定的策略来决定import/export哪些外部的route;
- 外部route传来时
  - 筛选过后的route会存在forwarding table中
  - 按照export policy来决定route要export到哪些neighbor



## The router's decision process

- 假设router要在两条目的地相同的route中选择一条, 那么router会根据一系列attribute做选择
- 重要的attribute包括:
  - LocalPref: 对某个AS传来的route的偏好程度. 会尽量选择LocalPref高的route, 这也影响了outbound traffic, 影响了exit point的选择.
    - customer > peer > provider > backup
  - MED: 选MED较小的route; 影响inbound traffic; 
    - B有两个boader router, B传route给A时把R1 MED设得小, R2 MED设得大, 这样A就会选择R1来传信息给B



## Challenges with BGP: Scalability and Misconfigurations

- misconfiguration
  - 造成大量的update, 消耗过多的资源
  - 解决: 
    - 限制routing table size; 
    - 限制route改变次数 (flap damping)



## Peering at IXPs

- IXP是物理设施, 连接多个ASes. 提供容错, 安全等功能.

- 为什么popular
  - 大容量
  - 组织DDos Attack
  - 真实世界的研究机会
  - 市场活跃



- AS如何在IXP peer
  - AS的router在IXP的一个switch的port上注册
  - router必须支持BGP
  - AS要接受IXP的使用条款(GTC)
  - 两AS在IXP peer的费用包括
    - 建立AS到IXP间线路
    - 使用IXP端口的月费
    - 可能的年费
  - 不过, IXP对peering本身一般不收费, 也不干扰 (除非违反GTC)



- 为什么AS喜欢在IXP peer
  - keep local traffic local
  - 费用低
  - 性能好 (延迟低)
  - 盈利 (很多内容提供者都用IXP)



- IXP services
  - peering: public, private, remote, mobile
    - remote: 第三方可以承包转售IXP端口
  - Route servers and Service level agreements 
    - 免费使用router, 和其他participant快速peering
  - DDoS blackholling
  - Free value-added services: router registry, speed test, DNS server...





## Peering at IXPs: How Does a Route Server Work?

- multi-lateral BGP peering session
  - IXP route servers直接和各个AS建立BGP连接(AS之间不连接), 集中式地完成调度
    - 收集分享route信息
    - 执行自身的BGP decision并推广到peer的route server上.



## How does a route server (RS) maintain multi-lateral peering sessions?

- 一台route server
  - 对连接的每个AS都维持一个AS-specific RIB
  - 拥有和其他RS共享access的集中式的master RIB
  - 具有peer specific (例: A能否传给Z, Z能否接受A)的filter来控制route信息在AS-specific RIB和Master RIB的流动



## Remote Peering

- 如何判断remote peering的participants:
  - 老办法: RTT
    - 但现在已经不适用
  - port capacity: 
    - reseller的capacity小
  - colocation: 
    - 比较IXP与participant的物理位置
  - multi-IXP router inference: 
    - 一个AS可能连多个router. 若在一个router上判断发现它是local / remote, 就可以直接参考这个router的结果.
  - Private connectivity with multiple existing AS participants: 
    - 有这个特征说明是local peering





# Router Design and Algorithms (Part 1)

## Router Components & Functions

- basic component of routers: port, switching fabric, routing processor
  - input port功能
    - line termination
    - data link processing
    - look up forwarding table, forwarding, queuing
  - switching fabric有三类: memory, bus, crossbar
  - output port功能
    - queuing (buffer)
    - data link processing
    - line termination
- 2 main functions
  - forwarding plane functions
    - 非常快; 硬件内编码
  - control plane functions
    - 执行routing protocal, 维持routing table, 计算forwarding table
    - routing processor运行的软件执行, controller可以是远程的



## Router Architecture

- packet在router中经历几个流程:
- time sensitive:
  - lookup: 
    - 查Forwaring table, 把destination IP转换为output link
    - longest prefix matching algorithm
  - switching & scheduling:
    - 将packet从input传输到output
    - 现代的router用crossbar作为fabric, 因为比较快
    - schedule是一个难题, 因为多个input可能想走同一个output
  - queuing: 
    - packet传送到output link后排队等待送出
    - 可能简单按照FIFO排, 也可能设计得更复杂来保证 延迟和带宽公平

- less time sensitive:
  - Header validation and checksum
    - 检查version number, 减小TTL值, 计算补码
  - route processing:
    - router processor 根据routing protocal (RIP, OSPF, BGP) 来建立forwarding table
  - protocal processing
    - router需要实现几个protocal
      - simple network management protocol (SNMP): 计数便于观察
      - TCP and UDP
      - Internet control message protocol (ICMP): 传送报错信息, 如TTL已小于0



## Different Types of Switching

- memory
  - port作为IO device, 向routing process发送interrupt来将packet拷贝到内存中, 然后processor查表, 再次IO把packet送到output
- bus
  - bus是一个中间设施, 把传入的packet送到所有output port, 且一次只能送一个, 限制了速度
  - packet在input处被额外加header以指明out port, 因此只有指定out port会接受该packet

- crossbar
  - 使用2N个bus交错, 来接受N输入, 传送N输出
  - 从A传送到Y时, crossbar关闭AY的交点, 让数据能够从A到Y
  - 可以同时传输多个packet, 只要他们destination不一样



## Challenges that the router faces

- Bandwidth and Internet population scaling
  - 设备数增加, 网络活动增加, 外部网线变快, 带来了传输的压力
- Services at high speeds
  - 新的应用需要防延迟, 防攻击



- 问题和解决方案
  - lookup
    - 数量太多, 查找困难
    - Longest prefix matching
  - Service differentiation
    - 不同应用所需的连接质量不同. 
    - 需要router在classsify packet时考虑更多东西, 如src, application
  - Switching limitations
    - 需要提升switching速度
    - 使用crossbar; 但高速下, 会发生head of line blocking问题
  - Bottlenecks about services.
    - 需要保证高性能
    - 通过提供测量, 安全保障等实现



## Prefix-Match Lookups



- Prefix的表示法

  - 16bit地址 (6位数字(3 + 3)): 132.234

    - Binary form of the first octet: 10000100 + Binary of second octet: 11101010
    - Binary prefix: 1000010011101010

  - slash notation

    - Standard notation: A/L (where A=Address, L=Length); Example: 132.238.0.0/16
    - 16 denotes that only the first 16 bits are relevant for prefixing

  - masking

    - The Prefix 123.234.0.0/16 is written as 123.234.0.0 with a mask 255.255.0.0
    - The mask 255.255.0.0 denotes that only the first 16 bits are important.

    

    

- fixed length prefixes很快耗尽, 所以CIDR规定使用variable length prefixes

  

  

- challenges revolve around lookup speed, memory, and update time.
  - measurement study: 超大的瞬时的并发流量让caching的方法效果下降
  - lookup的速度非常关键, 而一大部分的耗时来自accessing memory
  - 不稳定的routing protocal 会影响routing table的update time, 甚至让操作慢个几毫秒
  - memory usage tradeoff很重要. 选择贵但快的还是便宜但慢的?



## Unibit tries

- 掌握看图方法



## Multibit tries

- unibit tries需要的memory access太多了, 32位prefix就需要32次
- stride: 
  - 每个node不止放一位, 而是放多位; stride指的是位数
  - 一个node最多可以有2^stride的child node

- prefix expansion:
  - 采用了新的stride后, 需要扩展原来的prefix到指定的位数, 如3的倍数位数
  - 此时, 对于原本5位的prefix, 可以穷举产生2个新的6位的prefix
  - 有时会出现collision, 那么重复的就被移除了



## fixed stride & variable stride

- 掌握看图原理
- variable stride
  - 每个node的stride可以不同
  - 目的是节约内存, 并减少memory access
  - 最佳的stride值是通过dp算法决定的





# Router Design and Algorithms (Part 2)



## Why We Need Packet Classification?

- packet classification概念
  - handle packets on multiple criteria like TCP flags and source address
- why
  - keep up with quality of service and security.



- 已经建立的packet classification
  - firewalls, 过滤 
  - Resource reservation protocols (DiffServ),  保留带宽
  - Routing based on traffic type, 避免延迟





## Packet Classification: Simple Solutions

- Linear Search: 防火墙就用这个, 因为规则不多
- Caching
  - 仍会存在未缓存的东西
  - 即使命中缓存, 若采用linear search, 依然很慢
- passing labels
  - The Multiprotocol Label Switching (MPLS) and DiffServ use this technology.
  - 在sender和receiver添加 / 解读header (label), 中间的router无需重新分类, 直接按照label代表的特定规则执行



## Fast Searching Using Set-Pruning Tries

- 可能既需要根据dest, 也需要根据src来选择rule
- set pruning tries:
  - 上层是dest tries, leaf node连接到 src tries
  - 问题: memory explosion, 因为一个rules可能出现在多个dest tries对应的leaf node之后, 这就要求在每个leaf node后面重复的生成一些src tries (他们还不一样, 因为要整合不同的src prefix)



## Reducing Memory Using Backtracking

- dest tries的leaf node只存储精准命中的rules所对应的src tries
- 执行时, 首先根据dest prefix走到dest tries最深处, 然后搜寻src tries看是否能匹配成功, 若失败则backtrack, 搜寻上一个的dest node指向的src tries

- 解决了内存问题, 但时间消耗太大



## Grid of Tries

- 使用precomputation
  - 当在某src tries中不能找到rule时, precompute 一个switch pointer来直接指向另一个可能含有rule的src trie





## Scheduling and Head of Line Blocking

- schedule的情景
  - N in, N out, N^2 cross points
  - cross point 开/关
  - input link 在某一时刻最多和 1个 output link相连
  - 希望有尽可能多的input output pair能够并行传输



- take the ticket algorithm
  - 对于所有的input line, 每个output line维持一个分布式的队列 (实际存储位置是在input line)
  - input line向传输时, 先向output请求ticket, 之后会收到queue number
  - outline 收到多个请求时, 按顺序进行处理. input观察到自身queue number serve上之后, 就开始传输.
  - 请求同步发出, 传输按序执行; 完成传输后才能发起下一个请求



## Avoiding Head of Line Blocking

### Avoiding head-of-line blocking via output queuing:

- 不希望发送数据时还要在input端排队. 希望直接发送到out端, 直接在out端排队
- 当fabric传输速度是input的N倍时, 可以满足 (N: input line数量)
  - 即使所有(N个) input同时要求传输, 因为是N倍速度, 就可以在单位时间内完成传输



- 实践: knock out scheme
  - 实际情况中一般只有k个input同时要求传输, 这就只需要我们以k倍速度传输即可
  - 但有时, 我们以k倍速传输时, 还是可能出现要求传输的input量 (N)大于k的情况
    - k = 1 N = 2: concentrator 二选一
    - k = 1 N > 2: 2 by 2 concentrator, 多选一
    - N中选k个: k knock out trees; 太麻烦



### Avoiding head-of-line blocking by using parallel iterative matching:

- 仍然允许input端的排队
- 对于在input端排队的packet, 不仅只为head发起请求, 也为其他output不同的packet发起请求 (并发请求)
- output收到多个请求时, 只在多个请求中随机grant一个, 丢弃剩下的请求
- input收到多个grant时, 只随机选择其中一个进行传输
- 这样, 即使某个packet的output阻塞, 其他packet没阻塞还可以传递, 通过并行节约了时间



## Scheduling Introduction

- 这里研究, output link上排队的packet怎么发送的问题

### FIFO with tail drop

- 按照FIFO顺序发送. 若队列已满, 则不接受packet, 直接丢弃.



### Need for Quality of Service (QoS)

- 其他调度方法, 如priority, round robin, 可提供 quality of service.
  - guarantees to a flow of packets on measures such as delay and bandwidth
  - packet流: 同起点, 同终点, 同路径, 同服务
    - flows must be identifiable using fields in the packet headers.



- why quality of service
  - 帮助处理congestion
  - Fair sharing of links among competing flows.  (重要的flow可能被drop)
  - guarantee certain amounts of bandwidths to a flow / guarantee the delay through a router for a flow. 





## Deficit Round Robin

- 一种让scheduler强制保留带宽的方法
- 机制
  - 各flow轮流发送
  - 因为packet size大小不同, 单纯轮流还是导致不公平 -> Bit-by-bit Round Robin



### Bit-by-bit Round Robin

- the rate of increase in round number: `dR / dt = µ / N`
  - R: round, μ: bit速率, N: flow数量

- 对于一个新来的packet, 它达到队列顶部时的round数量是: `S(i) = max(R(t), F(i-1))`
  - `F(i-1)`: 同flow中前一个packet的传输完成时的round数
  - `F(i) = S(i) + p(i)`: p(i)是packet size.



### Packet-level Fair Queuing

- The packet which had been starved the most while sending out the previous packet from any queue, is chosen.

- 即, 在多个flow队列的头部的packet中, 选择当前F(i)最小的来运输

- 问题:

  - 需要记录F(i)

  - 为了比较大小, 还需要建立priority queue, 复杂度log k级别, k指flow数量

  - 建立新queue时所有的timestamp都要变化, O(n)操作, 难以接受

    

### Deficit Round Robin (DRR)

- 设置
  - 对于每个packet queue, 有
    - a quantum size, Qi, 分配的基础带宽 
    - a deficit counter, Di, 一个动态的带宽
  - 每轮会逐个发送packet queue中数据. 对一个queue, 会一直发送packet, 直到 `sent + next packets > Qi + Di`. 然后, `Di = Qi + Di - sent`, 没用完的存在Di里, 供下一轮使用. (如果这时queue里的packet以及全部发送完, 就直接把Di设置为0)



## Traffic Scheduling: Token Bucket

- 有时, 我们不想像DRR一样, 把每个flow用不同的queue分开. 这样我们就需要不同的算法.
- 令牌桶 shaper
  - 可以: 限制平均带宽 / 限制最大带宽
  - 设置
    - 令牌桶中以速率R添加令牌, 且最大令牌量为B
    - packet通过时, 若令牌量 大于 packet size, 则可正常通过, 否则就要等待令牌量增加
      - 通过后, 扣除相应令牌量
      - 这样, 最大速率就限制在了B, 平均速率限制在了R
    - 通过给每个flow设置一个counter即可实现
  - 这个算法还是multiple queue的, 因为混在一个queue时, 根据算法, 可能会因为某一个flow令牌不满而堵塞全局



- 令牌桶policing
  - packet到达队列头部时, 一旦发现令牌桶为空, 就直接drop掉, 这样就不会堵塞



- 对比
  - policer在traffic rate超过设置的上限时会直接drop超出的部分, 因此锯齿形图像被"斩首".
  - shaper则会将多余的packet额外放在一个queue或者buffer里, 之后再调度, 所以超出的部分会被延后. 因此锯齿形图像被拉平.



## Leaky Bucket

- 漏桶算法也可以实现policing和shaping, 和令牌桶是不同的思路
- 漏桶, 即一个buffer, 
  - 以恒定速率将其内的packet送出
  - 接收host传来的packet
    - 若传来的packet太大超出buffer size, 直接丢弃
    - 否则, 放进buffer 中, 等待送出