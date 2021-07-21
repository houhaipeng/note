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
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/8db4685607912b43afdc15fc8688d64e248aba132d73c7731631e37064c30c16/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/8db4685607912b43afdc15fc8688d64e248aba132d73c7731631e37064c30c16/hostname",
        "HostsPath": "/var/lib/docker/containers/8db4685607912b43afdc15fc8688d64e248aba132d73c7731631e37064c30c16/hosts",
        "LogPath": "",
        "Name": "/vigilant_chandrasekhar",
        "RestartCount": 0,
        "Driver": "overlay2",
        "MountLabel": "system_u:object_r:svirt_sandbox_file_t:s0:c476,c817",
        "ProcessLabel": "system_u:system_r:svirt_lxc_net_t:s0:c476,c817",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "journald",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "docker-runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Name": "overlay2",
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/4cfea1135f111e8dc556d7efd6707f423ec9a26216739e01d12ce9ca0ce1c7ea-init/diff:/var/lib/docker/overlay2/f6f87eb760fec031b779276aac476244b58dab8812c1a1564ca0586b3a399c16/diff",
                "MergedDir": "/var/lib/docker/overlay2/4cfea1135f111e8dc556d7efd6707f423ec9a26216739e01d12ce9ca0ce1c7ea/merged",
                "UpperDir": "/var/lib/docker/overlay2/4cfea1135f111e8dc556d7efd6707f423ec9a26216739e01d12ce9ca0ce1c7ea/diff",
                "WorkDir": "/var/lib/docker/overlay2/4cfea1135f111e8dc556d7efd6707f423ec9a26216739e01d12ce9ca0ce1c7ea/work"
            }
        },
        "Mounts": [],
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

## 10. Docker网络

