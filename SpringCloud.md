# SpringCloud

Spring官网

```
https://spring.io/
```

## 1. Hystrix

### **服务熔断**

​	熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，进而熔断该节点微服务的调用，快速返回错误的响应信息。**当检测到该节点微服务调用响应正常后，恢复调用链路。**

​	在SpringCloud框架中，熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。**熔断机制的注解是@HystrixCommand**

![熔断](/Users/haipenghou/Pictures/state.png)

**熔断类型**

- 熔断打开：
- 熔断关闭：
- 熔断半开：

**涉及到断路器的三个重要参数**

1. 快照时间窗：
2. 请求总数阈值：默认10秒内超过20个请求次数
3. 错误百分比：默认10秒内超过50%的请求失败

---

### **服务降级**

```
@DefaultProperties(defaultFallback = "xxx")
```

​	1. 1:1每个方法配置一个服务降级方法

	2. 1:N除个别重要核心业务有专属，其他普通的可以通过**@DefaultProperties(defaultFallback = "")**统一跳转到统一处理结果页面。

未来我们要面对的异常

- 运行
- 超时
- 宕机

---

## 2. Nacos

```
下载地址：http://nacos.io
				https://github.com/alibaba/Nacos
```

安装到本机的位置

```
/Users/haipenghou/software/nacos
```

启动服务器(standalone代表着单机模式运行，非集群模式)

```
//进入到nacos的bin目录
sh startup.sh -m standalone
```

访问服务器

```
http://localhost:8848/nacos
```

关闭服务器

```
sh shutdown.sh
```

---

Nacos支持AP和CP模式的切换：**C是所有节点在同一时间看到的数据是一致的；而A的定义是所有的请求都会收到响应。**

Nacos同SpringCloud-config一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。

springboot中配置文件的加载是存在优先级顺序的，**bootstrap优先级高于application**,全局的配置(共性)放在bootstrap.yml,各自的配置放在application.yml

在 Nacos Spring Cloud 中，**dataId** 的完整格式如下：

```plain
${prefix}-${spring.profiles.active}.${file-extension}
```

-  `prefix`默认为`spring.application.name`的值，也可以通过配置项`spring.cloud.nacos.config.prefix`来配置
- `spring.profiles.active`即为当前环境对应的profile,**注意：当`spring.profiles.active`为空时，对应的连接符`-`也将不存在，dataI的拼接格式变成`${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

---

实际开发中，通常一个系统会准备

- dev开发环境
- test测试环境
- prod生产环境

如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢？

一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境，测试环境，预发环境，正式环境....那么怎么对这些微服务配置进行管理呢？

Namespace+Group+Data ID三者关系？

<img src="../../Pictures/1561217857314-95ab332c-acfb-40b2-957a-aae26c2b5d71.jpeg" alt="关系图" style="zoom:50%;" />

默认情况：

```
Namespace=public, Group=DEFAULT_GROUP
```

---

**Nacos集群和持久化配置**

---

## 3. Sentinel

官网

```
https://github.com/alibaba/Sentinel
```

中文文档

```
https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D
```

下载地址

```
https://github.com/alibaba/Sentinel/releases
```

Sentinel本地安装路径

```
/Users/haipenghou/myjar/sentinel-dashboard-1.8.1.jar
```

运行命令

```
java -jar sentinel-dashboard-1.8.1.jar
```

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

Sentinel 的主要特性：

![Sentinel 的主要特性](../../Pictures/50505538-2c484880-0aaf-11e9-9ffc-cbaaef20be2b.png)

### 3.1 流控

流量控制主要有两种统计类型，一种是统计并发线程数，另外一种则是统计 QPS。类型由 `FlowRule` 的 `grade` 字段来定义。其中，0 代表根据并发数量来限流，1 代表根据 QPS 来进行流量控制。其中线程数、QPS 值，都是由 `StatisticSlot` 实时统计获取的。

**Sentinel 提供以下几种熔断策略**：

- 慢调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。
- 异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
- 异常数 (`ERROR_COUNT`)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

**熔断降级规则（DegradeRule）包含下面几个重要的属性：**

|                    |                                                              |            |
| ------------------ | ------------------------------------------------------------ | ---------- |
| Field              | 说明                                                         | 默认值     |
| resource           | 资源名，即规则的作用对象                                     |            |
| grade              | 熔断策略，支持慢调用比例/异常比例/异常数策略                 | 慢调用比例 |
| count              | 慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值 |            |
| timeWindow         | 熔断时长，单位为 s                                           |            |
| minRequestAmount   | 熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入） | 5          |
| statIntervalMs     | 统计时长（单位为 ms），如 60*1000 代表分钟级（1.8.0 引入）   | 1000 ms    |
| slowRatioThreshold | 慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）           |            |

### 3.3 热点

## 4. GateWay

访问服务url：`http://网关地址：网关端口/service名称/路由地址`

例如原始访问url：`http://localhost:8110/admin/core/integralGrade/list`

则通过网关访问的url：`http://localhost:80/service-core/admin/core/integralGrade/list`

或`http://localhost/service-core/admin/core/integralGrade/list`

### 4.1 路由配置

```
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #重点：gateway可以发现nacos中的微服务，并自动生成转发路由
      routes:
        - id: service-core
          uri: lb://service-core
          predicates:
            - Path=/*/core/**
        - id: service-sms
          uri: lb://service-sms
          predicates:
            - Path=/*/sms/**
        - id: service-oss
          uri: lb://service-oss
          predicates:
            - Path=/*/oss/**
```

通过`routes`配置后，访问路径可以变为`http://localhost/admin/core/integralGrade/list`

### 4.2 跨域配置

q