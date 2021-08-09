# Docker

## 1. Docker的基本组成

**仓库(repository)**:存放镜像的地方，仓库分为公有仓库(Docker Hub,阿里云)和私有仓库

**镜像(image)**:

**容器(container)**:可以把这个容器理解为一个简易的linux系统

## 2. Docker底层原理

**Docker是怎么工作的？**

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上。通过Socket从客户端访问

DockerServer接收到Docker-Client的指令，就会执行这个命令.

## 3. Docker的常用命令

### 3.1 镜像的基本命令

### 3.2 容器的基本命令

`run`:

```
docker run [可选参数] image

# 参数说明
--name="Name" #容器名字，用来区分容器
-d            #后台方式运行
-it						#使用交互方式运行，进入容器查看内容
-p 						#指定容器的端口 
	-p ip:主机端口:容器端口
	-p 主机端口:容器端口(常用)
	-p 容器端口
	容器端口
-P						#随机指定端口
--privileged=true  #权限访问受限时加上

# 例子
[root@localhost ~]# docker run -it centos /bin/bash
[root@2c3c92e6329f /]# ls 
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 退出容器
exit					#容器停止并退出
Ctrl+P+Q      #容器不停止退出

```

`ps`:列出正在运行的容器

```
docker ps [可选参数]  #当前正在运行的程序

# 可选参数
-a 						#当前正在运行的程序+历史运行过的容器
-n=?          #显示最近创建的n个容器
-q						#只显示容器的编号
```

`rm`:删除容器

```
docker rm 容器id  #不能删除正在运行的容器，如果要强制删除 rm -f

# 例子
docker rm -f $(docker ps -aq)  #删除所有容器
		docker ps -a -q|xargs docker rm  #删除所有的容器
```

**启动和停止容器的操作**

``` 
docker start 容器id        # 启动容器
docker restart 容器id      # 重启容器
docker stop 容器id				 # 停止当前正在运行的容器
docker kill 容器id				 # 强制停止当前容器
```

### 3.3 常用其他命令(重要)

```
docker run -d image
#问题：docker ps后发现容器停止了
#解释：docker容器使用后台运行，就必须要有一个前台进程,docker发现没有应用，就会自动停止
```

`logs`:查看日志

```
docker logs  -f -t --tail 容器id

# 自己编写一段shell脚本
docker run -d centos /bin/sh -c "while true;do echo hhp;sleep 1;done"
```

`top`:查看容器中进程信息

```
docker top 容器id
```

`inspect`:查看镜像元数据

```
# 命令
docker inspect 容器id

# 测试
[
    {
        "Id": "8db4685607912b43afdc15fc8688d64e248aba132d73c7731631e37064c30c16",
        "Created": "2021-07-18T08:01:48.463634657Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 6585,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-07-18T08:01:48.713502327Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "LogPath": "",
        "Name": "/vigilant_chandrasekhar",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Config": {
            "Hostname": "8db468560791",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "2c1557a62081dac4fa8b279817d07a892c59640774c1b094aef07c419292c26c",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/2c1557a62081",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "b693140eee5e5b0b899472436ff4333ff011be577f41e61df7292ed15b4b06a4",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "2534fc1e738356abd53d1ed4193ca3b81393b39574bbc6592add69c4c5686411",
                    "EndpointID": "b693140eee5e5b0b899472436ff4333ff011be577f41e61df7292ed15b4b06a4",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
        }
    }
]
```

`exec`:进入当前正在运行的容器

```
#场景：我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 命令
# 方式一：
docker exec -it 容器id bashShell #进入容器后开启一个新的终端，可以在里面操作(常用)

#例子
docker exec -it 8db468560791 /bin/bash

#方式二：
docker attach 容器id  #进入容器正在执行的终端，不会启动新的进程


```

**从容器内拷贝文件到主机上**:`cp`

```
docker cp 容器id:容器内路径  目的主机路径

#拷贝是一个手动过程，未来我们使用 -v卷的技术，可以实现，自动同步
```

`network`:网络相关

```
docker network --option
[root@localhost ~]# docker network --help
Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks     #列举网络
  prune       Remove all unused networks
  rm          Remove one or more networks //移除网络
```

### 3.4 命令小结

![docker命令](../../Pictures/Docker/docker命令.jpeg)

## 4. 部署Nginx

```
//搜索镜像
docker search nginx
//下载镜像
docker pull nginx
# -d:后台运行 --name:给容器起名字 -p:映射从宿主机（3344）到容器的端口（80）
docker run -d --name nginx01 -p 3344:80 nginx
//查看ngnix
docker ps
//访问nginx
curl localhost:3344
//进入容器
docker exec -it nginx01 /bin/bash

# 问题：我们每次改动ngnix配置文件，都需要进入容器内部，十分麻烦，如果可以在容器外部提供一个映射路径，达到在容器外部修改文件，容器内部就可以自动修改--数据卷
```

## 5. 部署Tomcat

```
//我们之前的启动都是后台，停止了容器之后，容器还是可以查到
//docker run -it --rm 用完即删
//docker run -it tomcat:9.0
#
docker run -d -p 3355:8080 --name tomcat01 tomcat
#进入容器
docker exec -it tomcat01 /bin/bash

#问题：1.linux命令少了，2.没有webapps.
#解释：默认是最小的镜像，所有不必要的都剔除了，保证最小可运行的环境
#解决方法：将webapps.dist下的文件拷贝到webapps文件夹下
cp -r webapps.dist/* webapps //-r是递归拷贝
#在地址栏访问192.168.0.108:3355
```

## 6. 部署ES

```
#--net somenetwork ? 网络配置
# 下载启动
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
# 访问es
[root@localhost ~]# curl localhost:9200
{
  "name" : "e27c88344d05",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "h0ZVw37uRNqKDDP9GKLBmQ",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ef48eb35cf30adf4db14086e8aabd07ef6fb113f",
    "build_date" : "2020-03-26T06:34:37.794943Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
# 查看内存占用情况
docker stats
# 修改配置，增加内存限制
# -e 环境配置修改
docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx521m" elasticsearch:7.6.2
# 查看内存
docker stats 10523c333a7f
```

## 7. 镜像原理

**所有应用，直接打包docker镜像，就可以直接跑起来**

如何得到镜像：

1. 从远程仓库下载
2. 朋友拷贝给你
3. 自己制作一个镜像DockerFile

### 7.1 Docker镜像加载原理

>UnionFS(联合文件系统)

### 7.2 分层理解

**特点**：

```
#
docker image inspect redis:latest

"Layers": [
	"sha256:764055ebc9a7a290b64d17cf9ea550f1099c202d83795aa967428ebdf335c9f7",
	"sha256:245c9d23f65373415922e53424032cabe7b282c5cf8f9f8070a7d1830fca6871",
	"sha256:ebef6caacb966ed54c0c3facf2288fa5124452f2c0a17faa1941625eab0ceb54",
	"sha256:0b7b774038f08ec329e4dd2c0be440c487cfb003a05fce87cd5d1497b602f2c1",
	"sha256:a71d36a87572d637aa446110faf8abb4ea74f028d0e0737f2ff2b983ef23abf3",
	"sha256:9e1fddfb3a22146392a2d6491e1af2f087da5e6551849a6174fa23051ef8a38f"
]
```

### 7.3 commit镜像

```
#提交容器成为一个新的副本，类似git
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

以tomcat为例测试:

```
#
docker run -it -p 8080:8080 tomcat
#进入tomcat目录，拷贝webapps.dist文件夹下的所有文件到webapps中
docker exec -it f84c02a82396 /bin/bash
cp -r webapps.dist/* webapps
#将修改的容器作为一个新的镜像提交
docker commit -a="hhp" -m="add webapps" f84c02a82396 tomcat01:1.0
#查看镜像
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
tomcat01            1.0                 921770ee91e4        17 seconds ago      672 MB
```

## 8. 容器数据卷

### 8.1 概念

**docker的理念**：将应用和环境打包成一个镜像

**问题**：数据？假设数据都在容器中，如果我们删除容器，数据就会丢失

**需求**：数据可以持久化

### 8.2 数据卷的使用

>方式一：直接使用命令挂载  -v

```
docker run -it -v 主机目录：容器内目录
```

```
 //挂载
 docker run -it -v /home/hhp_docker_test_01:/home centos /bin/bash
 #查看挂载情况
 docker inspect 81b3c505bcb6
 "Mounts": [
            {
                "Type": "bind",
                //主机内地址
                "Source": "/home/hhp_docker_test_01",
                //docker容器内的地址
                "Destination": "/home",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ]
```

**安装MySQL示例**

```
#下载镜像
docker pull mysql
#运行容器,
# -e 环境配置
# -v 卷挂载
# --name 容器名
docker run -d -p 3307:3306 -v /home/mysql/conf:/etc/mysql/conf.d 
-v /home/mysql/data:/var/lib/mysql --privileged=true 
-e MYSQL_ROOT_PASSWORD=hhp123 --name mysql01 mysql:5.7
```

**即使将容器删除，同步的数据也不会丢失**，这就实现了容器持久化功能

### 8.3 具名挂载和匿名挂载

```
# 匿名挂载
-v 容器内路径
# 例子
docker run -d -P --name nginx01 -v /etc/ngnix nginx
# 查看所有的volume的情况
docker volume ls
# 
DRIVER              VOLUME NAME
local               b002d6731edf1879a3754568496421799c24cf72ddce7511fa146db5facda31e


# 具名挂载
-v 卷名:容器内路径
# 例子
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
# 
docker volume ls
# 
DRIVER              VOLUME NAME
local               juming-nginx
# 查看一下这个卷
docker volume inspect juming-nginx
[
    {
        "Driver": "local",
        "Labels": null,
        //这是宿主机的路径
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": {},
        "Scope": "local"
    }
]
```

**所有docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxx/_data`**;

我们通过具名挂载可以方便的找到我们的卷，大多数情况下使用`具名挂载`

```
-v 容器内路径              #匿名挂载
-v 卷名:容器内路径         #具名挂载
-v /宿主机路径:容器内路径.  #指定路径挂载
```

**拓展**

```
# -v 容器内路径:ro(rw)改变读写权限
ro readonly  #只读
rw readwrite #读写
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

#ro：这个路径只能通过宿主机改变,容器内部无法操作。默认rw
```

## 9. DockerFile

dockerfile文件目录

`192.168.21.212`:`/home/docker_hhp`

`192.168.0.108`:

### 9.1 概念

```
# 创建一个dockerfile文件
# 文件内容 
# 指令(大写):参数,即镜像的一层
FROM centos
VOLUME ["volume01", "volume02"]  #匿名挂载
CMD echo "-------------end--------------"
CMD /bin/bash
# 通过dockerfile文件构建镜像
docker build -f dockerfile1 -t hhp/centos:1.0 .
# 查看镜像
[root@localhost hhp_docker_test_volume]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hhp/centos          1.0                 e7296d566925        28 seconds ago      209 MB
```

### 9.2 数据卷容器

```
# 运行容器docker01
docker run -it --name docker01 hhp/centos:1.0
# 容器docker02挂载容器docker01上，--volumes-from实现容器间的数据共享
docker run -it --name docker02 --volumes-from docker01 hhp/centos:1.0
# 通过inspect可以看出，两个容器挂载在同一个目录下
```

### 9.3 DockerFile构建过程

```
#dockerfile构建镜像的步骤
1. 编写一个dockerfile文件
2. docker build构建成为一个镜像
3. docker run 运行镜像
4. docker push发布镜像(DockerHub,阿里云镜像仓库)
```

`dockerfile指令集`

```
#dockerfile指令集
FROM:指定基础镜像
MAINTAINER:指定维护者信息
RUN:镜像构建的时候需要运行的命令
ADD:
WORKDIR:镜像的工作目录
VOLUME:挂载的目录
EXPOSE:暴露端口
CMD/ENTRYPOINT:指定这个容器启动的时候要运行的命令(CMD只有最后一个会生效,ENTRYPOINT追加命令)
ONBUILD:
COPY:类似ADD,将文件拷贝到镜像中.
ENV:构建的时候设置环境变量
```

### 9.4 实战1：构建自己的centos

```
#1. 编写配置文件,取名mydockerfile
FROM centos
MAINTAINER hhp<1439801553@qq.com>

ENV MYPATH /usr/local
#docker run的时候，直接进入这个目录
#之前的centos默认是根目录(/)
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "------end-----"
CMD /bin/bash

--------------------------
#2. 通过文件构建镜像,取名mycentos:0.1
docker build -f mydockerfile -t mycentos:0.1 .
--------------------------
#3. 测试运行
docker run -it mycentos:0.1
--------------------------
#4. 查看镜像构建历史,我们平时拿到一个镜像，可以研究一下他是怎么构造的
docker history d894122c7b25
--------------------------
```

### 9.5 实战2：构建tomcat

```
#1.准备：镜像文件tomcat压缩包，jdk压缩包
#2.编写dockerfile文件，官方默认命名Dockerfile,build会自动寻找这个文件，就不需要-f指定了
FROM centos
MAINTAINER hhp<1439801553@qq.com>

COPY /home/test.java /usr/local/test.java
ADD jdk-8u281-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.55.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_281
ENV CLASSPATH /usr/local/java/jdk1.8.0_281/lib/
ENV TOMCAT_HOME /usr/local/apache-tomcat-8.5.55
ENV PATH $PATH:$JAVA_HOME/bin:$TOMCAT_HOME

EXPOSE 8080

CMD /usr/local/apache-tomcat-8.5.55/bin/startup.sh && tail -f /usr/local/apache-tomcat-8.5.55/logs/catalina.out
---------------------------------以上是dockerfile内容----------------------------------
#3.构建镜像,取名为mytomcat
docker build -t mytomcat .
#4.运行容器，容器名为hhptomcat，对应镜像为mytomcat
docker run -it -p 9090:8080 --name hhptomcat -v /root/tomcat_rm/build/tomcat/test:/usr/local/apache-tomcat-8.5.55/webapps/test -v /root/tomcat_rm/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-8.5.55/logs --privileged=true mytomcat
#5.
```

发布项目:在`/root/tomcat_rm/build/tomcat/test/WEB-INF/web.xml`下写入如下内容

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
</web-app>
```

在`/root/tomcat_rm/build/tomcat/test/index.jsp`下写入如下内容

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>hello</title>
</head>
<body>
Hello World!<br/>
<%
out.println("你的 IP 地址 " + request.getRemoteAddr());
%>
</body>
</html>
```

访问：`192.168.0.108:9090/test`即可看到index.jsp页面

### 9.6 发布镜像到仓库

`DockerHub`

```
# 1.登录DockerHub
docker login -u 18340824089
# 2.改名，
docker tag c751032256be docker.io/18340824089/hhp/tomcat:1.0
# 2.提交镜像
docker push 18340824089/hhp/tomcat:1.0
```

`阿里云`

登录阿里云，找到`容器镜像服务`->`个人实例`->`创建命令空间`->`创建镜像仓库`

## 10. Docker网络

```
#获取ip地址
ip addr
```

### 10.1 Docker0

```
# 启动容器
docker run -d -P --name tomcat01 tomcat
# 查看容器的内部网络地址, ip addr
[root@localhost ~]# docker exec -it tomcat01 ip addr
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link 
       valid_lft forever preferred_lft forever
# 查看linux的网络,多了一个网络地址
[root@localhost ~]# ip addr
7: veth07a45c9@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 86:18:c6:ed:23:75 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::8418:c6ff:feed:2375/64 scope link 
       valid_lft forever preferred_lft forever
# 从上面可以看出linux和容器的网络在容器启动时是一对一对出现的
# 查看linux(192.168.0.108)和容器内(172.17.0.2)的网络是否能ping通
ping 172.17.0.2 //linux可以ping通容器内部
```

我们每启动一个docker容器，docker就会给docker容器分配一个ip,只要安装了docker,就会有一个网卡docker0(路由器ip192.168.0.1)，桥接模式，使用的技术是`evth-pair`技术

```
# 再启动一个容器
docker run -d -P --name tomcat02 tomcat
# 
[root@localhost ~]# docker exec -it tomcat02 ip addr
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.3/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:3/64 scope link 
       valid_lft forever preferred_lft forever
# 查看两个容器是否能ping通
docker exec -it tomcat02 ping 172.17.0.2 //也能ping通
# 但是通过容器名无法ping通
docker exec -it tomcat02 ping tomcat01
# 解决,--link 要连接的容器名.
docker run -d -P --name tomcat03 --link tomcat02 tomcat
docker exec -it tomcat02 ping tomcat03
# 但是反向ping不通
docker exec -it tomcat03 ping tomcat02
# 列出docker的网络
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e470750a85c0        bridge              bridge              local
1031d61ab17c        host                host                local
091ca26b986c        none                null                local
# tomcat03查看hosts配置
[root@localhost ~]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      tomcat02 4c28076e1fbf
172.17.0.4      a10e99711a94
# --link就是在hosts配置中增加了tomcat02的映射，所以tomcat03 ping tomcat02能ping通
# 现在已经不建议使用--link了

```

结论：容器和容器之间是可以互相ping通的

`6: eth0@if7(容器1)`->`7: veth07a45c9@if6(linux)`->`9: vethcb0b891@if8(linux)`->`8: eth0@if9(容器2)`

所有的容器在不指定网络`--net`的情况下,都是docker0路由的，

```
172.17.0.0/16 255*255个地址(172.17.x.x)
172.17.0.0/8  255个地址(172.17.0.x)
.....
```

docker0在linux中.

![5e89c3d8d6f28773fabb1138b860d01f](../../Pictures/Docker/5e89c3d8d6f28773fabb1138b860d01f.jpeg)

### 10.2 自定义网络

**docker0的问题**：不支持容器名连接访问

**网络模式**

1. `bridge`:桥接模式 docker（默认,自己创建的也使用这个模式）
2. `none`:不配置网络
3. `host`:和宿主机共享网络
4. `container`:容器内网络连通

```
# 我们直接启动的命令，默认是加了--net bridge这个参数的
docker run -d -P --name tomcat01 tomcat
# 创建自定义网络
//gateway路由器地址，网络从这里出去
//subnet 192.168.0.0/16,  192.168.0.2 - 192.168.255.255
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
# 查看创建的网络，mynet
[root@localhost ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
eb656a2075c7        bridge              bridge              local
1031d61ab17c        host                host                local
f943fdbfea5a        mynet               bridge              local
091ca26b986c        none                null                local
#
docker network inspect mynet
# 使用自己的网络启动容器
docker run -d -P --name tomcat-net-01 --net mynet tomcat
docker run -d -P --name tomcat-net-02 --net mynet tomcat
# 此时用下面命令查看，有了上述两个容器的ip
docker network inspect mynet
# 自定义网络,可以用容器名ping通两个容器，不用再使用--link了
docker exec -it tomcat-net-01 ping tomcat-net-02
docker exec -it tomcat-net-02 ping tomcat-net-01
```



## 11 可视化

### 11.1 portainer

Docker图形化界面管理工具!提供一个后台面板供我们操作

```

```

### 11.2 rancher

## 12 综合应用

### 12.1 Redis集群部署实战

```
#
docker network create redis --subnet 
```



### 12.2 Springboot微服务打包Docker镜像

```
#1. 构建springboot项目
#2. 打包应用
#3. 编写dockerfile
#4. 构建镜像
#5. 发布运行
```

