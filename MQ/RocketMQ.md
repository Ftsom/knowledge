## RocketMq：阿里托管apache消息队列开源组件

## 结构：
	Name Server：自研注册中心，用于producer和consumer的路由机制，提供心跳检测机制
	Broker：存储消息
	Producer：生产消息客户端
	Consumer：消费消息客户端

## 模块：	
	Name Server cluster：互相之间不联系
	Broker：Remoting Module（远程连接）、Client service（客户端）、HA Service（高可用）、Store Service（存储）、Index Service（索引）
	Producer Cluster：生产者集群
	Consumer Cluster：消费者集群

## 集群启动：
	1.Name Server cluster启动，等待broker注册信息；120s故障感知机制，每10s轮询broker，如果120s没有心跳则剔除
	2.broker启动，连接上所有的name server机器，进行注册，每30s进行一次上报，包括topic信息
	3.producer启动，连接一台name server机器获取topic的broker信息，选择broker进行消息生产
	4.consumer启动，连接一台name server机器获取topic的broker信息，链接想要消费的broker，进行消息消费

## 文件组织：
	采用数据和索引分离的方式，数据统一放在一个文件，而消息的消费索引是放在另外的队列文件中，存储了消费的offset和size、tag hashcode
	1.commit log：消息实体的存储文件，固定大小1g，不足则新建
	2.consume queue：消息消费的队列，里面存储了commit log的索引位置和size，消费的时候通过索引和尺寸获取到commit log中存储的消息消费
	3.index file：索引文件，主要用于根据消息的key值进行查询，可以理解成是一个消息 key的hash存储

## 高性能：
	1.顺序写：消息文件使用顺序写的方式
	2.zero copy（Mmap）：减少用户空间和内核空间下的数据拷贝过程
	3.broker水平扩展

## 高可用：
	1.主从机制：broker支持主从，通过数据同步机制完成消息从主到从的备份，从支持消息的读取，后期支持自动切换master
	2.发送高可用：同步复制、非同步复制
	3.消费高可用：

## 特性：
	可靠性：刷盘方式决定不丢消息或者丢少量消息
	At least Once：消息消费之后发送一个ack返回，如果没有返回会做相应的重试等机制
	事务消息：2pc形式的分布式事务支持
	延迟消息：固定level的延迟消息，将延迟消息存储到延迟队列，通过定时触发移动到正常的消息队列进行后续流程
	回溯消费：按照时间回溯消费
	消息重试：生产者消息发送重试、消费者消息消费重试
	死信队列：重复消费消息到一定的阈值，存储到该队列
	流量控制：生产者流控、消费者流控
