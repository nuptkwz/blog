---
layout: post
title: "Linux环境下ElasticSearch单实例以及集群搭建"
category: 'ElasticSearch'
---

本文介绍了ElasticSearch的基本概念以及基于Linux环境下的单实例和集群安装

@[TOC]
# 什么是ElasticSearch
 - 基于Apache Lucene构建的开源搜索引擎
 - 采用Java编写，提供简单易用的RESTFul API
 - 轻松的横向扩展（增加节点数），可支持PB级别的结构化或者非结构化数据
 - 在elasticsearch中，索引被分为多个分片，每份分片是一个Lucene的索引。
#  ES的应用场景
 - 海量数据分析引擎（可利用es的聚合搜索功能来解决）
 - 站内搜索引擎
 - 数据仓库（ES具有强大的分布式数据处理能力，存储Pb级别的结构化或者非结构化数据）
# 环境
 - Maven、Spring Boot
 - IDE工具 IntelliJ IDEA
 - Java JDK1.8
 - 其他依赖 NodeJs(6.0以上)

# es的安装
本节es的安装主要分为单实例安装以及集群的安装，下面我们就从这两部分分别介绍具体操作
## 单实例安装

 - 首先打开es的官网的下载页面：[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)
![\[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Lw3rOCz0-1574696769464)(es)\]](https://upload-images.jianshu.io/upload_images/9905084-b51ac4e76c3a1078?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 - 右击复制链接地址，在linux环境下建一个es的文件夹下，将刚才复制的链接黏贴上去，进行下载：
 wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz
 - 我们进行解压：tar -vxf elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz
 - 启动es: cd到刚才解压的目录下运行  sh ./bin/elasticsearch ，它会打印一些日志，当我们看见started的时候，es就正常启动了
 - 我们进入浏览器输入127.0.0.1:9200我们可以看见es的服务正常启动了

## es的head插件安装
es的查询结果是json结构，head插件解决帮我们解决了界面的问题，除此之外，它还可以实现基本信息的查看，restful请求的模拟，以及数据的基本检索。
### 安装
 - 我们打开github搜索elasticsearch-head
[https://github.com/search?q=elasticsearch-head](https://github.com/search?q=elasticsearch-head)
 - 下载mobz/elasticsearch-head，然后解压，进入elasticsearch-head-master里面
 - 检查一下我们的Node环境，输入node -v ,看见版本是v8.2.1
 - 我们执行npm install，然后输入npm run start，可以看见9100就出来了
 - 在浏览器端输入http://localhost:9100，就可以看见ElasticSearch的界面了

我们如果在一个控制台同时运行elasticSearch和head插件，那么需要先要将es采用后台的方式启动，加上-d参数就行了。
### 跨域问题及修改
因为es和head插件属于两个独立的进程的，所以访问的话会遇到跨域问题，因此需要进行一下跨域的修改。
  vim config/elasticsearch.yml 我们在配置的最后面加上：
```yml
http.cors.enabled:true
http.cors.allow-origin:"*"
```
然后分别运行es和head插件，我们可以在localhost:9100界面上看见集群健康值，如果是**绿色**的代表很健康，**黄色**代表集群健康不是很好，但是可以正常使用，而**红色**代表集群的状况已经很差了，这个时候，虽然可以正常使用，但是已经出现了开始丢失数据的问题了。
## 分布式安装
head控制台不要关闭，然后我们新建一个窗口，我们尝试新建一个es的集群，这个集群包括三个节点，一个master和两个slave，master表示指挥官，slave就是这随从。
### master节点的安装
 把刚才那个作为master，修改elasticsearch.yml,在后面加上如下配置
```yaml
#集群的名字
cluster.name: keweizhou
#master节点的名字
node.name: master
node.master: true

network.host: 127.0.0.1
```
我们找到原来的服务ps -ef | grep 'pwd'，找到进程号xxx,然后kill xxx，然后我们重新启动一下原来的es服务，去浏览器检查一下我们的服务是否正常启动，http://localhost:9200。我们再用原生的API检查一下集群的名字，输入127.0.0.1可以看见，集群的名字已经变成keweizhou了。
### slave节点的安装
 - 新建一个目录 mkdir es_slave
 - 然后将es包拷贝到这个目录下，然后解压，解压之后复制重命名，分别为es_slave1，es_slave2。然后分别修改它的Yml配置。如下：
```yaml
cluster.name: keweizhou
node.name: slave1
network.host: 127.0.0.1
#指定一下es的端口号，不然会默认取9200端口，就会冲突
http.port: 8200
#这个需要加，不然它就游离于集群之外了，找不到指挥官了
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```
我们分别对它进行服务启动。./bin/elasticsearch -d
然后我们在head插件上就可以看见了：
![在这里插入图片描述](https://upload-images.jianshu.io/upload_images/9905084-8bddf20d48f8dd1c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 遇到的问题
 - **无法启动es**
 1.自己linux自带的openjdk的版本不支持，卸载之后，重新安装了jdk，配置环境变量就行了（值得注意的是要想环境变量立即生效，得用**source /etc/profile**）
 2.以后台的方式启动，再次启动es起不来，需要把原先的es进程给杀掉，再次启动就ok了

 参考：https://github.com/nuptkwz/notes/tree/master/technology/elasticsearch
