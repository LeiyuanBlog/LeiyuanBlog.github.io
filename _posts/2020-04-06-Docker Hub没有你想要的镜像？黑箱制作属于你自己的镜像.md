---
layout:     post
title:      Docker Hub没有你想要的镜像？黑箱制作属于你自己的镜像
subtitle:   Docker Hub没有你想要的镜像？黑箱制作属于你自己的镜像
date:       2020-04-06
author:     LY
header-img: img/post-docker-bk.png
catalog: true
tags:
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

## Docker Hub没有你想要的镜像？黑箱制作属于你自己的镜像

#### 开头说几句

1. 在前面的章节中，我们分别介绍了**Docker镜像**的使用以及一些基本操作。
2. 在前面我们说的都是使用**官方、开源作者**提供的镜像。
3. 这一章节中，我们就来说说**Docker Images**的制作——**黑箱**。

### 正式开始

##### 为什么自己制作Docker镜像，官方不是很齐全吗？

1. 确实官方的镜像仓库已经很齐全了，而且还有很多第三方的镜像仓库比如网易的镜像仓库。
2. 但是——各方需求千奇百怪，而且根据系统架构的不同，以来服务的方式也有所不同。
3. 所以多数时候，我们还是会使用自己制作的镜像。

##### 为什么使用“黑箱”方式制作，有什么好处吗？

1. 其实这是一种很不规矩的制作方法，如果硬要说有什么好处的话，那就是门槛低、操作简单。
2. 同时无可避免的会有些不好的地方——“体积”大。
3. 同样的镜像，**黑箱**制作的大小可能是**dockerfile**制作的大小**两倍**。
4. 下个章节我们会介绍**dockerfile**的制作方式，相对需要学习和掌握的东西会稍多一些。

##### 开始前的准备工作——找到自己熟悉的操作系统镜像

1. 打开Docker Hub。
2. 左侧**Categories**复选框中，选中**Operating Systems**以筛选操作系统镜像。
3. 我们可以看到右边列出了许多**发行版linux**的系统镜像，有我们常用的**Ubuntu、Centos、Debian**等等。
4. 这里我选择我常用的**Centos**镜像作为我们即将制作镜像的基础镜像。
5. 在**Tags**中，我们可以看到镜像又根据版本分为多个**标签（Tags）**，我直接选择latest版本，大家可以根据自己的需求选择不同标签。
6. 选择好之后，直接复制拉取命令在本地拉取镜像即可。
7. 如果有的小伙伴因为网络问题无法正常拉取，建议大家使用网易的镜像仓库进行拉取，速度会快很多。

##### 对操作系统镜像做一些简单的操作

1. 在操作**linux**系统的时候，我们都会有个习惯，那就是`yum -y update`。

2. 为什么这么做呢？这样是为了避免安装的软件由于依赖服务版本不兼容导致异常。

3. 所以对于我们的系统镜像，也需要执行，我建议将执行过更新操作的镜像保存起来，作为我们后续镜像制作的基础镜像，能和节省很多时间。

4. 具体操作如下：

   ```shell
   # 宿主机操作
   $ docker pull centos
   $ docker -d --name centos centos
   $ docker exec -it centos bash
   # 容器伪终端操作，这步可能需要一点时间哦，耐心等待
   $ yum -y update
   $ exit
   # 宿主机操作
   ## $ docker commit -m "备注信息" -a "更新者" ${容器ID} ${镜像名称:镜像标签}
   $ docker commit -m "centos update" -a "LY" ${容器ID} centos_update:latest
   ## 查看镜像
   $ docker images
   ```

##### Docker黑箱制作Tomcat镜像并通过环境变量配置运行端口

1. 第一步我们运行刚才制作的**centos_update:latest**镜像。

2. 第二步进入容器伪终端并且配置**Java**环境，并且安装**tomcat**。

3. 第三步编写脚本进行字符替换、启动tomcat。

4. 第四步保存容器为镜像，以备后用。

5. 具体操作如下：

   ```shell
   # 第一步
   $ docker run -d -v ./jdk-8u151-linux-x64.tar.gz:/usr/local/java/jdk-8u151-linux-x64.tar.gz -v ./apache-tomcat-9.0.26.tar.gz:/app/software/apache-tomcat-9.0.26.tar.gz --name centos_update centos_update:latest
   # 第二步
   ## 宿主机终端
   $ docker exec -it centos_update bash
   ## 容器伪终端
   $ tar -zxvf /usr/local/java/jdk-8u151-linux-x64.tar.gz -C /usr/local/java/
   $ echo 'export JAVA_HOME=/usr/local/java/jdk1.8.0_151' >> /etc/profile
   $ echo 'export PATH=.:$JAVA_HOME/bin:$PATH'  >> /etc/profile
   $ echo 'export TOMCAT_PORT=8080'  >> /etc/profile
   $ echo 'export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar'  >> /etc/profile
   $ source /etc/profile
   $ tar -zxvf /app/software/apache-tomcat-9.0.26.tar.gz -C /app/software/
   $ mv /app/software/apache-tomcat-9.0.26 /app/software/tomcat
   $ ln -s /app/software/tomcat /root/tomcat
   # 第三步
   $ mkdir -p /app/software/bash
   $ touch /app/software/bash/start.sh
   $ vim /app/software/bash/start.sh
   ## start.sh脚本内容 Start{
         #!/bin/bash
         ## author：来自底层程序员的仰望
         sed -i 's/8080/'${TOMCAT_PORT}'/g' /app/software/tomcat/conf/server.xml
         /app/software/tomcat/bin/startup.sh
         tail -f /app/software/tomcat/logs/catalina.out
   ## }start.sh脚本内容 End
   $ exit
   # 第四步
   $ docker commit -m "my tomcat image" -a "LY" ${容器ID} my_tomcat:latest
   ```

##### 制作的镜像该如何使用？如何指定tomcat运行端口？如何部署项目？

1. 与正常的镜像无异，只需在启动镜像时指定运行我们在镜像内的**start.sh**脚本即可。

2. 若需指定tomcat运行端口，只需添加环境变量配置即可。

3. 至于部署项目，我们只需要将我们的服务包（**war**）在容器启动时挂载至容器中即可，或者启动后copy进去也可以。

4. 具体操作如下：

   ```shell
   $ docker run -d -p 8080:9999 -v ./war/:/app/software/tomcat/webapps/ -e "TOMCAT_PORT=9999" --name tomcat my_tomcat:latest
   ```

### 最后说几句

1. 今天我们简单的说了一下**黑箱**环境下的镜像制作。
2. 虽不是很规矩的做法，但是有些复杂的软件镜像我觉得只能这样来制作。
3. 下一章节中我们会讲解使用**dockerfile**的镜像制作方法。
4. 黑箱的缺点特别的明显，至于有多明显，大家通过下章节的学习后可以使用两者制作的镜像做对比看看。
5. 希望今天的文章可以给大家带来一定的收获，同时感谢大家支持和关注我。
6. 年轻人们，一起努力吧！！！