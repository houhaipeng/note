# rabbitMQ



## 1. 基本概念

![5015984-367dd717d89ae5db](../../Pictures/RabbitMQ/5015984-367dd717d89ae5db.png)

1. Message:消息，由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key(路由键)，priority(相对于其他消息的优先权)，delivery-mode（指出该消息可能需要持久性存储）等。
2. Publisher:消息的生产者，也是一个向交换器发布消息的客户端应用程序
3. Exchange:交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列，Exchange有4种类型：direct(默认),fanout,topic和headers。
4. Bingding:绑定，用于消息队列和交换器之间的关联，可以是多对多的关系
5. Queue:消息队列，用于保存消息直到发送给消费者，它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里，等待消费者连接到这个队列将其取走
6. Connection:网络连接，比如一个TCP连接
7. Channel:信道

## 2. 常见命令

已在虚拟机,`192.168.0.108`,`192.168.0.109`,`192.168.0.110`上安装了RabbitMQ,并设置了开机自启动，访问路径为

```
http://192.168.0.110:15672/
```

RabbitMQ官网

```
https://www.rabbitmq.com/
```

## 3. 常见模型

1. `"Hello World"`

![python-one](../../Pictures/RabbitMQ/python-one.png)

```

```

2. `Work queues`

![python-two](../../Pictures/RabbitMQ/python-two.png)



3. `Publish/Subscribe`

![python-three](../../Pictures/RabbitMQ/python-three.png)

交换机类型：`fanout`

4. `Routing`

![python-four](../../Pictures/RabbitMQ/python-four.png)

交换机类型：`direct`

- 队列与交换机绑定，不能是任意绑定了，而是指定一个`RoutingKey`(路由key)
- 消息的发送方在向Exchange发送消息时，也必须指定消息的`RoutingKey`
- Exchange不在把消息交给每一个绑定的队列，而是根据消息的`RoutingKey`进行判断，只有队列的`RoutingKey`与消息的`RoutingKey`完全一致，才会接收到消息

5. `Topics`

![python-five](../../Pictures/RabbitMQ/python-five.png)

交换机类型：`topic`

- `*`**(star) can substitute for exactly one word**.匹配一个单词
- `#` **(hash) can substitute for zero or more words.**匹配0或多个单词

## 4. RabbitMQ集群

### 4.1 集群架构

#### 4.1.1 普通集群（副本集群）

> All data/state required for the operation of a RabbitMQ broker is replicated across all nodes. An exception to this are message queues, which by default reside on one node, though they are visible and reachable from all nodes. To replicate queues across nodes in a cluster,use a queue type that supports replication. 

**只能同步交换机，无法同步队列**

默认情况下：RabbitMQ代理操作所需的所有数据/状态都将跨所有节点复制。这方面的一个例外是消息队列。

核心解决问题：`当集群中某一时刻master节点宕机，可以对Queue中的信息，进行备份。`

```

```

#### 4.1.2 镜像集群

**可以同步交换机和队列**

镜像队列机制就是将队列在三个节点之间设置主从关系，消息会在三个节点之间进行自动同步，且如果其中一个节点不可用，并不会导致消息丢失或服务不可用的情况，提升MQ集群的整体高可用性。

## 5. 消息的可靠性

### 5.1 丢失数据的场景及防止策略

1. 生产者弄丢了数据:
	- 开启RabbitMQ事务（同步，不推荐）
	- 开启confirm模式（异步，推荐）
2. RabbitMQ弄丢了数据
	- 开启RabbitMQ持久化
3. 消费者弄丢了数据
	- 关闭RabbitMQ自动ACK,在程序中手动调用ack

