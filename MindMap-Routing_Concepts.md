# 路由基础

## 互联网络的由来

- 两台PC用一条双绞线相连组成最小的局域网
- 通过集线器连接PC
	- 集线器是线缆的扩展，仅仅起到信号放大和整形的作用
	- 不能隔离冲突域也不能隔离广播域
- 通过交换机连接PC
	- 可以隔离冲突域，但不能隔离广播域
- 广播域过大会产生什么问题
	- 广播风暴
	- 安全问题
	- 组网简单，但是不灵活
- 通过路由器连接不同网段
	- 路由器可以隔离广播域

## PC的工作原理

- 如果目标地址和自己的IP地址在同一网段，以目的地址做ARP,取得对方的MAC地址，然后和对方通信。
- 如果目标地址和自己的IP地址不在同一网段，以网关做ARP,取得网关的MAC地址，然后把包发给网关。
- PC设置网关的作用是？
- PC什么时候一定要设置网关？什么时候可以不设置网关？

## 交换机的工作原理

- 根据目的MAC转发
- 学习源MAC
- 组播、广播、未知单播泛洪

## 路由器的工作原理

- 转发、查询、建立路由表
- 1.查询路由表：依据目标IP地址转发
- 2.建立路由表：路由表通过动态路由协议、静态路由协议等进行填充
- 3.包转发：单播，组播转发、广播隔离（特例：全广播隔离，定向广播默认转发）
- 什么是最长匹配
- 什么是递归查询

## 数据包转发的过程

- 数据包的转发（路由器的转发功能）
	- 目标IP地址是自己
		- 拆开头部，看具体内容
	- 目标IP地址不是自己
		- 目标IP地址是非直连网段，但是在路由表中：查找下一跳地址的MAC地址（ARP缓存表），封装，转发
		- 目标IP地址是直连网段：查找目标IP地址的MAC地址（ARP缓存表），封装，转发
		- 目标IP地址在不在路由表中：丢弃数据包
- 经过交换机，不会改变帧，源目MAC地址不会改变，源目IP地址不会改变
- 经过路由器，会改变帧，源目MAC地址会改变，源目IP地址不会改变

## 路由的元素

- 目标网络和前缀
- 下一跳和/或出接口
- 管理距离
- 度量值
- 路由类型
- 用于比较优先级
- 必需的

## 路由的类型

- 直连路由：C
	- 两个前提条件
		- 接口配置IP地址
		- 接口状态是UP
	- 直连路由是自动产生（满足两个前提条件）、自动填充到路由表中的
- 静态路由：S
	- 静态路由的优缺点
		- 1.对CPU、内存等硬件的需求不高
		- 2.不占用带宽
		- 1.配置工作量大且容易出错
		- 2.适应拓扑变化的能力较差
	- ip route 2.2.2.0 255.255.255.0 12.1.1.2
- 动态路由：D
	- 概述
		- 动态路由是“一台路由器通过其他路 由器了解到的路径信息
		- 因此管理员配臵动态路由协议时的工作，就不再是告诉路由器去往某个网络的路径了，而变成了“告 诉路由器要与其他（使用这个协议的）路由器分享哪些自己的网络”
		- 不同的动态路由协议，它们的区别常常是采用了不同的路由信息分享 规则，而这个规则就称为路由算法。
	- 动态路由算法的工作
		- 1. 向其他路由器传输路由信息
		- 2. 接收其他路由器传输过来的路由信息
		- 3. 根据路由信息计算出去往各个目的网络的最优路径并生成路由表
		- 4. 对网络拓扑的变化及时作出响应，更新路由表并把拓扑变化信息宣 告给其他路由器
	- IGP内部网关路由协议
		- 根据计算最优路径的方式，动态路由协议可以粗略地分为距离矢 量和链路状态两类
		- 距离矢量路由协议
			- RIP (* R)
			- EIGRP (* D, * D EX)
		- 链路状态路由协议
			- OSPF (* O, * O IA, * O E1, * O E2, * O N1, * O N2)
			- ISIS
	- EGP外部网关路由协议
		- BGP (* B)
- 子类：默认路由*
	- 默认路由=缺省路由
	- 默认配置和可选配置，默认路由是首选路由吗？
	- 默认路由是最后的选择，根据最长匹配原则，明细路由，汇总路由，最后才是默认路由
	- ip route 0.0.0.0 0.0.0.0 下一跳
	- 0.0.0.0代表不知道该怎么走的路由全扔给默认路由
- 静态路由和动态路由协议的比较

## 管理距离和度量值

- 路由比较的三个步骤（查询）
	- 最长匹配
		- 前缀越大越优先，也意味着知道的路由信息越详细
	- 管理距离
		- 不同路由协议之间进行比较优劣， 反映了网络的靠谱程度
	- 路由类型
	- 度量值
		- 相同路由协议不同路径进行比较优劣
- 管理距离
	- 直连路由：0
	- 静态路由：1（在静态路由中，可配置比默认值1大的管理距离，来设置浮动静态路由。或配置相同的管理距离来设置负载均衡。）
	- RIP：120
	- OSPF：110
	- EIGRP
		- 5（EIGRP的汇总路由）
		- 90
		- 170（外部路由，将其他路由协议重分布到EIGRP）
	- BGP
		- 20
		- 200
	- 其他：管理距离为255的路由不会在路由表中显示
- 度量值
	- 参数标准
		- 跳数：到达目的地会途径几座城市
		- 带宽：中途道路有多宽，路况好不好
		- 负载
		- 时延
		- 可靠性
	- 直连路由没有度量值
	- 静态路由没有度量值
	- RIP的度量值是跳数
	- EIGRP的度量值是复合度量值
	- OSPF的度量值是Cost

## 实验

- 实验一
```
	一分钟Cisco路由器变PC
	- no ip routing
	- int fa 0/0
	- ip add 192.168.1.1 255.255.255.0
	- no shutdown
	- ip default-gateway 192.168.1.254
```
- 实验二
```
	从PC到网关
	- 192.168.1.1 ping 192.168.1.254
	- show arp
```
- 实验三
```
	PC设置默认网关
	- 192.168.1.1 ping 2.2.2.2
	- show arp
```
- 实验四
```
	PC不设置默认网关
	- 192.168.1.1 ping 2.2.2
	- show arp
	- interface fa 0/0
	- no ip proxy-arp
	- 192.168.1.1 ping 2.2.2.2
	- PC interface fa 0/0 shutdown no shutdown 清空arp表
```
- 测试
```
	- ping 127.0.0.1
	- ping 192.168.1.1
	- ping 192.168.1.254
	- ping 8.8.8.8
	- nslookup www.baidu.com
```
- 关于loopback接口
	- 总是up
	- 一般用于测试连通性
	- 作为路由器的标识
		- router-id
	- 用于管理
