---
layout: post
title: "Redis--Redis cluster核心原理分析"
category: 'Redis'
---
 
> 本文主要分析了redis cluster的节点间通信机制，包括基础通信原理，gossip通信，jedis smart定位，主备切换。
# 节点间的通信机制
## 基础通信原理
1.redis cluster节点间采用gossip协议进行通信
  跟集中式不同，不是将集群元数据（节点信息，故障等等）集中存储在某个节点上，gossip协议是互相之间不断通信，保持个集群所有节点的数据是完整的
  ![集中式的元数据集群存储和维护.png](https://upload-images.jianshu.io/upload_images/9905084-f96e7bcee01ef1ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![gossip协议维护集群元数据.png](https://upload-images.jianshu.io/upload_images/9905084-fbb19615a10e65e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

集中式和gossip协议维护集群元数据对比：
**集中式**：
- 好处在于元数据的更新和读取，时效性非常好，一旦元数据出现了变更，立即就更新到集中式的存储中，其他节点读取的时候立即就可以感知到
- 不好在于，所有元数据的更新压力全部集中在一个地方，可能会导致元数据的存储有压力

**gossip**:
- 好处在于元数据的更新和读取比较分散，不是集中在一个地方，更新请求会陆陆续续，打到所有节点上去更新，有一定的延迟，降低了压力
- 缺点在于元数据的更新有延迟，可能导致集群的一些操作会有些滞后。因此在做reshard的时候，去做另外一个操作，会发现有configuration error

2.10000端口号
  每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如1端口号，节点间通信的端口号就是10001。每个节点每隔一段时间就会往另外几个节点发送ping请求，同时其他几个节点接收到ping之后会返回pong。

3.交换的信息
  节点间交换了故障信息，节点的增加和移除，hash slot信息，等等

## gossip协议
  gossip协议包含多种消息，包括ping，pong，meet，fail等等。
  **meet**: 某个节点发送meet给新加入的节点，让新节点加入集群中，然后新节点就会开始与其他节点进行通信（比如在执行add node的时候，内部就会发送一个gossip meet消息，给新加入的节点，通知那个节点去加入我们的集群）

**ping**: 每个节点都会频繁给其他节点发送ping，其中包含自己的状态还有自己维护的集群元数据，互相通过ping交换元数据（每个节点每秒都会频繁的发送ping给其他的集群，ping:频繁的互相之间交换信息，互相进行元数据的更新）

  **fail**: 某个节点判断另一个节点fail之后，就发送fail给其他节点，通知其他节点，指定的节点宕机了
## ping消息探入
 ping很频繁，而且要携带一些元数据，所以可能会加重网络负担每个节点每秒会执行10次ping，每次会选择5个最久没有通信的其他节点，当然如果发现某个节点通信延迟达到了cluster_node_timeout/2，那么立即发送ping，避免数据交换延时过长，所以cluster_node_timeout可以调节，如果调节比较大，那么会降低发送的频率



