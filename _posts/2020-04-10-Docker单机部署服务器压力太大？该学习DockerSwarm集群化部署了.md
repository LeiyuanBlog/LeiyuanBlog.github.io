---
layout:     post
title:      Docker单机部署服务器压力太大？该学习DockerSwarm集群化部署了
subtitle:   Docker单机部署服务器压力太大？该学习DockerSwarm集群化部署了
date:       2020-04-10
author:     LY
header-img: img/post-docker-bk.png
catalog: true
tags:
    - docker
	- 专栏

---

> 欢迎大家关注我的以下主页，尤其是今日头条！！！谢谢🙏🙏🙏
>
> csdn：[雷园的csdn博客](https://blog.csdn.net/leiyuan2580)
>
> 个人博客：[雷园的个人博客](https://imlcl.store)
>
> 简书：[雷园的简书](https://www.jianshu.com/u/016322e40e1f)
>
> 今日头条：[来自底层程序员的仰望](https://www.toutiao.com/c/user/6132192948/#mid=1616456407686158)

## Docker单机部署服务器压力太大？该学习DockerSwarm集群化部署了

### 依然是习惯性的开头说几句

1. 马上就是除夕啦，现在这里给大家拜个年！新年快乐各位小伙伴。
2. 直到这一章节，我们算是基本讲述完毕了，大部分相关的知识和操作方式我们都已经讲过啦。
3. 上一章节中我们讲述了docker-compose编排工具批量启动容器。
4. 但是在较多时候，单个宿主机无法完全满足我们的性能需求，这个时候我们就会用到Docker Swarm了。

### 正式开始

#### 为什么我们要使用Docker Swarm集群化部署

1. 在多数时候，单个服务器无法满足我们的性能需求。
2. Docker Swarm可以对容器进行动态扩容。

#### 开始前的准备工作

1. 我们需要准备两台（及以上）的服务器或虚拟机。
2. 安装并运行Docker服务。
3. 我们选择其中一台为Leader节点，其他的均为Worker节点。
4. 所有机器均需要开放以下端口：TCP:2377、2376、7946；UDP:7946、4789。这些端口均是Docker Swarm通讯所需节点。

#### 创建Docker Swarm集群

1. 首先我们在选中的Leader节点中进行集群的创建，**--advertise-addr**参数并非必须，若**Leader**拥有多个网卡对应的ip，请使用该参数指定其中之一：

   ```shell
   $ docker swarm init --advertise-addr 192.168.99.100
   # 执行结果如下
   Swarm initialized: current node (dxn1zf6l61qsb1josjja83ngz) is now a manager.
   
   To add a worker to this swarm, run the following command:
   
       docker swarm join --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c 192.168.99.100:2377
   
   To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
   ```

2. 其次我们就需要在选中的Worker节点中执行命令主动加入集群：

   ```shell
   # 该命令包含在Leader创建集群时返回值中
   $ docker swarm join --token SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c 192.168.99.100:2377
   ```

3. 最后呢，我们就需要在Leader节点中检查一下集群状态是否正常：``docker node ls``

4. 这样以来，我们的集群算是部署完毕了，接下来我们简单的启动一些容器，看一下效果。

#### 简单实现

1. 我们需要创建一个跟上一章节类似的yml配置文件，并在其中配置好我们所需服务的相关参数：

   ```yaml
   version: "3"
   services:
     redis:
       image: hub.c.163.com/library/redis:latest
       networks:
         overlay:
       command: redis-server --requirepass leiyuan
     zoo:
       image: zookeeper:latest
       ports:
         - 2181:2181
       restart: always
       networks:
         overlay:
     tomcat:
       image: tomcat
       ports:
         - 80:80
       networks:
         overlay:
       depends_on:
         - zoo
         - redis
       links:
         - redis:redis
         - zoo:zoo
   volumes:
     db-data:
   networks:
     overlay:
   ```

2. 这样我们就可以根据该配置文件启动相关的容器了。

#### 常用的命令

1. 离开集群`docker swarm leave`
2. 离开并删除集群`docker swarm leave --force`
3. 删除指定节点`docker node rm 节点ID`
4. 启动`docker stack deploy -c docker-compose.yml leiyuan`
5. 停止`docker stack down leiyuan`
6. 查看swarm中容器的ip`docker service inspect --format '{{ .Endpoint.VirtualIPs }}'  服务名`
7. 查看集群中的服务`docker service ls`
8. 动态扩容，修改服务数量`docker service scale 服务名=数量`

### 最后说几句

1. 今天呢，就是大年三十了，祝大家新年快乐。希望大家在新的一年里，工作顺顺利利、身体健健康康、家庭和和睦睦、爱情甜甜美美。
2. 我们的章节算是全部都更新完了，很高兴我自己可以坚持下来，在这段时间里，我的收获也是非常丰富，不知道大家是否有所收获呢？可以大家一起加入圈子交流沟通。
3. 希望以后，我们还能一起学习，一起进步！！！