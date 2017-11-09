---
layout: post
title: Zookeeper特性一
date: 2017-11-06 
tags: Zookeeper    
---

### Zookeeper特性

集群中只要有超过过半的机器是正常工作的，那么整个集群对外就是可用的

也就是说如果有2个zookeeper，那么只要有1个死了zookeeper就不能用了，因为1没有过半，所以2个zookeeper的死亡容忍度为0；

同理，要是有3个zookeeper，一个死了，还剩下2个正常的，过半了，所以3个zookeeper的容忍度为1；

同理你多列举几个：2->0;3->1;4->1;5->2;6->2会发现一个规律，2n和2n-1的容忍度是一样的，

都是n-1，所以为了更加高效，何必增加那一个不必要的zookeeper呢。