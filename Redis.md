# Redis

## 1. Redis.conf

启动的时候，可通过配置文件来启动

**位置**:`/etc/redis/6379.conf`

`NETWORK`:网络配置

```
bind 0.0.0.0
protected-mode yes
port 6379
```

`GENERAL`:通用配置

```
#是否以守护进程运行
daemonize yes
#如果以后台进程运行，我们就需要指定一个pid文件
pidfile /var/run/redis_6379.pid
#日志级别
loglevel notice
#日志文件
logfile /var/log/redis_6379.log
#数据库数量,默认是16
databases 16
#
```

`SNAPSHOTTING`:快照,RDB持久化配置

```
#
save 900 1      #如果900s内，至少有一个key进行了修改，就进行持久化操作
save 300 10			
save 60 10000
#持久化如果出错，是否还需要继续工作
stop-writes-on-bgsave-error yes
#是否压缩rdb文件,需要消耗一些cpu资源
rdbcompression yes
#保存rdb文件时，进行错误检查校验
rdbchecksum yes
#rdb文件保存的目录
dir /var/lib/redis/6379
#默认文件名
dbfilename dump.rdb   //根据dir和dbfilename可得出文件位置var/lib/redis/6379/dump.rdb
```

`REPLICATION`:主从配置

```

```

`SECURITY`:安全配置

```
#设置密码
requirepass ""
```

`CLIENTS`:

```
#设置连接redis的最大客户端数量
maxclients 10000
#

```

`MEMORY MANAGEMENT`:

```
#redis配置最大内存容量
maxmemory <bytes>

#内存达到上限的处理策略
maxmemory-policy noeviction
	1.noeviction:用不过期，返回错误

```

`APPEND ONLY MODE`:AOP配置

```
#默认不开启AOF模式,而是使用RDB持久化，在大部分情况下,RDB完全够用
appendonly no
#持久化文件的名字
appendfilename "appendonly.aof"

# appendfsync always   #每次修改都会同步,消耗性能
#每秒执行一次同步
appendfsync everysec
# appendfsync no
```

## 2. 持久化

### 2.1 RDB

保存的文件是`dump.rdb`，新的会覆盖旧的

`dump.rdb`触发条件

1. `SNAPSHOTTING`的`save`满足下
2. 执行`FLUSHALL`命令
3. 退出`redis`

如何恢复rdb文件？

​	只需要将`dump.rdb`放在`redis`启动目录下就可以了,`redis`启动的时候会自动检查该文件,恢复里面的数据

```
#查看redis启动目录
127.0.0.1:6379> config get dir
1) "dir"
2) "/var/lib/redis/6379"
```

