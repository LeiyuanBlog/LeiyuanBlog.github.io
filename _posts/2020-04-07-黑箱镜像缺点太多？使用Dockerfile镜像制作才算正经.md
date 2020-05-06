---
layout:     post
title:      黑箱镜像缺点太多？使用Dockerfile镜像制作才算正经
subtitle:   黑箱镜像缺点太多？使用Dockerfile镜像制作才算正经
date:       2020-04-07
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

## 黑箱镜像缺点太多？使用Dockerfile镜像制作才算正经

### 开篇说几句

1. 上一章节我们说了镜像的黑箱制作方式。
2. 黑箱制作确实简单，但是步骤却也很是复杂，且制作出来的镜像体积有些大。
3. 这一章节我们就来说说使用Dockerfile制作我们需要的镜像。
4. 那～在这里感谢各位长久以来的关注及阅读，大家如果有任何的疑虑，可以在评论区写明，我一定会一一回复！
5. 当然也可以私信与我交流。
6. 今日刚创建了圈子，已经与专栏关联，非常欢迎大家加入，互相交流沟通。

### 正式开始

##### 为什么使用Dockerfile制作镜像

1. 通过上一篇的介绍以及大家的自我实践，相信大家可以看出来黑箱制作的镜像缺点确实比较明显，最主要的是体积比较大，而且运行起来速度也会有所受限。
2. 使用**Dockerfile**制作我们的思路也会更加清晰，做起来更加有条理。
3. 使用**Dockerfile**制作的镜像体积更小。
4. 只要有相关文件以及**Dockerfile**在，任何机器都可以构建镜像。

##### 利用Dockerfile制作Docker Image前的准备工作

1. 不可避免要有一台安装了**Docker**的终端，可以是Mac OS、Linux、Windows，这些个玩意都可以。
2. 其次要有一个编辑工具，我一般都是直接使用**IDEA**打开编辑**Dockerfile**文件，这样格式明确利于编辑。
3. 再然后呢就是整理一下安装步骤，即为需要操作的步骤有哪些，梳理清楚避免因为编辑错误导致**Docker image**构建失败。

##### Dockerfile常用命令

1. FROM：基础镜像
2. MAINTAINER：备注信息
3. WORKDIR：作业目录
4. ADD：添加文件至镜像
5. COPY：复制本地文件至镜像
6. RUN：执行shell命令
7. ENV：定义环境变量
8. EXPOSE：开放端口信息
9. CMD：启动命令或运行脚本
10. 这里简单说一下ENV（环境变量），一般来说我们会将原有变量名称全部**大写**并添加前缀表明环境变量的作用范围，例如tomcat运行端口为**port=8080**，那环境变量中我们一般以**TOMCAT_PORT=8080**的形式来表示。

##### Dockerfile编辑

1. 一般来说，Dockerfile的编辑我们分为以下几个步骤。

2. 第一步引用基础。

3. 第二步将所需用到的外部文件COPY或者ADD至镜像内部，两者略有些区别。COPY可以从宿主机复制文件至镜像中。ADD可以直接使用链接下载文件至镜像中。

4. 第三步编辑操作步骤。

5. 第四步添加启动命令，这里我一般添加启动脚本并维持前端运行，否则脚本执行完回自动停止Docker容器的运行。

6. 当然并不只有这些，如果我们镜像中有使用环境变量，则我们需要在Dockerfile中给出环境变量的初始值，避免启动镜像时没有制定环境变量导致启动失败。

7. 还有如果需要对外开放端口时，我们需要规定容器的开放端口以便用于宿主机端口映射。

8. 包含**ffmpeg**转码工具以及**tomcat**的Docker镜像示例**Dockerfile**如下：

   ```dockerfile
   FROM hub.c.163.com/public/centos:7.2-tools
   MAINTAINER Leiyuan
   COPY ffmpeg.tar.gz /usr/local/src/
   COPY startup.sh /root/
   COPY server.xml /root/
   RUN curl http://mirrors.aliyun.com/repo/Centos-7.repo | grep -v 'mirrors.aliyuncs.com' > /etc/yum.repos.d/CentOS-Base.repo \
       &&curl http://mirrors.aliyun.com/repo/epel-7.repo  | grep -v 'mirrors.aliyuncs.com' > /etc/yum.repos.d/epel-7.repo \
       &&yum clean all \
       &&yum makecache \
       &&yum -y update \
       &&yum -y install zlib zlib-devel pcre pcre-devel gcc gcc-c++ openssl openssl-devel libevent libevent-devel perl \
       &&cd /usr/local/src \
       &&tar -zxvf ffmpeg.tar.gz \
       &&rm -rf ffmpeg.tar.gz \
       &&yum -y install java-1.8.0-openjdk* \
       &&yum -y install gcc automake autoconf libtool make \
       &&sh ffmpeg/install_ffmpeg.sh \
       &&rm -rf ffmpeg \
       &&yum -y install wget \
       &&cd /root/ \
       &&wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.21/bin/apache-tomcat-9.0.21.tar.gz \
       &&tar -zxvf apache-tomcat-9.0.21.tar.gz \
       &&rm -rf /root/apache-tomcat-9.0.21.tar.gz \
       &&mv apache-tomcat-9.0.21 tomcat \
       &&\cp /root/server.xml /root/tomcat/conf
   ENV TOMCAT_SERVER_PORT=8080 MQTT_SERVER_IP=127.0.0.1 MQTT_SERVER_PORT=1883 REDIS_SERVER_PORT=6379 REDIS_SERVER_IP=127.0.0.1 REDIS_SERVER_PASSWORD=admin MK_MYSQL_IP=127.0.0.1 MK_MYSQL_PORT=3306 MK_MYSQL_USER=root MK_MYSQL_PASS=root MK_ZOO_IP=127.0.0.1 MK_ZOO_PORT=2181 MK_FDFS_TRACKER_IP=127.0.0.1 MK_FDFS_TRACKER_PORT=22122 MK_FDFS_DOWN_IP=127.0.0.1 MK_FDFS_DOWN_PORT=8888
   CMD ["bash", "/root/startup.sh"]
   ```

##### 关于Docker image的其他问题

1. 在镜像启动后，一定要维持运行状态，否则在执行完启动命令后容器会自动停止。
2. 所以我在编写tomcat镜像的启动脚本时，会加入`tail -f catalina.out`命令以保持容器的运行状态。

### 最后说几句

1. 这一章节中我们说了**dockerfile**的制作方式，大家可以自行尝试做一些简单的镜像，与黑箱制作方式进行比较，看看两者有什么区别。
2. 如果大家有什么想法想与我沟通，可以通过评论或者私信，我会一一回复。
3. 新开通的圈子希望大家可以踊跃的参加并发表自己的动态哦！！！
4. 下一章节中，我们会介绍环境变量的具体使用方式以此实现动态配置。
5. 谢谢大家的持续关注和支持！！！