---
title: 基于网络回溯分析技术的异常行为分析
layout: post
author: WenfengShi
category: 技术
tags: [security]
---
博客链接: [http://codeshold.me/2017/03/backtrack_technique.html](http://codeshold.me/2017/03/backtrack_technique.html)

> ISC安全课程（http://www.ichunqiu.com/course/57277）自学知识点总结和补充  
> 关键字：TAP方式、蠕虫网络、木马、僵尸网络、DNS放大攻击

## 0x0 网络回溯分析技术简介
1. 回溯分析以数据包基础（保存最基础的网络数据）
2. 通过七层的分析来透视网络传输的数据
3. 核心作用
- 了解（网络和应用运行规律、用户行为特点）
- 发现（安全隐患、异常通讯行为特征）
- 证明（历史数据回溯、数字取证）
4. 网络流量获取
- 流量镜像（交换机）
- TAP方式（分路器、分光器直接获取网络链路上的物理信号从而获取网络流量）
5. 应用
- 提供了透视网络行为和事件追溯的重要手段
- 发现并追踪定位可疑通信行为
        - 流量突发、蠕虫传播、木马／僵尸网络、DOS攻击、渗透攻击等
- 各类安全设备警报事件分析验证
- 恶意样本网络行为特征分析
- `评估系统性能、瓶颈`
        
## 0x1 流量突发监测分析方法
1. 流量突发监测分析方法
- 影响
        - 导致网络拥塞（利用率长时间超过70%）
        - 产生丢包、延时、抖动，网络质量下降
        - 造成网络无法提供正常的服务
- 成因
        - 突发大数据传输，P2P应用
        - 网络设备配置问题（路由环路、交换环路）
        - DoS攻击（DNS、NTP、SSDP放大攻击等）
        - 蠕虫、病毒爆发
        - 主机、操作系统、应用程序异常
        - 网络设备故障
- 方法
        - 寻找源头（有无明显大流量的应用、主机、会话）
        - 进一步判断产生的原因（网络行为分析，DOS？配合原始数据包解码分析协助定位）
2. 流量突发分析案例
- 科来网络回溯分析系统
- 定位、行为分析（查看数据包 wireshark或特定的分析工具）

## 0x2 蠕虫网络
1. 蠕虫监测分析方法
- 蠕虫通过网络主动复制自己传播的程序
- 传播途径：邮件蠕虫（Loveletter）、即时通信漏洞（MSN/Worm.MM）、操作系统或应用网络漏洞（CodeRed，Nimda）
- 网络行为特征
        - 网络层（同大量主机通讯；大多是发包、每个会话流量很少）
        - 会话层（会话连接很多；大多是发出的TCP SYN包，大部分没有响应或被拒绝）
        - 总体流量不一定很大，但发包远大于收包数量
- 蠕虫传播
        - 扫描阶段（TCP握手包数量异常、IP会话数异常、连续行扫描）
        - 渗透传播阶段（同一目标多次会话、会话内容编码可疑）

2. 蠕虫分析案例
- syn包较多、syn-ack包较少！
- 单位时间内有大量的TCP会话！
- 在段时间内扫描了大量的主机
- 某IP中了CIFS病毒(通用Internet文件系统 在windows主机之间进行网络文件共享是通过使用微软公司自己的CIFS服务实现的。
)
- 每次通讯都传输相同的内容，时序图内容完全一样
- 通信内容雷同，唯一变化的就是密码部分
- 通过警报设置预警同类问题
- 总结：找到流量异常处、分析数据包和行为模式、设置预警

## 0x3 木马和僵尸网络分析方法
1. 木马行为特征
- 监测分析方法
        - 木马隐蔽性很强、易于伪装
        - 传输途径多，如邮件、挂马、漏洞（渗透提权再植入）
        - 与病毒进行捆绑（如蠕虫病毒）
        - 特殊处理的兔杀木马（把杀毒软件报有病毒或是有恶意行为的软件经过修改后杀毒软件可以正常通过，方法有修改软件的源代码，加壳，或是修改软件入口），可以躲过绝大部分查杀
- 网络行为模型
        - 服务端（肉鸡）主动和客户端（控制端）进行连接，即反向连接
        - 先查询木马客户端域名对应的IP地址；再进行反向连接
        - 通过会话的时长进行分析（通信量又很少）
        - 可能会通过常用的端口进行通信（隐蔽信道的方式），通过监测其编码方式进行判断
2. 木马分析案例
- 域名解析阶段（可以域名解析请求）
        - 有两个域名解析频率很高
        - 看似无关的域名解析到相同的IP
- 上线连接阶段（可疑外网通信对象；频繁对外连接请求）
        - 追踪可疑IP通讯会话
        - 提取通信数据流量，发现起下载了zip文件和ini文件
        - google、whois自己查询
- 上线后（可疑长链接、周期性心跳、特征编码、隐蔽信道传输）
3. 僵尸网络特征
- 常见僵尸网络行为模型
        - 传播（漏洞攻击、邮件携带、即时通信、网站挂马、网络共享）
        - 感染（下载文件、关闭特定程序、执行IRC客户端程序
        - 集合（BOT向C&C服务器发送信息，是用户计算机加入僵尸网络，然后等待命令）
        - 接受控制（接受到C&C命令，然后执行，很多时候是通过下载配置文件进行执行）
- 僵尸网络域名解析特征
        - 会利用大量的域名（规避安全设备的域名特征库；会产生大量的奇怪的域名解析）
        - 往往使用动态域名，成本低廉、管理不严格的顶级域名
        - 常见的使用域名有`*.cc`, `*.ws`, `*.info`, `*.do`
        - 通过回溯分析追踪历史上与蠕虫C&C服务器IP有过通讯的主机
        
## 0x4 DoS攻击监测分析方法
1. DoS攻击基本原理
- 消耗带宽([Smurf][1], UDP flood, DNS放大)；消耗计算机资源（ping of death, syn flood）
- DNS放大攻击
        - 请求包小，相应包大，可疑放大几倍、几十倍
        - 准备条件：1. 僵尸网络上的主机；2. 攻击者申请一个做攻击的要解析的域名（将解析内容更大），让肉鸡到各互联网上的DNS服务器上进行查询，让DNS服务器上缓存该域名的记录
        - 60字节的请求，返回512字节的响应（type=255)更大
- 受害者端特征
        - DNS流量占比极高
        - DNS流量平均包长超过1000B
        - 大量的TYPE＝255的应答包
        - 可能出现大量IP分片报文
- 跳板机特征
        - 单个外部域名解析请求频繁（潜伏期）
        - 频繁出现TYPE＝255的查询和应答（潜伏期）
        - 攻击发生时特征与受害端接近
2. DNS放大攻击案例分析
- 对DNS服务器请求type=255的纪录，这个纪录最多可存放5K的数据
- 解决方式
        - 受害者：无解，需要借助运营商
        - 跳板机：关闭递归或进行[递归查询][2]验证（window服务器不支持递归查询验证）
- DNS(53, type=255), NTP(123, monlist), SSDP(1900, UPnP)
        


  [1]: http://blog.csdn.net/wdkirchhoff/article/details/45560513
  [2]: http://blog.csdn.net/wuchuanpingstone/article/details/6720723