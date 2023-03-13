#计算机网络重点整理

## Chapter1——概述

### 因特网概述

- 网络network：许多计算机连接在一起
- 互联网internet：许多网络连接在一起
- 因特网Internet：全球最大的互联网

### 发展三阶段：

1. ARPANET to internet  
1969 分组交换  
1975 互联网  
1983 TCP/IP 互联网起源 美国军方研究  
2. 三级结构的Internet  
	1985 局域网 地区网 主干网 美国政府支持
	多层次的ISP(internet service provider)  
	1993 正式民营  
3. 因特网的标准化：因特网协会ISOC

### 因特网的组成
- 边缘部分：由主机与伺服器构成，实现主机间的通信
- 核心部分：由路由器与网络构成，实现数据的交换
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-0.png)

### 报文交换方式

- 电路交换：建立专线连接，适合数据量大的实时传输
- 报文交换：直接发送一个个报文，时延比分组交换长
- 分组交换：将报文分成多段添加首部之后分别传输，线路复用。路由器有存储转发功能
	优点：高效、灵活、迅速、可靠
	缺点：时延、开销
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-1.png)

### 计算机网络的类别
1. 依作用范围：
	- 广域网WAN：花钱向ISP买服务、买带宽  
	- 城域网MAN  
	- 局域网LAN：自己购买设备自己维护，带宽固定 (100M、1000M)，距离100米以内  
	- 个人区域网PAN
2. 依使用者划分：
	- 公用网Public Network:Internet
	- 专用网Private Network:内网
3. 依拓扑结构：
	- 总线型
	- 环形
	- 星型
	- 树型
	- 网状结构
4. 依交换方式：
	- 电路交换网  
	- 报文交换网  
	- 分组交换网
5. 依工作方式：
	- 资源子网  
	- 通信子网  
	- 接入网  

### 计算机网络的性能指标
1. 速率(data rate/bit rate)：主机在数字信道上传输数据位的效率
	 - 单位：b/s(bps), kb/s, Mb/s, Gb/s
2. 带宽(bandwidth)：数字信道所能达到的最高速率
	- 单位：b/s, kb/s, Mb/s, Gb/s
3. 吞吐量(throughput)：单位时间内通过某个网络的数据量
	- 单位：b/s, Mb/s, etc.
4. 时延：
	- 发送时延=数据块长度(bit)/信道带宽(bit/s)
	- 传播时延=信道长度(meter)/信号在信道传播速率(meter/s)
	- 处理时延=处理分组，查找适当路由的时延
	- 排队时延=在路由中等待转发的时延（取决于网络当时通信量）
	- 总时延=发送时延+传播时延+处理时延+排队时延
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-2.png)
	- 时延带宽积(表示链路能够容纳的最高比特)=传播时延*带宽
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-3.png)
5. 往返时间(Round-Trip Time):双向交互一次所需要的时间
6. 利用率：
	- 信道利用率：某信道有百分之几的时间是有被利用的（有数据通过）
	- 信道利用率=有数据通过时间/全部时间
	- 网络利用率：全网信道利用率的加权平均值
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-4.png)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-5.png)
7. 非性能指标：  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-6.png)

### 计算机网络体系结构
1. OSI/RM(Open Systems Interconnection Reference Model)七层模型：互联网法律上的国际标准
	- 应用层(Application layer):能够产生网络流量、能够与用户交互的程序
	- 表示层(Presentation layer):加密、压缩、面向开发人员
	- 会话层(Session layer)：服务和客户端建立的会话、查木马   
	----------以上为开发人员主要业务范围----------  
	- 传输层(Transport layer)：可靠传输（建立会话）、不可靠传输（不建立会话）、流量控制
	- 网络层(Network layer)：IP地址编址、选择最佳路径
	- 数据链路层(Data link layer)：数据如何封装、添加物理层地址(MAC地址)  
	----------以上为网络工程师主要业务范围----------  
	- 物理层(Physical layer)：规定电压、接口标准
2. 网络排错：从底层到高层逐个排错
3. OSI模型的网络安全：
	- 物理层安全：交换机/路由器接口管理
	- 数据链路层安全：ADSL密码、无线AP(access point)密码
	- 网络层安全：路由器限制IP网段
	- 应用层安全：SQL注入漏洞、上传漏洞
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-7.png)
4. 开放系统信息交换概念：
	- 实体(entity):交换信息的硬件或软件进程
	- 协议(protocol):两个对等实体的通信规则
	- 服务(service):下层向上层提供服务，上层通过下层的服务实现本层共能
	- 服务访问点(Servise Access Point):相邻两层实体间交换信息的地方  
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-8.png)
5. 五层协议的数据单元：
	- 应用层：传输数据单元PDU（ex.文档）
	- 运输层：报文（分段）
	- 网络层：IP数据包（给每个段加上IP地址）
	- 数据链路层：数据帧（加上物理层地址）
	- 物理层：bit（加上帧头帧尾）
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-9.png)

---

## Chapter2——物理层

### 物理层的基本概念  
**任务描述**：解决如何在连接各种计算机的传输媒体上传输数据比特流的问题
**主要任务**：确定与传输媒体的接口的特性，如：

- 机械特性：接口形状、大小、引线数
- 电气特性：电压范围
- 功能特性：0与1的电压表示
- 过程特性：建立连接时各个部门的工作步骤


### 典型数据通信模型
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10.png)
### 相关术语

- 通信的目的是传送**消息**
- 数据(data)：运送消息的实体
- 信号(signal)：数据的电气或电磁表现
- 模拟信号：代表消息参数的取值是**连续的**
- 数字信号：代表消息参数的取值是**离散的**
- 码元(code)：代表不同离散数值的基本波形
- 信道(channel)：向一个方向传送信息的媒体
- 单向通信（单工通信）：只能有一个方向的通信而没有反方向的交互
- 双向交替通信（半双工通信）：通信双方都能发送/接收，但不能同时发送/接收
- 双向同时通信（全双工通信）：双方可以同时发送/接受消息
- 基带信号(baseband)：来自信源的信号（原始的信号）
- 带通信号(band pass)：把基带信号经过载波调制（编码）后，把基带信号的频率范围搬移到较高频段并转换为模拟信号
- 调制方法：调幅(AM)、调频(FM)、调相(PM)
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-11.png)

### 常见编码方式
- 不归零制：正电平为1、负电平为0
- 归零制：正脉冲为1、负脉冲为0
- 曼彻斯特编码：位周期中心向上跳为0、向下跳为1
- 差分曼彻斯特编码：位周期中心始终有跳变，位边界有跳变为0、无跳变为1（抗干扰性较强）
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-12.png)

### 信道的极限容量
1. 奈式准则：在任何信道中码元传输速率是有上线的，否则会出现码间串扰问题  
	>理想低通信道最高码元传输速率=2WBaud
	>W为带宽，单位Hz
	>Baud为波特，码元传输速率的单位（1码元有1bit信息量则1波特=1bit/s）
	
2. 信噪比：信号平均功率和噪声平均功率之比
	>信噪比(dB)=10log(S/N)  (dB)
	
4. 香浓公式：有噪信道的极限传输速率S
 $W \times \log_2 (1+ \frac{S}{N})$(b/s)

### 信道复用技术(multiplexing)
1. 频分复用FDM(Frequency Division Multiplexing):每个用户分配一定的频带并一直占用
2. 时分复用TDM(Time Division Multiplexing):所有用户在不同的时间占用同样的频带宽度，每个用户占用的时隙周期性出现  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(1).png)
3. 统计时分复用STDM(Statistic TDM):为每个用户添加地址信息并不固定分配时隙给不同用户
4. 波分复用WDM(Wavelength Division Multiplexing)：光的频分复用
5. 码分复用CDM(Code Division Multiplexing):通过码片编解码  

---

## Chapter3——数据链路层 
### 数据链路层的信道类型
- 点对点信道：一对一的通信方式
- 广播信道：一对多的广播通信、较复杂、使用共享信道协议来协调主机的数据发送
### 链路与数据链路
- 链路(link)：一条点到点的物理线路段，中间没有其他的点、链路是通路的一个组成部分
- 数据链路(data link)：物理线路+通信协议
- 一般适配器（网卡）包含了数据链路层与物理层
### 三个基本问题	
- 封装成帧(framing)：在IP数据报的前后分别添加首部与尾部构成一个帧。  

>帧=帧开始符SOH+控制信息（位于首部与尾部）+数据部分+帧结束符EOT

![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(2).png)

- 透明传输：如果数据部分有SOH或EOT必须使用字节填充插入转义字符   
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(3).png)
- 差错控制：传输过程可能出现的比特差错（0变成1）

>误码率BER(Bit Error Rate)：传输错误的比特占总比特的比率

>循环冗余检验CRC：原数据（被除数）加上n位0，使用固定n+1位除数P求得余数R，将R替换n位0传输，接收方使用P除数据若商为0则接受，否则丢弃  

![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(4).png)

### 点对点协议PPP(Peer to Peer Protocol)  
1. 满足的要求：
	- 简单
	- 封装成帧
	- 透明性
	- 支持多种网络协议
	- 支持多种类型链路
	- 差错检测
	- 检测连接状态
	- 最大传送单元
	- 网络层地址协商数据压缩协商
2. PPP协议的组成：
	- 最底层：高级数据链路控制协议(HDLC)
	- 中间层：链路控制协议(LCP)
	- 上层：网络控制协议(NCP)
3. PPP协议帧格式  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(5).png)  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(6).png)  
	- 字节填充（如果信息字段出现0x7E）：  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(7).png)
	- 零比特填充（当传送比特位时）：  
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(8).png)
4. PPP 协议的工作状态  
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(9).png)
	- LCP(link control protocol)：链路控制
	- NCP(network control protoco)：网络控制

### 广播信道
1. 局域网的拓扑
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(10).png)
2. 局域网主要优点与特点：
	- 具有广播功能，便于访问全网，局域网上 的主机可共享硬件与软件
	- 便于扩展，灵活
	- 提高系统的可靠性、可用性，生存性
3. 共享通信媒体
	- 静态划分
		- 频分复用
		- 时分复用
		- 波分复用
		- 码分复用
	- 动态媒体接入控制（多点接入）
		- 随机接入（主要为以太网）：所有用户随机发送消息，可能产生碰撞
		- 受控接入（已停用）：服从一定控制的发送消息，如轮询
4. 以太网协议：
	- CSMA/CD(Carrier Sense Multiple Access with Collision Detection)载波监听多点接入/碰撞检测  
		- 载波监听：每个站在发送数据之前要先检测是否有其他计算机在发送数据，如果有则暂时不要发送数据
		- 多点接入：许多计算机以多点接入的方式连接在总线上
		- 碰撞检测：计算机边发送数据边检测信道信号电压，如果电压摆动只超过阀值（信号叠加）则表示产生碰撞（冲突）
		- 半双工通信
	- 截断二进制指数退避算法(truncated binary exponential backoff)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(11).png)
### 以太局域网（以太网）
1. 以太网的两个标准：DIX Ethernet V2、802.3(IEEE)
2. 以太网与数据链路层的两个字层
	- 逻辑链路控制LLC(Logical Link Control)
	- 媒体接入控制MAC(Medium Access Control)
	- 局域网只有MAC协议
3. 以太网的服务：不可靠的（最大努力交付）
4. 以太网的信道利用率
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(12).png)
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(13).png)
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(14).png)

### 以太网的MAC层
1. MAC地址
	- MAC地址：aka.物理地址、硬件地址
	- 6个字节（48位）
	- 由IEEE的注册管理机构RA(Registration Authority)管理
	- 前三位为组织唯一标识符OUI(Organizationally Unique Identifier)
	- 后三字节由厂家指派，为扩展标识符(extended identifier)
	- MAC地址被固化在适配器的ROM中
	- 适配器有过滤功能过滤MAC帧中的目的地址
2. MAC帧的种类
	- 单播(unicast)帧（一对一）：帧的目的地址与本机相同
	- 广播(broadcast)帧（一对全体）：发送给局域网的全部地址（全1:FFFFFF）
	- 多播(multicast)帧（一对多）：发送给局域网一部分站点的帧3. MAC帧格式
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(15).png)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(16).png)

### 扩展以太网
1. 在物理层上扩展
	- 利用光纤低时延扩展主机与集线器的距离
	- 利用多个集线器组成多级星型结构以太网
		- 优点：扩大以太网物理范围
		- 缺点：扩大了碰撞域、只能连接相同数据率的集线器
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(17).png)
2. 在数据链路层扩展以太网
	- 网桥(Bridge)：根据帧的目的地址决定转发的接口，而不是在所有接口转发帧
		- 优点：有自学习算法、过滤通信量、扩大物理范围、提高可靠性、可互连不同物理层，MAC子层，不同速率的局域网
		- 缺点：存储转发增加了时延、没有流量控制功能、具有不同MAC子层的网段桥接时时延会增大、适合在用户数不多与通信量不大的局域网，否则会因广播过多而造成拥塞
	- 以太网交换机
		- 可以看作多接口的网桥
		- 接口直接连接主机或者其他交换机
		- 全双工方式工作
		- 支持多对初级同时通信，独占通道，无碰撞
		- 接口有存储器进行忙时的帧缓存
		- 帧交换表使用自学习算法自动建立
		- 有多种速率接口
		- 支持直通(cut-through)传输
		- 生成树算法：避免交换机无限循环广播
			![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(18).png)
			1. 确定根(RootBridge)
			2. 确定根端口RP：距离根最近的端口
			3. 确定指定端口DP：一条通路中距离根最近的端口
			4. 阻断非指定端口NP：通路中非DP非RP的端口
3. 虚拟局域网VLAN(Virtual LAN)
	- VLAN的概念
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(19).png)
	- 802.3ac标准支援VLAN的以太网帧
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(20).png)  
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(21).png)  

### 高速以太网
1. 100Base-T(Fast Ethernet)
	- 全双工工作无冲突
	- 不使用CSMA/CD协议但帧格式仍然是802.3标准
	- 保持最短帧长不变，规定最大电缆长度为100m
	- 帧间时间间隔为0.96$\mu$s
	- 自适应10Mbit/s和100Mbit/s
2. 吉比特以太网
	- 允许在1Gbit/s以全双工与半双工工作
	- 使用IEEE802.3协议的帧格式
	- 半双工方式使用CSMA/CD协议、全双工不需要
	- 与10BASE-T和100BASE-T向后兼容
	- 物理层标准支持光纤与铜缆

### 数据链路层安全
1. MAC地址绑定：将交换机一个接口绑定一台mac地址
2. MAC/IP绑定：限制一条链路的机器数量

---
## Chapter4——网络层

### 网络层提供的两种服务
1. 虚电路服务（电信网）
	- 建立独占连接
	- 预留通信资源
	- 可靠传输（保证发送成功）
	- 所有分组沿着同一条链路传输
	- 通信结束释放虚电路
2. 数据报服务（因特网）
	- 简单灵活
	- 无连接
	- 尽最大努力交付
	- 所有分组可能沿着不同链路
	- 网络层不提供可靠交付（由端系统负责）
	- 优点
		- 网络造价降低
		- 运行方式灵活
		- 适应多种应用

![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(22).png)

### 网际协议IP
1. 虚拟互联网
	-  网络互连的设备（中间设备/中继系统）
		- 物理层：转发器、hub
		- 数据链路层：网桥(bridge）
		- 网络层：**路由器(Router)**
		- 网络层以上：网关(gateway)*少用*
	- 虚拟互联网：利用IP协议将性能不同的互联网络在**网络层**上看起来好像一个统一的网络*(aka. 互联网)*
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(23).png)
		- 优点：可以忽略具体的网络异构细节

2. IP协议概览
	- 与IP协议配套使用的四个协议
		- 地址解析协议ARP(Address Resolution Protocol)
		- 逆地址解析协议RARP(Reverse Address Resolution Protocol)**淘汰了**
		- 网际控制报文协议ICMP(Internet Control Message Protocol)
		- 网际组管理协议IGMP(Internet Group Management Protocol)  

	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(24).png)

	- IP地址
		- IPv4地址是*全球唯一*的*32位*标识符
		- IP地址={<网络号>,<主机号>}
		- 主机号不可全0、全1
			- 主机号全0:表示该网段
			- 主机号全1:表示广播地址
		- IP地址的分类
		- ---A类----**128**----B类----**192**----C类---

	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(26).png)

	- 特殊的几个地址
		- 127.0.0.1 本地环回地址
		- 169.254.0.0 Windows系统自动分配的地址
		- 10.0.0.0 A类保留私网地址
		- 172.16.0.0——172.31.0.0 B类保留私网地址
		- 192.168.0.0——192.168.255.0 C类保留私网地址
	- 子网掩码
		- 作用：帮助判断要通信的设备是否为一个子网
		- 原理：利用子网掩码与地址**按位与**运算求网络号，若网络号相同则为同一子网
			- 若不在同一网段则使用网关的MAC地址封装数据帧转发给路由器
			- 若在同一子网则直接发送
		- A类地址子网掩码：255.0.0.0
		- B类地址子网掩码：255.255.0.0
		- C类地址子网掩码：255.255.255.0
	- 等长子网划分
		- 可以通过更改子网掩码达到划分子网的目的：
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(27).png)
		- 划分的子网是等长的
		- 点到点的子网掩码是255.255.255.**252**
	- 变长子网划分(VLSM:Variable Length Subnet Mask)
		- 利用不同的子网掩码，来划分不等长的子网
	- 构造超网(CIDR:Classless Inter-Domain Routing)
		- 无分类域间路由选择
		- 消除ABC类地址与子网划分概念
		- IP地址={<网络前缀>,<主机号>}
		- **CIDR记法**：IP地址+“/xx”  

			> xx表示子网掩码占用位数 
			> 如：192.168.0.12/24表示子网掩码为255.255.255.0
		
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(28).png)
		
		- 将要合并的子网除以4根据余数可以判断要移几位
	- MAC地址与IP地址差异
		- MAC地址决定下一跳的路由
		- IP地址决定讯息传送的终点

3. ARP协议(Address Resolution Protocol)
	- 主要功能
		- 从在网络层使用的**IP地址**中解析出在数据链路层使用的**MAC地址**
	- 工作步骤：
		1. 当主机A要往*本局域网*的主机B发送数据报时，先在ARP高速缓存(cache)中查找主机B的IP地址
		2. 若*有*，则往MAC帧中写入对应的物理地址
		3. 若*无*，则使用ARP进程**广播**发送ARP请求分组
		4. 本局域网的任何主机都能接受广播
		5. 若主机B的IP地址与请求分组地址一致，则返回*写有自己物理地址的响应分组*
		6. 当主机A接受此响应分组后，将B的物理地址写入ARP cache中
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(29).png)
	- **ARP欺骗**
		- 若在*步骤5*有危险主机C也发送了自己的响应分组，则可以截断通信
		- 主机C接收了主机A的通信，再转给主机B，就可以监听通信
	- RARP
		- 通过已知的MAC物理地址找出IP地址，已经停用
4. IP数据报的格式
	- IP数据报由**首部**与**数据**两部分组成
	- 首部分为固定部分与可变部分，固定部分共20字节，**必须具有**
	- 数据部分长度可变
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(30).png)
	- 首部介绍
		- 版本：分辨IPv4或IPv6
		- 首部长度：描述可变部分+固定部分（20字节）长度，长度从5(0101)到15(1111)，单位为**4个字节**。数据不到4个字节则使用*字节填充*
		- 区分服务：用以区分数据报的服务，如：紧急程度。少用
		- 总长度：首部与数据之和，单位是字节。
			- 数据链路层规定了**最大传输单元**MTU(Maximum Transfer Unit)
			- 若超过则需要**分片**
			![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(31).png)
		- 标识(id)：计数器，每产生一个数据报就+1。并不是序号，因为IP没有顺序。
		- 标志(flag)：占3位，只有前2位由意义。
			- 最低位是MF(More Fragment)=1表示后面还有*分片*、MF=0表示最后一个分片。
			- 中间位是DF(Don't Fragment)=0表示允许分片
		- 片偏移：表示分片的分组在原分组的相对位置。单位：8个字节。
		- 生存时间TTL(Time To Live)：数据报在网络中的生存时间，防止永远到达不了的数据报占用资源。路由器在每次转发数据报的时候就-1。TTL=1表示只能在本局域网传送。
		- 协议：指明上层的协议，以便目的主机知道要使用哪个协议处理数据
		- 首部检验和：验证**首部**有没有出错

5. IP转发分组的流程
	- 数据路由：数据经过**路由器**在不同网段转发数据报
	- 网络畅通的条件：数据报必须**能去能回**
	- 数据路由的条件：沿途的路由器必须知道**目标/源**网络下一跳给哪个接口
	- ***静态路由***：手动配置每个路由器接口的路由表
	- 网关：网络的关口。不仅具有路由功能，也能在两个不同协议集之间转换。**实现两个网络的互联**
	- 默认网关：如果主机找不到可用网关，则统一把数据报发给默认网关（没有网关不能连接外网）aka.默认路由
	- 网络负载均衡：主机A到主机B有两条路由线路，可以通过配置实现负载均衡

### 网际控制报文协议ICMP
1. ICMP协议特点
	- 目的：为了提高IP数据报交付成功的机会
	- 允许主机或路由器报告差错情况与提供有关异常情况的报告
	- ICMP不是高层协议，而是IP层协议
	- ICMP作为IP数据报的数据，加上数据报的首部，组成IP数据报发送出去
2. ICMP报文格式
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(32).png)
	- ICMP报文有两种类型，**ICMP差错报告报文**与**ICMP询问报文**
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(33).png)  
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(34).png)  
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(35).png)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(36).png)
	- PING使用了ICMP回送请求与回送回答报文
	- PING是应用层直接使用网络层ICMP的例子，没有通过运输层的TCP或UDP

### 路由选择协议（动态路由协议）
1. 内部网关协议IGP(Interior Gateway Protocol):一个自治系统内用的路由选择协议
	- 路由信息协议**RIP**(Routing Information Protocol)
		- 最早的IGP
		- 根据跳数(hop count,每经过一个路由器就+1)来选择最短路由
		- 允许最高跳数为15，因此只适合小型互联网
		- 仅与相邻路由交换信息，不相邻则不交换
		- 交换信息的内容是自己的路由表
		- 周期性广播交换信息
	- 开放最短路径优先协议**OSPF**(Open Shortest Path First)
		- 在internet ISP使用的协议
		- 直接用IP数据报传输
		- Open表示OSPF不受单一厂商控制，而是公开发表的
		- 根据**带宽**来选择路由
		- 使用**触发式更新**：只有路由链路发生改变才会更新
		- 路由表的构造
			1. 发送hello包构造**邻居表**
			2. 使用洪泛法(flooding)交换各自的邻居表构造**链路状态表**
			3. 对链路状态表使用Dijkstra的SPF最短路径算法构造**路由表**
		- 支持多区域，链路状态表只在区域内传播

2. 外部网关协议EGP(External Gateway protocol):传输不同自治系统间的路由选择信息协议
	- 边界网关协议BGP(Border Gateway Protocol)
		- 功能：用于在不同的**自治系统AS**中选择路由
		- 配置BGP时，每个自治系统要选择一个路由器作为**BGP发言人**
		- BGP发言人通过TCP连接交换路由信息
		- BGP发言人除了运行BGP协议外，还要运行AS内部协议，如RIP或OSPF
		- 优点：交换信息数量少、发言人少，路由选择简单、支持CIDR
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(37).png)

### 虚拟专用网VPN
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(38).png)

### 网络地址转换NAT
1. 传统网络地址转换
	- 优点：简单
	- 缺点：需要很多公网地址，否则NAT路由接受回传包不知道要给哪个内网地址
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(39).png)
2. 网络地址与端口转换NAPT
	- 优点：为每个内网地址配置一个端口，可以节省公网地址
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(40).png)

## Chapter5——传输层
### 传输层两个协议应用场景
1. TCP：报文有分段、编号、流量控制、建立会话（开销大）、可靠传输
	- 传输协议数据单元是**TCP报文段segment**
	- 场景：传输文件，大档案，访问网站等
2. UDP：一个数据报完成通信、不建立会话、多播、不可靠传输
	- 传送协议数据单元是**UDP报文**或**用户数据报**
	- 场景：传消息，直播等


### 传输层与应用层关系
1. http=TCP+80端口
2. https=TCP+443
3. ftp=TCP+21
4. SMTP=TCP+25
5. POP3=TCP+110
6. RDP（远端）=TCP+3389
7. 共享文件夹=TCP+445
8. SQL=TCP+1433
9. DNS=UDP+53 || TCP+53

>服务运行后在TCP或UDP的**某个端口**监听客户端请求
>端口代表服务
>可以通过更改端口号提升安全性
>**TCPIP筛选**可以限制端口号从而限制服务

### Windows防火墙的作用
1. 拦截某个服务的端口号，限制外部访问
2. 允许防火墙内部的数据发送与接受，但不允许防火墙外的数据进入
3. 如果A能ping通B但Bping不通A，有可能是A有防火墙
4. 防火墙不能防止木马程序（查会话查木马）

### 用户数据报服务UDP
1. UDP是无连接的，不建立会话，因此减少了发送前的时延并较少开销
2. UDP尽最大努力交付，不保证可靠交付
3. UDP是面向报文的，对于应用层的报文，只添加首部后就交付IP层
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(41).png)
4. 没有拥塞控制，网络拥塞不会导致主机降低发送速率，适合实时应用（直播）
5. 支持一对一、一对多、多对一、多对多的交互通信
6. 首部开销小，只有8字节
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(42).png)

### 传输控制协议TCP
1. TCP是面向连接的传输层协议
2. 每一条TCP连接只能有两个端点(endpoint)，只能是点对点（一对一）的连接
3. TCP连接的端点叫**套接字(socket)=IP地址+端口号**
4. 提供可靠交付的服务
5. 提供全双工通信
6. 面向字节流
7. 实现拥塞控制

### TCP可靠传输原理
1. 停止等待协议
	- 发送完一个分组之后停止发送并等待对方确认
	- 自动重传请求ARQ(Automatic Repeat reQuest)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(43).png)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(44).png)
	- 缺点：信道利用率太低$U=\frac{T_D}{T_D+RTT+T_A}$
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(45).png)
2. 连续ARQ协议
	- ARQ协议的目的：实现流水线传输
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(46).png)
	- 滑动窗口：
		1. 发送方可以连续发送滑动窗口内的数据
		2. 只有当接收方确认之后，窗口才能向前滑动
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(47).png)
		3. **累计确认**：接收方对于按序到达的最后一个分组发送确认（表示这个序号之前的分组都收到了
	- 缺点：当网络质量不好的时候连续ARQ会有负面影响

###  TCP报文段的首部格式
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(48).png)
- 原端口&目的端口：与UDP一样
- 序号：当前发送数据包第一个数据的编号（每一个字节按顺序编号）
- 确认号：接收方收到了数据之后向发送方发送的**渴望接收的下一个报文段的第一个数据序号**
- 数据偏移：指出TCP报文段首部的长度（最长60byte）
- 保留：没用，置0
- 紧急URG：当URG=1时，为紧急数据，优先传送
- 确认 ACK：只有当ACK=1时，确认号才有用，建立连接后ACK必须是1
- 推送PSH：当PSH=1时，发送方不等缓存满就马上推送数据包出去
- 复位RST：当RST=1，释放连接后重新链接，还用于拒绝非法报文或拒绝打开链接
- 同步SYN：SYN=1表示连接请求或连接接受报文，连接请求时SYN=1、ACK=0
- 终止FIN：FIN=1表示数据已发送完毕并要求释放连接
- 窗口：指定己方的接收窗口，也就是接收缓冲区大小，用以告知下一个报文的发送方自己只能接受缓冲区大小的数据
- 校验和：与UDP相同1，
- 紧急指针：仅在URG为1的时候才会使用，指向紧急数据

###  TCP可靠传输的实现
1. 以字节为单位的滑动窗口
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(49).png)
	- 后沿：后沿的后面部分表示已发送且已收到了确认，这些消息不需要再保留了，后沿的变化一般有两种
		1. 不动：没有收到新的确认
		2. 前移：收到了新的确认
		3. 后沿不可能向后移动，因为已经收到的确认是不可能撤销的
	- 前沿：前沿前面的窗口不允许发送，因为接受缓存大小受到了限制，前沿的变化有三种：
		1. 不断向前移动：不断发出消息
		2. 不动：没有收到新的确认、收到确认而且对方同时通知窗口缩小
		3. 向后移动：对方通知窗口缩小（可能产生错误）
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(50).png)
	- 描述发送窗口的状态需要三个指针：P1(后沿),P2(允许发送但尚未发送),P3(前沿)
		1. P1之前代表已发送并且已收到确认的部分
		2. P3之后代表还不允许发送的部分
		3. P3-P1代表A发送窗口的大小
		4. P2-P1代表已发送但未确认的部分
		5. P3-P2表示允许发送但还没发送的子节（可用窗口、有效窗口）
	- B只能对按序收到的最高序号发出确认
	- 没有按序到达的数据（提前到达的数据）会先暂存在接受窗口中等待前面的收到再一起发送确认
	- P2与P3重合的时候表示A的发送窗口已满，必须停止发送直到B传送了新的确认
	- 如果A没能接收到B的确认，会在一段时间后重传数据
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(51).png)
	- 缓存有限且是循环使用的
	- 发送缓存用来存放：
		1. 发送程序准备发送给对方的数据
		2. TCP发送但未确认的数据
	- 接收缓存用来存放：
		1. 按序到达但尚未被接受程序读取的数据
		2. 未按序到达的数据
	- 如果接收程序来不及读取收到的数据，接收缓存最后会被填满，接收窗口会减少到0；反之如果接收应用程序能及时读取数据，接收窗口就可以增大，但不会超过接收缓存的大小
	- A的窗口并不总是和B一样大（因为窗口值的传递有网络延迟）
	- TCP对不按序接收的数据会先存放在接收窗口中，等到缺少的字节收到之后再按序交付给上层进程
	- TCP要求接收方必须有累积确认功能以便减小传输开销（接收方不应该过分推迟发送确认）；接收方可以在自己有数据发送时顺便捎带上确认信息（少见）
	- TCP的通信方式为全双工，每一方都有自己的发送与接收窗口
2. 超时重传时间的选择
	- 报文往返时间RTT
	- 报文加权平均往返时间$RTT_s$:S=smoothed（因为加权了所以更加平滑）  
		$new RTT_s=(1-\alpha)\times (old RTT_s)+\alpha \times (new RTT sample) $  
		$0 \leq \alpha < 1$
		- 若$\alpha$接近于0，表示新的RTT样本影响不大（更新慢）
		- 若$\alpha$接近于1，表示新的RTT样本影响较大（更新快）
	- 超时重传时间RTO(Retransmission Time-Out)：应该略大于上面计算的$RTT_s$
	- 对于“如何判定此确认报文段是对先前的确认或者是对后来重传的确认？”这个问题的解决方法：
		- Karn算法：报文段每重传一次，就把超时重传时间RTO增大一些
3. 选择确认SACK(Selective ACK)
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(52).png)

 	- 如上图，一段字节块需要两个指针指出边界
 		- 左边界指出字节块第一个字节的序号
 		- 右边界-1指出字节块最后一个字节序号
 	- 使用首部的选项部分（扩充首部）储存SACK选项
 		- 最多只能指明4个字节边界块信息
 		- 每个序号需要**4个字节**$\times$每个边界块需要**2个序号**+**1个字节**表示使用SACK+**1个字节**表示占用字节$\leq$40字节（选项最大长度）

###  TCP的流量控制
1. 利用滑动窗口实现流量控制
	- 流量控制(flow control)：让发送方的发送熟虑不要太快让接收方来得及接收
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(53).png)
	- TCP窗口单位是字节
	- 死锁局面：
		- 在B向A发送零窗口报文后，B接收缓存又有了储存空间，于是B向A发送了rwnd=400的报文段，可是这个报文段丢失了。造成双方都在等对方响应的死锁局面
		- 解决方案：TCP为每一个连接设置了一个持续计时器(persistence timer)，只要接收到零窗口报文，就会启动计时器，计时器到期就发送一个零窗口探测报文段，对方会在确认这个报文段的时候给出现在的窗口值。
2. TCP的传输效率
	- TCP报文的发送时机：
		1. TCP维持一个变量等于最大报文长度MSS，只要缓存数据到达MSS，就发送一个TCP报文段
		2. 由发送方进程要求发送报文段，如推送push操作
		3. 发送方的某个计时器到了，就发送
	- Nagle算法：
		1. 发送方把要发送数据的第一个字节发送出去，然后把后面到达的字节缓存起来
		2. 当发送方接收到第一个数据的确认时，再将缓存中的所有数据封装成一个报文段发送
		3. 只有在收到前一个报文段的确认后才能发送下一个报文段
		4. 当缓存的数据已达到发送窗口大小的一半或已达到MSS时，就立刻发送一个报文段
	- 糊涂窗口综合症(silly window syndrome)
		- 接收方的应用进程读取缓存速度太慢，导致接收方每次发送确认时的接受窗口都很小，使得网络效率很低
		- 解决方案：让接收方等待一段时间，或者等到接收缓存已经➡️一半空闲时，才发出确认报文

###  TCP的拥塞控制
1. 出现资源拥塞的条件：对资源需求的总和>可用资源
2. 拥塞控制vs.流量控制
	- 拥塞控制是一个全局性的过程，涉及到所有主机、所有路由器
	- 流量控制指在点对点发送端与接收端之间的通信量控制
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(54).png)
3. TCP拥塞控制的方法
	1. 慢开始与拥塞避免算法
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(55).png)
		- 发送方维持**拥塞窗口cwnd**(congestion window)
		- 发送方让自己**发送窗口=min（拥塞窗口，发送窗口）**
		- 维持拥塞窗口的原则：没有出现拥塞就增大窗口，反之亦然
		- 慢开始：由小到大逐渐增加拥塞窗口数值（而非一次梭哈）
		- 慢开始门限sshtresh：
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(56).png)
		- 拥塞避免：不是指完全避免拥塞，而是在这个阶段将拥塞窗口控制为按线性增长，让网络较不容易出现拥塞
	2. 快重传算法
		- 能让发送方尽早得知丢包的发生
		- 接收方在接收到不连续的报文时，需要对已收到的报文段马上重复发三次确认
		- 发送方收到接收方发的重复确认应当立即重传丢包的数据
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(57).png)
	3. 快恢复算法
		- 在快重传算法运行后，直接将sshtresh=cwnd/2然后执行拥塞避免算法
		- 而不是重新执行慢开始算法
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(58).png)

###  TCP的运输连接管理
1. 运输连接有三个阶段：**连接建立**，**数据传送**，**连接释放**
2. 客户机服务器方式(C/S)：主动发起连接建立的进程叫客户(client)、被动等待连接建立的进程叫服务器(server)
3. 连接建立：**三次握手**
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(59).png)
4. 连接释放
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(61).png)

##  Chapter6——应用层
###  域名系统DNS
1. 域名系统DNS(Domain Name System)
	- 用途：把域名解析为IP地址
	- DNS为联机分布式数据库系统
2. 域名结构
	- 域名由英文，数字与（-）组成，不区分大小写
	- 级别最低的域名在最左边，级别最高的域名在最右边
	- 域名只是逻辑概念，与物理地点无联系
	- 域名树组成域名结构，从上到下分别为**根、顶级域名、二级域名。。。**
	- 域名解析由根DNS服务器开始逐层往下解析，每层的每个域名都负责解析各自下面的域名
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(62).png)
3. 配置自己的DNS服务器
	- 解析内网自己的域名
	- 降低到Internet的域名解析流量
	- 创建自己的域环境

###  动态主机配置协议DHCP
1. 动态主机配置协议DHCP(Domain Host Configuration Protocol)
	- 用途：为PC自动分配动态IP地址
2. DHCP客户端请求IP地址的过程
	1. 客户端在互联网上发布广播
	2. DHCP服务器从地址池中选择一个IP地址发给客户端（可能有多个DHCP server）
	3. 客户端发送接收特定IP地址的通知给DHCP server
	4. DHCP server发送子网掩码、网关、DNS server给客户端
3. 跨网段地址分配
	- 通过对路由器端口设置，将请求IP的广播包定向发给DHCP server
	- ip helper address ""

###  文件传输协议FTP(File Transfer Protocol)
1. FTP采用C/S模式
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(63).png)
2. 客户端与服务器传数据时并行运行两种进程
	- 控制进程：进行控制连接的进程，server使用**21端口**
	- 数据传送进程：进行数据连接的进程，有主动与被动模式
3. 控制连接：进行数据控制的连接（传送文件操作命令），标准端口为**21**
4. 数据连接：实际传输数据的连接，有两种模式（站在server角度看），标准端口为**20**
	- 主动模式：client告诉server要用什么端口，server主动使用**20端口**与此端口建立连接
	- 被动模式：server被动打开一个端口（针对每一个client不同）等待client发起连接
5. FTP传输模式：ASCII模式、二进制模式
6. FTP server如果有防火墙需要进行端口控制，需要打开20与21端口，然后使用主动模式连接

###  远程终端协议TELNET
1. 默认使用TCP**23端口**
2. 也是使用C/S模式，在本地运行**TELNET客户端**进程可以通过TCP登入远地的一个运行**TELNET伺服器**进程的主机

###  远程桌面协议RDP(Remote Desktop Protocol)
1. 有UI的TELNET

###  万维网WWW(World Wide Web)
1. WWW概述
	- WWW提供分布式服务
	- WWW是一个超媒体(Hypermedia)系统
	- WWW以C/S方式工作：Client向Server发出请求，Server向Client发回它需要的WWW文档
	- 互联网中网站的标识：
		1. IP地址
		2. 端口号
		3. 域名
2. 统一资源定位符URL(Uniform Resource Locator)
	- URL用来表示从互联网得到的资源的位置与访问这些资源的方法
	- 互联网上的所有资源都有一个唯一确定的URL
	- URL中不区分大小写
	- URL的形式由以下四个部分组成：
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(64).png)
		1. 协议：指使用什么协议获得万维网文档，常用的有HTTP、FTP
		2. 主机：是一个网站的域名Domain
		3. 端口：指出连接该主机使用的端口（省略则使用协议默认端口）
		4. 路径：指出当前页面在该主机的路径（省略则现实主机的主页面home page）

3. 超文本传输协议HTTP
	- HTTP使用TCP**80端口**
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(65).png)

4. 代理伺服器Proxy server
	- Proxy server是一个网络实体，又被称为Web cache
	- Proxy server可以把最近的一些请求与响应暂存在本地磁盘中，能快速响应请求，减少内网访问Internet的带宽
	- Proxy server可以限制特定主机连接外网
	- Proxy server可以绕过防火墙
	- Proxy server可以避免跟踪

5. 在伺服器存放用户信息Cookie
	- 工作原理：
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(66).png)

###  电子邮件E-mail
1. Internet中收发邮件的过程
	- 邮件客户端/用户代理UA(outlook,Mail,etc.)通过实名中继(通过账号密码登入)连接上邮件伺服器(126,yahoo,icloud,etc.)收发email
	- 伺服器通过SMTP发送邮件，POP3/IMAP接受邮件到另一个邮件伺服器
2. 简单邮件传送协议SMTP
	- 使用TCP25端口
	- SMTP不使用中间邮件伺服器
	- UA向发送方服务器发送邮件、发送方伺服器向接收方伺服器发送邮件使用的协议
3. 邮件读取协议POP3/IMAP
	- UA从接收方伺服器读取邮件使用的协议
	- POP3会将邮件下载到UA中
	- IMAP是一个联机协议，只有当用户打开邮件的时候才会下载到本地


##  Chapter7——网络安全
###  网络安全问题概述
- 安全包含哪些方面：
	1. 数据存储安全
	2. 应用程序安全
	3. 操作系统安全
	4. 网络安全
	5. 物理安全
	6. 用户安全教育
- 主动攻击与被动攻击
	1. 主动攻击：中断（有意中断他人网络通信）、篡改（故意篡改网络上的报文）、伪造（伪造信息在网络传送）、恶意程序(rogue program)
	2. 被动攻击：截获（窃听他人通信内容）
- 恶意程序
	1. 计算机病毒(computer virus)：会传染给其他程序的程序，传染是通过修改其他程序来把自身或其变种复制进去完成的
	2. 计算机蠕虫(computer worm)：利用网络通信功能将自己从A发到B并自动启动运行的程序
	3. 特洛伊木马(Trojan horse)：它执行的功能与宣称的功能不同而是有某种恶意功能
	4. 逻辑炸弹(logic bomb)：一种当运行环境满足某种条件时执行其他功能的程序
	5. 后门入侵(backdoor knocking)：利用系统实现的漏洞通过网络入侵系统
	6. 流氓软件：一种未经允许便在计算机上安装运行损害用户利益的软件

###  加密技术
1. 对称加密
	- 加密密钥与解密密钥相同,加密方与解密方共享一个相同的密钥
	- 缺点：
		1. 密钥不适合在网络传输
		2. 维护密钥成本大（两两之间就有一对密钥）
	- 优点：
		1. 效率高
	- 数据加密标准**DES算法**
		1. 对明文分组，每一组长度为64位
		2. 对每一组64位数据进行加密为64位密文
		3. 将各组密文串接起来得出整个密文
		4. 密钥长度为64位，包含8位奇偶校验（现多为128位）
		5. DES算法保密性取决于密钥长度
2. 非对称加密
	- 加密密钥与解密密钥不同，分为**公钥**与**私钥**
	- 私钥自己保留，公钥公开
	- 私钥加密能用公钥解密，公钥加密能用私钥解密
	- 优点：
		1. 公钥可以在网络传播（公钥公开）
		2. 多个用户可以共用同一公钥
	- 缺点：
		1. 效率低
3. 应用：
	1. 使用非对称公钥加密对称加密密钥，然后用对称加密加密信息（兼顾安全与效率）
	2. 数字签名：用私钥加密（签名）可以防止抵赖、确保不被更改
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(67).png)
	3. 加密的数字签名
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(68).png)
	4. 鉴别
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(69).png)
	5. 证书颁发机构CA(Certificate authority)
		- 为企业与用户颁发数字证书，确认企业与个人身份
		- 颁发证书吊销列表
		- 企业与个人信任证书颁发机构

###  Internet的安全协议
#### 安全套接字SSL(Secure Socket Layer)
- 位置：应用层与传输层之间
- 功能
	1. 发送方：加密应用层的数据之后传输给传输层的TCP套接字
	2. 接收方：从TCP套接字读取数据解密后传输给应用层
- 应用
	1. https TCP-443端口
	2. imaps TCP-993端口
	3. pop3s TCP-995端口
	4. smtps TCP-465端口
- 原理与工作过程
![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(70).png)
- 提供的服务
	1. 服务器鉴别
	2. 客户鉴别
	3. 加密会话

#### IPsec(IP security)
- 位置：网络层
- 安全关联SA(Security Association)
	- 定义：在使用AH或ESP之前，要从源主机到目的主机建立一条网络层的**单向**逻辑连接，这个逻辑连接就叫做安全关联SA
	- 作用：SA时IPsec的基础，是两个通信实体通过IKE协议协商建立的协定，它决定了：
		1. 用来保护数据安全的协议(AH,ESP)
		2. 转码方式
		3. 密钥
		4. 密钥的有效时间
- 两个主要协议：
	- 鉴别首部AH(Authentication Header)协议：
		1. 鉴别源点
		2. 检查数据完整性（数字签名）
		3. 不能保密
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(71).png)
	- 封装安全有效载荷ESP(Encapsulation Security Payload)协议：
		1. 较AH复杂
		2. 鉴别源点
		3. 检查数据完整性
		4. 提供保密
		![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(72).png)

#### 数据链路层的安全
- 可以通过设定路由器PPP协议进行实名验证与加密

#### 防火墙Firewall
- 由软硬件构成的系统
- 是一种特殊编程路由器，实施访问控制策略
- 防火墙内的网络称为“可信网络”，反之称为“不可信网络”
- 防火墙技术分类
	- 网络层防火墙：基于数据包、源地址、目标地址、协议和端口、控制流量
	- 应用层防火墙：数据包、源地址、目标地址、协议端口、用户名、时间段、内容、防病毒
- 防火墙网络拓扑
	- 三向外围网
	- 背靠背防火墙
	- 单一网卡
	- 边缘防火墙
	![Alt](https://raw.githubusercontent.com/Joyee691/image-hosting/main/Computer%20Network/img-ref-10(73).png)

## Chapter8——Internet上的音频/视频服务

### 视频/音频服务面临的问题
1. 延迟（对交互式/实时的影响尤其大）
2. 带宽不稳定（可以通过缓存/播放延时减缓）
3. 占用带宽高

### 视频/音频服务的三种类型
1. 流式(streaming)**存储**音频/视频——边下载边播放
	- 特点：节省客户端物理空间、保护版权
	- 例子：Youtube,Bilibili
2. 流式**实况**音频/视频——边录制边发送
	- 特点：实时编码至**流媒体服务器**传送至客户端
	- 例子：直播、实况
3. **交互式**音频/视频——实时交互式通信
	- 特点：双向实时编码传送
	- 例子：视讯、通话、IP电话

## End









