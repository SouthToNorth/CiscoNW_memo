# 动态路由协议：RIP

## 特点

- 属于IGP DV协议（使用bellman-ford算法）
- 基于UDP 端口号520
- 周期性广（组）播更新 完整更新 把整个路由表发给邻居
- metric度量只考虑跳数
- 只支持等价负载均衡
	- 默认4条，最多6条
- 管理距离120

## 传递路由信息，通过什么方式传递

- 1.是否建立邻居？
	- show ip rip neighbor
	- 建立邻居之后才能互相交换路由信息
	- RIP直接通过广播、组播的方式路由信息扔出去，不知道自己有几个邻居，也不知道自己的邻居是谁
- 2.是广播、组播、单播更新？
	- RIPV1使用广播
	- RIPV2使用组播
	- 使用组播地址224.0.0.9
- 3.应用层还是传输层，TCP还是UDP,端口号是？
	- UDP 520
- 4.定期更新还是触发更新
	- 30s周期更新全部的路由表
- 5.全部更新还是增量更新
- …

## 版本

- RIPv1
	- 广播更新
	- 跨越主类边界自动汇总（不可关闭）
	- 不支持VLSM 更新不带掩码
	- 不支持认证
	- 不支持手工汇总
- RIPv2
	- 组播更新（224.0.0.0）
	- 跨越主类边界自动汇总（可关闭）
	- 支持VLSM 更新带掩码
	- 支持认证
	- 支持手工汇总
- RIPv2和RIPv1的对比
	- 结论是：RIPv1怎么怎么不好，RIPv2怎么怎么比RIPv1强（虽然也不怎么样），从而得出“大家以后不用RIPv1了，只用RIPv2就可以了”
  ```
  于是乎
	router rip
	version 2
	no auto-summary
  成了标准配置
  ```

## 拓扑变更和收敛

- 1.周期更新还是触发更新
- 2.收敛时间
- 3.失效时间——检测拓扑变更

## 冗余备份和负载均衡

## 防环机制

- 1.最大跳数——切断全网的环路：16跳
	- RIP的TTL值最大会设置成16 一个数据包传输了15跳后到了第16跳就会被丢弃以防止环路的出现
- 不可达路由直接置为16跳
- 2.水平分割——切断两台路由器之间的环路
	- **Split horizon** 水平分割 从一个接口收到的数据包不会从相同的接口发出（毒性逆转会打破水平分割）默认开启水平分割
- 3.抑制计时器 Holddown time（相当于冻结期 有三种情况）
	- holddown time时间内 不收任何条目 
		如果holdtime内收不到任何有关条目就砍掉该路由
	- 在Holddown time内路由恢复则继续采纳放入路由表
	- 在Holddown time内收到更好的路由则采纳更好的路由放入路由表
	- 在Holddown time内收到更差的路由则不采用更差的路由
- 更新源检查（防止路由黑洞）
	- 关闭更新源的检测可以收到直连的非本网络的路由更新
	```
	R(config)#router rip 
	R(config-router)#no validate-update-source 

	说明：validate-update-source  【Perform sanity checks against source address of routing updates
	```

## 实验

- 1.基本配置
	```
	(config)# network
	(config-router)# network 0.0.0.0 //意思是宣告所有直连网段
	```
- 2.查看路由协议
	```
	# show ip protocols
	能看到哪些信息？
	拓扑图：RIP
	```
- 3.查看路由表
	```
	# show ip route
	```
- 4.负载均衡
	```
	(config)# router rip
	(config-router)# maximum-paths xx
	```
- 5.触发更新
	```
	(config)# int s1/1
	(config-router)# ip rip triggred
	在串行接口上使用触发更新，以太网接口不支持触发更新
	设置触发更新的目的是减少路由更新包的传输，减少带宽的浪费
	```
- 6.被动接口
	```
	(config)# router rip
	(config-router)# passive-interface loopback 0
	只收不发
	```
- 7.单播更新
	```
	(config)# router rip
	(config-router)# passive-interface default
	(config-router)# neighbor 123.1.1.2
	```
- 8.水平分割
	```
	(config)# int fa 0/0
	(config-if)# no ip split-horizon
	```
- 更改定时器的值
	```
	(config)# router rip
	(config-router)# timer basic 40 240 0 320
	关于时间的修改原则：除非你知道你在干什么，否则什么都不要做
	```
- 关于VLSM和CIDR
	- 所谓VLSM就是，同一网络出现不同前缀的子网
	- 所谓CIDR，可以支持汇总和超网
	- 回顾：VLSM网络设计的思路
		- 每个网段50个主机位：192.168.1.0/26
		- 每个网段25个主机位：192.168.1.64/27
		- 每个网段15个主机位：192.168.1.96/27
		- 每个网段10个主机位：192.168.1.128/28
		- 每个网段5个主机位：192.168.1.144/29
		- 每个网段2个主机位：192.168.152/30
- 自动汇总
	- 一般情况下，自动汇总是关闭的
	- 自动汇总可能会出现问题，比如说不连续子网
	- 什么是不连续子网
	- (config-router)# no auto-summary
- 手工汇总
	- (config-if)# ip summary-address rip 190.1.0.0 255.255.252.0 在路由流向的出接口使用
	- 本地只有明细路由，从做汇总的接口发出汇总路由。
	- 直到明细的最后一条路由消失，汇总才会消失。
	- 使用最小的metric做汇总metric
- 默认路由
	```
	1）(config-router)# redistribute Static
	2）(config-router)# network 0.0.0.0
	3）(config-router)# default-information originate
	4）(config-router)# default-network
	```
- 验证
	```
	(config)# key chain test
	(config-keychain)# key 1
	(config-keychain-key)# key-string cisco
	(config)# int fa 0/0
	(config-if)# ip rip authentication mode md5
	(config-if)# ip rip authentication key-chain test
	```
- 汇总（补充）
	- 配置
		- 在RIP中关闭自动汇总后，可以在某一个接口去做手动的汇总。但要注意，汇总出的网络号不能越过主类网络。RIP不支持CIDR。
		```
		(config)#int s1/1
		(config-if)#ip summary-address rip 172.16.0.0 255.255.0.0 //在路由流向的出接口作
		```
	- 特点
		- 本地有汇总，才能从做汇总的接口发出汇总路由
		- 全部明细路由消失，汇总路由才会消失
		- 汇总路由metric值取明显条目中最小的metric值
- 认证（补充）
	- 支持方式及特点
		- 明文认证匹配规则：
			- 只发送KEY ID最小的KEY，并不携带KEY ID，接收方与KEY列表中所有的KEY匹配，只要有一个能匹配上则通过认证。
		- MD5密文认证匹配规则：
			- 发送时：只发送最小的KEY ID，并且携带KEY ID。
			- 接受时：当接收到时，先只匹配相同KEY ID密钥，如果不匹配，则通不过认证。但如果没有相同KEY ID，只向下查找一次大的KEY ID密钥，不匹配也不通过认证，并且不会向下查找了
	- 配置
		- 第一步：定义密码库
			```
			(config)# key chain R2 //本地有效
			(config-keychain)# key 1 //建议两端一致
			(config-keychain-key)# key-string cisco
			可以定义多个KEY值，按从小到大的顺序进行匹配，发送KEY值时也是发送最小的一个，还可以设定KEY值的有效时间。
			```
		- 第二步：在接口下应用密码库
			```
			(config)# int s0
			(config-if)# ip rip authentication key-chain R2
			```
		- 第三步：在接口下指定认证模式
			```
			(config)# int s0
			(config-if)# ip rip authentication mode [md5|text]
			```
		- 查询
			```
			R1# show key chain
			R1# debug ip rip
			// 小知识：RIP中每一个路由更新报文最大可包含25条路由,做了明文认证后只能包含24条，做了MD5认证后只能包含23条。

			R2(config-keychain-key)# accept-lifetime 04:00:00 jan 2006 infinite //定时接收
			R2(config-keychain-key)# send-lifetime 04:00:00 jan 2006 04:01:00 jan 2006 //定时发送
			R2(config-keychain-key)# send-lifetime 04:00:00 jan 2006 duration 300  //有效期300S
			//注意：密码库中可以同时定义多个密码，在匹配时要按规则来匹配，明文认证和md5认证的匹配规则各不一样，下面分别说明。
			```
- 基本配置及调试（补充）
	- 基本配置及调试
		- RIP v1
			- 配置
			```
			router rip            //在路由器上启用RIP协议
			network 10.0.0.0    //宣告网络，只能主类宣告 
			·在CISCO路由器上，运行RIP后，默认即不是V1版也不是V2版，而是一种特殊状态。
			show ip protocols     //查看当前运行的协议
				Interface             Send Recv  
				Serial0/0             1    1 2  （默认）
			router rip
			version 1    //如果你想运行V1版，打上这条命令
				Interface             Send Recv
				Serial0/0             1    1   （指定v1)
			debup ip rip   //本命令可调试路由更新
			```
		- RIP v2
			- RIP-v2的特点
				- 以组播地址224.0.0.9发送更新。
				- 默认情况下路由在跨越主类网络边界时，会自动汇总，但是也可以关闭自动汇总，进行手动汇总
				- RIPv2支持VLSM，更新发送时携带掩码信息。
			- 配置
			```
			router rip
			version 2  //启用V2版
				Interface             Send Recv
				Serial0               2    2   （指定v2）
			·正常情况下，RIP-V2是发送组播更新，下面这条命令强制让RIP使用广播更新

			int s0
			ip rip v2-broadcast    //在接口下配置
			·虽然v2携带了掩码信息，但跨越不同网络边界时，默认还是会自动汇总成主类。

			router rip
			no auto-summary    //通过这一命令可关闭自动汇总
			·RIP在auto-summary时，会将本地及收到的路由都汇总成主类然后发出。
			```
		- 实现V1和V2版的兼容性
			```
			int s0
			ip rip send version 1       //设定接口只发送出V1版的更新
			ip rip receive version 1    //设定接口只接收V2版的更新
			ip rip send version 1 2     //设定接口同时发送V1和V2版的更新
			```
	- 下发默认
		- Redistribute Static
			```
			ip route 0.0.0.0 0.0.0.0 serial 0
			router rip
			redistribute static
			```
	- 宣告
		- Network 0.0.0.0
			```
			ip route 0.0.0.0 0.0.0.0 serial 0（写下一跳不行）
			router rip
			network 0.0.0.0
			default-network：
			ip default-network 12.0.0.0（写成主类）#
			如写成ip default-network 12.1.1.0
			```
	- 重分布
		- default-information originate
			```
			router rip
			default-information originate
			```
	- 参数调节
		- 计时器的调节——RIP Timer
			- 1）Update（30S）：随机变量是更新周期的15%，即4.5S（25.5S-30S)。
			- 2）Invalid（180S）：180S后置为Possible Down，之后立即启动hold Down计时器。
			- 3）Hold Down（180S）：实际只用到60S。
			4）Flush（240S）：240S还没收到路由更新，才将此路由删除。
			```
			测试Possible Down：
			1）设置Passive-interface
			2）中间接SW
			3）中间是FR
			4）认证不匹配
			router rip
			timers basic 10 20 20 40 //修改四个计时器
			```
		- ＜更新方式的修改——单播更新＞
			默认RIP只发送广播或组播更新包，下面的命令让RIP发出单播包
			router rip
			  neighbor 10.0.0.2
			版本二中启用广播更新
			R4(config-if)#ip rip v2-broadcast 
			触发更新-Triggered
			周期更新：路由器周期性的向外发送出自已的路由更新
			触发更新：路由器平时不会周期性的发送路由更新，只会在拓朴改变的情况下（也就是路由变化了），才会向外发送出路由更新。
			RIP默认只做周期更新，通过以下命令，可以实现触发更新。
			使用触发更新后：
			·路由器不再周期发更新，只触发更新。
			·计时器会自动变成Timers basic 30 180 0 240
			·只能打在低速点对点链路上,E0口是不能打的
			int s0
			  ip rip triggered （E0/Lo0不支持）
			　　　　　　　（两端都配）
			debug ip rip
			关闭更新-被动接口：
			在RIP协议中，如果一个接口被设定为被动接口，这个接口将不能向外发送路由更新，不过还可以接收对端发送过来的路由更新。
			router rip
			  passive-interface s1/0     //将接口设为被动接口，只收不发
			router rip
			  passive-interface default    //将所有接口设为被接口
			router rip
			  no passive-interface s1/0    //取消一个被动接口
			debug ip rip
		- 调节Metic以及负载均衡—偏移列表
			可以用来增加路由的metric值，需要先用ACL抓出路由
			access-list 1 permit 2.2.2.0 0.0.0.0
			router rip
			  offset-list 1 out 3 ethernet 0/0  //对ACL1所匹配的路由加三点的metric值　　　
			或：
			  offset-list 0 out 3 ethernet 0/0
			　　　　　　（0代表对所有路由）
			负载均衡指的是将去往一个特定目的地的多条路由同时放进路由表，用来做流量的转发。
			负载均衡有两种：
			1、等价负载均衡--将metric值相等的路由同时放入路由表用来做流量转发。
			2、不等价负载均衡--将metric值不相等的路由也同时放入路由表用来做流量转发。
			·RIP只支持等价的负载均衡。也就是说用来做负载均衡的路由metric必须一致。
			·默认RIP只支持四条路径的负载均衡，在新的IOS中可以通过命令修改为最多16条
			router rip
			  maximum-paths 2    //最多16条
			show ip protocols   //可以用来查看
