---
layout: post
title: "Centos安装gitlab"
date: 2018-02-06  
tag: 开源学习
---
### Centos安装 `gitlab`,确保服务器硬件资源(2颗核心+4G内存),这已经是最低配了

1. 安装前预备操作

```shell
//安装服务与启动sshd服务、打开防火墙
yum install -y curl policycoreutils-python openssh-server
systemctl enable sshd
systemctl start sshd
firewall-cmd --permanent --add-service=http
systemctl reload firewalld

//可选的开启邮件服务
yum install postfix
systemctl enable postfix
systemctl start postfix

```

2. 因为国外的源需要翻墙，所以先配置清华大学的mirror

```shell
mkdir /etc/yum.repos.d/gitlab-ce.repo

//vi /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1

//wq!
yum clean all
yum makecache
```

3. 接下来安装 `gitlab`

```shell
yum install gitlab-ce
//根据配置生成运行环境
gitlab-ctl reconfigure
//启动gitlab ,查看端口号: netstat -ntlp |grep 80
gitlab-ctl start
//查看gitlab各个服务的状态啊
gitlab-ctl status
```

4. 修改配置的地方

```shell
//位置:/etc/gitlab/gitlab.rb 
gitlab-ctl reconfigure
//一般为
external_url 'http://192.168.1.106:90' 或者改为域名
```

5. 汉化工作

gitlab网站app在目录,`/opt/gitlab/embedded/service/gitlab-rails`,汉化的时候，我们只需从
[gitlab 汉化版下载](https://gitlab.com/larryli/gitlab) 去替换就好。

```shell
//git clone 下来
git clone https://gitlab.com/larryli/gitlab  
//备份好原来的文件
tar -zcvf /opt/gitlab/embedded/service/gitlab-rails.tar.gz /opt/gitlab/embedded/service/gitlab-rails
//删除老文件
rm -rf /opt/gitlab/embedded/service/gitlab-rails/
//拷贝汉化文件
cp -rf gitlab/* /opt/gitlab/embedded/service/gitlab-rails/
```

6. 可能出问题的地方

```shell
//selinux没有关闭
getenforce 
vi /etc/selinux/config
调整 SELINUX=disabled
setenforce 0


//防火墙
systemctl status firewalld 
systemctl stop firewalld
systemctl disable firewalld
```
7. [可以参考的文章](http://blog.csdn.net/chenjh213/article/details/50527877)