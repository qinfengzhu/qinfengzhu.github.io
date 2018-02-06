---
layout: post
title: "Centos安装Zookeeper"
date: 2018-02-06  
tag: 开源学习
---
### 安装 `Zookeeper`

1. 下载 Zookeeper,进行简单配置并启动 

```shell
//下载并且解压
wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/stable/zookeeper-3.4.10.tar.gz
tar -zxvf zookeeper-3.4.10.tar.gz -C /usr/
//修改Profile文件，方便执行zk命令  /etc/profile
PATH=$PATH:/usr/zookeeper-3.4.10/bin
//先写个简单配置 
cd /usr/zookeeper-3.4.10/conf
mv zoo.simple.cfg zoo.cfg
//调整配置为
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181

//启动zookeeper
zkServer.sh start
```

2. 测试是否启动成功

```shell
zkServer.sh status   //如果出现standalone 表明单机部署成功
netstat -ntlp |grep 2181
```