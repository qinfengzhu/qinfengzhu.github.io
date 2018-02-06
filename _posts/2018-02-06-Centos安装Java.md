---
layout: post
title: "Centos安装Java"
date: 2018-02-06  
tag: 开源学习
---

### Centos安装Java

1. 下载`Java`,[download java](http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz?AuthParam=1517896878_83c799462bef8ecd0ecea2b4b14dc1a8)

```shell
wget http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz?AuthParam=1517896878_83c799462bef8ecd0ecea2b4b14dc1a8
```

2. 解压Java压缩包到指定位置 `/usr/java/jdk-8u161`

```shell
mkdir -p /usr/java
tar -zvxf jdk-8u161-linux-x64.tar.gz -C /usr/java
```

3. 写入环境变量 

```shell
vi /etc/profile

//写入的内容
JAVA_HOME=/usr/java/jdk1.8.0_161
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
export JAVA_HOME CLASSPATH PATH

//wq!

source /etc/profile
```

4. 验证是否安装成功

```shell
java -version
echo $JAVA_HOME
```

### 当国外的 yum 源不可用的时候，调整为阿里云的yum源

```shell
rm -rf /etc/yum.repos.d/*
wget -O /etc/yum.repos.d/Centos-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum clean all
yum makecache
```
