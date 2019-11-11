---
title: docker搭建springcloud环境
author: Juntech
top: false
cover: false
toc: true
mathjax: false
summary: 'docker搭建mysql,redis,rabbitmq,elasticsearch...'
categories: Docker
tags: Docker
keywords: docker,springcloud
abbrlink: ead6358a
date: 2019-08-17 18:42:05
password:
copyright: true
---

## docker环境搭建 

####                       工欲善其事，必先利其器

#### 1、所需环境

​	我们所需环境：win7/8/10,virtualbox,centos7镜像文件

#### 2、所需工具地址：

​	所需工具如下： 

​		xshell: <http://www.ddooo.com/softdown/123749.htm>

​		virtualbox: <https://pc.qq.com/detail/3/detail_1023.html>

​		xftp: <https://pc.qq.com/search.html#!keyword=xftp>

​		Typora: <https://pc.qq.com/detail/1/detail_24041.html>

​		postman: <http://www.downza.cn/soft/205171.html>

或者直接下载资源包：

​		 地址： https://pan.baidu.com/s/1WRCfRvT7MDTi10qzChKzhQ

​		提取码：vs1g

#### 3、搭建centos7虚拟机

​  	搭建centos7虚拟机：默认就行，网络选择NAT模式

​	搭建完成后大致如下：

##### 3.1、启动虚拟机，并更新系统内核

##### 3.1.1、查看内核版本

​	使用 uname -r 命令

##### 3.1.2 使用命令更新系统

​	sudo yum update

##### 3.1.3 设置yum源

​	sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

![img](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128094640209-1433322312.png)

##### 3.1.4查看docker版本 


yum list docker-ce --showduplicates | sort -r


![img](https://images2017.cnblogs.com/blog/1107037/201801/1107037-20180128095038600-772177322.png)

##### 3.1.5 安装docker


sudo yum install docker-ce


默认安装最新版stable

#####  3.1.6 启动并加入开机启动


$ sudo systemctl start docker
$ sudo systemctl enable docker


##### 3.1.7 验证是否安装成功


$ docker version




#### 4、安装springcloud微服务所需组件

#####  4.1.1 **首先获取rabbit镜像：**

​	docker pull rabbitmq:management

#####  4.1.2运行容器

​	docker run -d --hostname my-rabbit --name rabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management 

​	其中，15672：控制台端口号5672：应用访问端口号


--hostname：指定容器主机名称
--name:指定容器名称
-p:将mq端口号映射到本地


查看rabbit运行状况：


docker logs rabbit


容器运行正常，使用http://server_ip:15672可以访问rabbit控制台

##### 4.1.3 查看docker容器内运行的容器

使用docker ps    -------------> 目前正在运行的容器

docker ps - a     ---------------> 注册了的容器

docker iamges    -------------->所有下载了的容器

docker exec -it ... -------------->到目录下运行容器

docker run -d imageId -p port  ------>运行docker 容器

#### 4.2.1获取redis
	docker pull redis

#####  4.2.2创建目录

​	2.1 配置文件目录  mkdir -p /root/docker/redis/conf

​	2.2 数据目录  mkdir -p /root/docker/redis/data

##### 4.2.3 启动容器，加载配置文件并持久化数据

docker run -d --privileged=true -p 6379:6379 --restart always -v /root/docker/redis/conf/redis.conf:/etc/redis/redis.conf -v /root/docker/redis/data:/data --name myredis redis redis-server /etc/redis/redis.conf --appendonly yes

##### 4.2.4 涉及到的命令行参数


-d                                                  -> 以守护进程的方式启动容器
-p 6379:6379                                        -> 绑定宿主机端口
--name myredis                                      -> 指定容器名称
--restart always                                    -> 开机启动
--privileged=true                                   -> 提升容器内权限
-v /root/docker/redis/conf:/etc/redis/redis.conf    -> 映射配置文件
-v /root/docker/redis/data:/data                    -> 映射数据目录
--appendonly yes                                    -> 开启数据持久化


#### 4.3.1 elasticsearch

docker search elasticsearch

docker pull 一个镜像

docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.0.1


至此便可以在浏览器中通过9200端口访问到es了。

如果显示有跨域问题，则需要另外进行配置：


执行docker exec -it es bash。以交互模式进入容器

es的容器带有vi指令，所以可以直接执行 vi config/elasticsearch.yml

加入跨域配置

http.cors.enabled: true
http.cors.allow-origin: "*"


保存修改后重启容器即可。

docker restart es


#### 4.4.1mysql

参考菜鸟教程：<https://www.runoob.com/docker/docker-install-mysql.html>

#### 4.5.1 Nginx

参考菜鸟教程： <https://www.runoob.com/docker/docker-install-nginx.html

#### 4.6 连接xshell

由于安装的时候是最小安装，则使用ip addr 显示虚拟机的ip，![56593890562](C:\Users\Ryan\AppData\Local\Temp\1565938905627.png)

记住ip

打开xshell，填上ip和用户密码

![56593896282](C:\Users\Ryan\AppData\Local\Temp\1565938962821.png)

名称随便填写，主机填写刚才获取到的ip端口默认22

![56593900564](C:\Users\Ryan\AppData\Local\Temp\1565939005643.png)

用户填写root,密码：你设置的密码

![56593904286](C:\Users\Ryan\AppData\Local\Temp\1565939042869.png)

显示root@localhost则连接成功！enjoy!

#### 4.7 致谢

至此，本教程就结束了，谢谢大家的阅读！