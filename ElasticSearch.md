

# ElasticSearch

## 1. 安装

### 1.1 ElasticSearch

官网：`https://www.elastic.co`

位置：`/Users/haipenghou/software/elasticsearch-7.13.4`,解压即用

默认端口：9200

### 1.2 head插件

作用：主要用于数据展示

位置：`/Users/haipenghou/WebProjects/elasticsearch-head`

启动：

- `npm install`:有错误不影响
- `npm run start`

端口：9100

### 1.3 Kibaba

位置：`/Users/haipenghou/software/kibana-7.14.0-darwin-x86_64`

端口：5601

![68a9a5ffcaf8747d780d2c3f316f7ef7](/Users/haipenghou/Pictures/ELK/68a9a5ffcaf8747d780d2c3f316f7ef7.jpg)

### 1.4 IK分词器

位置：`/Users/haipenghou/software/elasticsearch-7.13.4/plugins/elasticsearch-analysis-ik-7.13.4`

## 2. 目录

```
bin #启动文件
config #配置文件
	log4j2.properties #日志配置文件
	jvm.options #java虚拟机相关配置
	elasticsearch.yml #elasticsearch的配置文件,配置跨域
lib #相关jar包
plugins #插件，ik
logs #日志
```

跨域配置:在`elasticsearch.yml`添加

```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 3. IK分词器

这里的索引类似一个数据库

IK提供了两个分词算法：`ik_smart`(最少切分)和`ik_max_word`(最细粒度划分)

```
#查看安装的插件
elasticsearch-plugin list
```

`ik_smart`

```
#请求
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "中华人民共和国"
}
#响应
{
  "tokens" : [
    {
      "token" : "中华人民共和国",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 0
    }
  ]
}
```

`ik_max_word`

```
#请求
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "中华人民共和国"
}
#响应
{
  "tokens" : [
    {
      "token" : "中华人民共和国",
      "start_offset" : 0,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "中华人民",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "中华",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "华人",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "人民共和国",
      "start_offset" : 2,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "人民",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "共和国",
      "start_offset" : 4,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "共和",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 7
    },
    {
      "token" : "国",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "CN_CHAR",
      "position" : 8
    }
  ]
}
```

自己需要的词，需要自己加到分词器的字典中,

在`config/`下创建`hhp.dic`,添加如下内容

```
侯海鹏
```

在`config/IKAnalyzer.cfg.xml`中添加`hhp.dic`

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!--用户可以在这里配置自己的扩展字典 -->
	<entry key="ext_dict">hhp.dic</entry>
	 <!--用户可以在这里配置自己的扩展停止词字典-->
	<entry key="ext_stopwords"></entry>
	<!--用户可以在这里配置远程扩展字典 -->
	<!-- <entry key="remote_ext_dict">words_location</entry> -->
	<!--用户可以在这里配置远程扩展停止词字典-->
	<!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

## 4. 基本操作

### 4.1 Rest风格

| method |                     url地址                     |         描述         |
| :----: | :---------------------------------------------: | :------------------: |
|  PUT   |     localhost:9200/索引名称/类型名称/文档id     | 创建文档(指定文档id) |
|  POST  |        localhost:9200/索引名称/类型名称         | 创建文档(随机文档id) |
|  POST  | localhost:9200/索引名称/类型名称/文档id/_update |       修改文档       |
|  POST  |    localhost:9200/索引名称/类型名称/_search     |     查询所有数据     |
| DELETE |     localhost:9200/索引名称/类型名称/文档id     |       删除文档       |
|  GET   |     localhost:9200/索引名称/类型名称/文档id     |       查询文档       |

### 4.2 创建索引

```
# test1:索引名
# type1:类型名
# 1:文档id
PUT /test1/type1/1
{
  "name": "侯海鹏",
  "age": 27
}
#以后类型名默认是_doc,如果没有指定数据类型，es默认配置
PUT /test3/_doc/1
{
  "name": "侯海鹏",
  "age": 11,
  "birth": "1994-10-18"
}
```

### 4.3 数据类型

- 字符串:`text`,`keyword`
	- text可以被分词器解析，keyword不可以
- 数值：`long`,`integer`,`short`,`double`,`float`,`half float`,`scaled float`
- 日期：`date`
- 布尔：`boolean`
- 二进制：`binary`

```
#创建索引，指定类型
PUT /test2
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "age": {
        "type": "long"
      },
      "birthday": {
        "type": "date"
      }
    }
  }
}

#获取索引
GET test2
#结果
{
  "test2" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birthday" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test2",
        "creation_date" : "1628084029009",
        "number_of_replicas" : "1",
        "uuid" : "5fpA4dtiQ3OaTjd_hshQEw",
        "version" : {
          "created" : "7130499"
        }
      }
    }
  }
}
```

### 4.4 文档基本操作（重点）

#### 4.4.1 基本操作

```
POST _update,推荐使用这种更新方式,只会对需要修改的数据进行变动，不会替换原有的所有数据

POST /hhp/user/2/_update
{
  "doc":{
    "name":"刘籽鑫"
  }
}

#查询
GET hhp/user/_search?q=name:刘籽鑫
#等价于
GET hhp/user/_search
{
  "query": {
    "match": {
      "name": "刘籽鑫"
    }
  }
}
#hits会对应一个java对象
```

#### 4.4.2 复杂操作

```
#排序，分页，高亮，模糊查询，精准查询

GET hhp/user/_search
{
  "query": {
  	//match会使用分词器解析
    "match": {
      "name": "刘籽鑫"，
      //多条件查询,多条件使用空格隔开,只要满足其中一个就可以被查出来，可以通过score查看匹配度
      "tags": "男 技术",
      //通过倒排索引进行精确查询
      "term": 
    },
    //或，布尔值查询，
    //must(类似and)
    //should(类似or)
    //must_not(类似not)
    "bool": {
      "must": [
        {
          "match": {
            "name": "刘籽鑫"
          }
        },
        {
          "match": {
            "age": 60
          }
        }
      ],
      //过滤条件
      "filter": [
        {
          "range": {
            "age": {
              "gte": 10,
              "lte": 80
            }
          }
        }
      ]
    }
  },
  //类似sql：select name,desc from ....
  "_source": ["name", "desc"],
  "sort": [
      {
        "age": {
        	//降序排序
          "order": "desc"
        }
      }
    ],
    //类似sql:limit 0,2或limit 2 offset 0.从第0个数据开始，取2个数据
    "from": 0,
    "size": 2
}


```

## 5. 集成SpringBoot

`MAVEN`

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.13.4</version>
</dependency>
```

初始化

```
RestHighLevelClient client = new RestHighLevelClient(
        RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")));
```

关闭客户端

```
client.close();
```

