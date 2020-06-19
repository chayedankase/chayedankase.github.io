---
layout: post
title: Elastic Search入门
categories: [搜索技术]
description: Elastic Search搜索
keywords: Elastic Search
---



## Elastic Search

> ElasticSearch是一个基于[Lucene](https://baike.baidu.com/item/Lucene/6753302)的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于Restful web接口。ElasticSearch是用Java语言开发的，并作为Apache许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch用于[云计算](https://baike.baidu.com/item/云计算/9969353)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby和许多其他语言中都是可用的。根据DB-Engines的排名显示，ElasticSearch是最受欢迎的企业搜索引擎，其次是Apache Solr，也是基于Lucene。它是 Elastic Stack的核心模块
>
> index索引-type类型-document文档-field字段

### 一. helloword

> 我们先来快速启动一个ElasticSearch服务。系统环境为 CentOS release 6.8 (Final)

#### 1.下载

> 官网：https://www.elastic.co/cn/products/elastic-stack/features
>
> 下载页面： https://www.elastic.co/cn/downloads/
>
> 点击下图红框处可以选择更多版本，笔记中使用了6.5.4版，Linux的安装包
>
> ![image-20200120103814309]({{ site.url }}\imgs\elaticsearch\image-20200120103814309.png)
>
> ![image-20200120104041449]({{ site.url }}\imgs\elaticsearch\image-20200120104041449.png)

#### 2.安装

> 将下载的安装包 elasticsearch-6.5.4.tar.gz 上传到服务器的 /opt目录
>
> ![image-20200120104336101]({{ site.url }}\imgs\elaticsearch\image-20200120104336101.png)
>
> 使用命令： tar -zxvf elasticsearch-6.5.4.tar.gz -C /usr/local/   
>
> 将安装包解压至/usr/local/目录
>
> ![image-20200120104501861]({{ site.url }}\imgs\elaticsearch\image-20200120104501861.png)

#### 3.启动

> 在启动前，我们需要修改一些配置文件。
>
> 1. 进入解压好的目录中的config目录
>
>    ![image-20200120105925972]({{ site.url }}\imgs\elaticsearch\image-20200120105925972.png)
>
> 2. vim 打开 elasticsearch.yml，修改配置  network.host: 0.0.0.0 ，保证任意IP均可以访问服务。线上服务不要这样设置，要设成具体的 IP。
>
>    ![image-20200120110051341]({{ site.url }}\imgs\elaticsearch\image-20200120110051341.png)
>
> 3. vim 打开 jvm.options,elasticserch默认分配1G内存，我们调小一点128m。
>
>    ![image-20200120110439835]({{ site.url }}\imgs\elaticsearch\image-20200120110439835.png)
>
> 4. 修改完后我们返回上级目录，执行 bin/elasticsearch 命令启动服务。
>
>    ![image-20200120110702560]({{ site.url }}\imgs\elaticsearch\image-20200120110702560.png)
>
> 5. 如上图，启动失败了，提示需要配置JAVA_HOME 环境变量，因为elasticSearch是java写的，运行时需要jvm,所以我们要先安装jdk，不多赘述，快速安装一下。
>
>    ```
>    1.上传jdk安装包至 /opt目录 需要1.8以上版本，我们用的是jdk-8u211-linux-x64.tar.gz
>    2.解压至/usr/local/ 目录：tar -zxvf jdk-8u211-linux-x64.tar.gz -C /usr/local/
>    3.配置环境变量 JAVA_HOME:
>    	编辑 vim /etc/profile，底部增加内容
>    		JAVA_HOME=/usr/local/jak1.8.0_211
>    		export PATH=$JAVA_HOME/bin:$PATH
>    	保存并推出。
>    	执行 source /etc/profile 刷新配置文件。
>    	执行java -version命令，显示版本则配置完成
>    ```
>
>    ​	![image-20200120112725295]({{ site.url }}\imgs\elaticsearch\image-20200120112725295.png)
>
> 6. 再次执行启动命令。很遗憾又失败了，提示不能使用root用户运行。那就创建个其他用户吧。
>
>    ![image-20200120113819200]({{ site.url }}\imgs\elaticsearch\image-20200120113819200.png)
>
> 7. 创建其他用户，并将文件夹权限赋予相应用户。
>
>    ```shell
>    #创建一个elasticsearch用户使用默认的elasticsearch用户组
>    useradd elasticsearch
>    #修改文件所有权
>    cd ..
>    chown elasticsearch elasticsearch-6.5.4/ -R
>    ```
>
>    ![image-20200120115107183]({{ site.url }}\imgs\elaticsearch\image-20200120115107183.png)
>
> 8. 好的改完后重启，还是报错，这次是创建内存映射的最大数量太低。
>
>    ![image-20200120152346959]({{ site.url }}\imgs\elaticsearch\image-20200120152346959.png)
>
> 9. 我们接着改。
>
>    ```shell
>    #先切换回root用户
>    #修改进程在VMAs（虚拟内存区）创建的内存映射最大数量
>    vim /etc/sysctl.conf
>    vm.max_map_count=655360
>    #配置生效
>    sysctl -p
>    #除了上面的方式，还可以直接
>    sudo sysctl -w vm.max_map_count=262144
>    ```
>
> 10. 切换回elasticSearch用户，重新启动项目，出现如下信息表示启动成功。
>
>     ![image-20200120153705874]({{ site.url }}\imgs\elaticsearch\image-20200120153705874.png)
>
> 11. 浏览器访问服务地址的9200端口，出现数据，OK完成启动。
>
>     ![image-20200120153849394]({{ site.url }}\imgs\elaticsearch\image-20200120153849394.png)
>
> 12. 这个是前台启动，启动没问题后我们加上-d参数后台启动 bin/elasticsearch -d
>
>     本次使用的系统环境是CentOS release 6.8 (Final)，如果是其他环境可能遇到的问题不一样，一步一步解决即可。

#### 4.关闭

> 关闭后台启动的elasticSearch。
>
> ```SHELL
> #使用jps命令查询进程号
> jps
> #再使用kill 进程号命令 关闭
> kill  进程号
> ```
>
> ![image-20200120155059654](H:\笔记\imgs\elasticStack\image-20200120155059654.png)

#### 5. 安装elasticSearch-head工具

> 这个工具有好几种安装方法，为了使用简单，我们选择chrome插件。具体请百度。
>
> 安装成功后,点击插件，输入服务地址ip点击链接，ok。
>
> 使用此工具连接属于前后端分离，因而需要在后台配置跨域问题。
>
> ```YML
> http.cros.enabled: true
> http.cros.allow-origin: "*"
> # 跨域坑了我好久。。。这个插件不用跨域，其他的连接方式或许需要
> ```
>
> ![image-20200120162642269]({{ site.url }}\imgs\elasticStack\image-20200120162642269.png)

### 二、restFull API

> elasticSearch 提供了丰富的restFull api可以用来操作服务。
>
> 我们使用chrome插件postman作为测试工具

#### 1.创建非结构化索引

> 索引就相当于数据库中的表，要保存数据，就要先创建索引。
>
> ```JSON
> #创建索引
> PUT /kase
> {
> "settings":{
>   "index": {
>       "number_of_shards":"2", #分片数
>       "number_of_replicas":"0" # 副本数
>   }
> }
> }
> #删除索引
> DELETE /kase
> ```
>
> 如下图，可以看到kase索引创建成功。
>
> ![image-20200120202956245]({{ site.url }}\imgs\elaticsearch\image-20200120202956245.png)

#### 2.插入数据

> `
>
> ```JSON
>#插入一个user
> POST /kase/user/1001
>{
> "id": 1001,
>"name": "张三",
> "age": 20,
> "sex": "男"
> }
> #响应
>  {
>  "_index": "kase",
>  "_type": "user",
>  "_id": "1001",
> "_version": 1,
> "result": "created",
> "_shards": {
>    "total": 1,
>    "successful": 1,
>    "failed": 0
>  },
>  "_seq_no": 0,
>  "_primary_term": 1
>    }
>    ```
>    
>  在数据浏览标签中可以看到数据已经插入进去了。**前面的非结构化索引指的就是创建索引时并没有指定字段，但是在插入数据时，就根据插入的数据类型来自动确定了字段的类型**。
>  
>  ![image-20200120203113120]({{ site.url }}\imgs\elaticsearch\image-20200120203113120.png)
> 
> 路径不带id的插入，可以看到下面的响应中_id的值是自动生成的
>
> ```JSON
>POST /kase/user
> {
>"id": 1002,
> "name": "李四",
>"age": 30,
> "sex": "女"
> }
> #响应
>  {
>  "_index": "kase",
>  "_type": "user",
>  "_id": "o1Dzwm8Bp_jVXyAu_0MD",
> "_version": 1,
> "result": "created",
> "_shards": {
>    "total": 1,
>    "successful": 1,
>    "failed": 0
>  },
>  "_seq_no": 1,
>  "_primary_term": 1
>    }
>    ```
>    
>  可以看到生成的唯一标识 _id与我们传递的参数id并不一致。
>  
>  ![image-20200120203807875]({{ site.url }}\imgs\elaticsearch\image-20200120203807875.png)

#### 3.更新数据

> 在elasticSearch中数据是不能被更改的，但是可以通过覆盖的方式来对数据进行更新

##### 3.1全量更新

> url规则：PUT  /索引/类型/id
>
> ```JSON
> PUT /kase/user/1001
> {
>  "id": 1001,
>  "name": "张三",
>  "age": 10, #年龄由20改为10
>  "sex": "男"
> }
> #响应
> {
>    "_index" : "user",
>    "_type" : "_doc",
>    "_id" : "1001",
>    "_version" : 2,
>    "result" : "updated",
>    "_shards" : {
>        "total" : 1,
>        "successful" : 1,
>        "failed" : 0
>    },
>    "_seq_no" : 2,
>    "_primary_term" : 1
> }
> ```
>
> ![image-20200120204605862]({{ site.url }}\imgs\elaticsearch\image-20200120204605862.png)
>
> 再修改一次
>
> ```json
> PUT /kase/user/1001
> {
>  "id": 1001,
>  "name": "张三",
>  "age": 12, #年龄改为12,同时不传递sex的值  
> }
> #响应
> {
>  "_index": "kase",
>  "_type": "user",
>  "_id": "1001",
>  "_version": 3,
>  "result": "updated",
>  "_shards": {
>      "total": 1,
>      "successful": 1,
>      "failed": 0
>  },
>  "_seq_no": 3,
>  "_primary_term": 1
> }
> ```
>
> sex字段丢失了。为了保证数据不丢失，必须每次都附带全部的字段值，全量的覆盖，这不是我们想要的效果。
>
> ![image-20200120204938652]({{ site.url }}\imgs\elaticsearch\image-20200120204938652.png)

##### 3.2局部更新

> 要进行局部的更新需要在路径上增加额外的参数 _update，传参也有所不同
>
> POST /索引/类型/id/_update
>
> ```json
> POST /kase/user/1001/_update
> {
>  "doc": {
>     "age": 33 
>  }
> }
> #响应
> {
>     "_index": "kase",
>     "_type": "user",
>     "_id": "3",
>     "_version": 5,
>     "result": "updated",
>     "_shards": {
>         "total": 1,
>         "successful": 1,
>         "failed": 0
>     },
>     "_seq_no": 4,
>     "_primary_term": 1
> }
> ```
>
> ![image-20200120210018128]({{ site.url }}\imgs\elaticsearch\image-20200120210018128.png)

#### 4.删除数据

> url规则：DELETE  /索引/类型/id
>
> ```JSON
> DELETE /kase/user/1001
> ```
>
> 执行后将_id = 1001的数据删除掉。
>
> 如果删除一条不存在的数据，则会响应404，响应的结果会是 not_found
>
> 删除一个文档，不会立即从索引中删除，而是先标记为已删除，在后续操作中在统一删除。

#### 5.查询

##### 5.1 id搜索

> url规则：GET /索引/类型/id
>
> ```JSON
> GET /kase/user/o1Dzwm8Bp_jVXyAu_0MD
> #响应
> {
>  "_index": "kase",
>  "_type": "user",
>  "_id": "o1Dzwm8Bp_jVXyAu_0MD",
>  "_version": 1,
>  "found": true,
>  "_source": {
>      "id": 1002,
>      "name": "李四",
>      "age": 30,
>      "sex": "女"
>  }
> }
> ```

##### 5.2 全部搜索

> 搜索全部数据,默认只会返回10条数据，如果要获取更多数据，则要通过分页来获取。
>
> ```
> GET /kase/user/_search
> ```

##### 5.3 关键字搜索

> 关键字搜索是在全部搜索的基础上携带路径参数实现的
>
> ```JSON
> #搜索年龄=22的user对象。
> GET /kase/user/_search?q=age:22
> ```

#### 6.DSL搜索

> elasticSearch提供了丰富且灵活的查询方式，叫做DSL查询，它允许你构建更加复杂、强大的查询。
>
> 查询年龄是33的user 
>
> ```json
> POST /kase/user/_search
> {
> 	"query": {
> 		"match": { #匹配
> 			"age": 33
> 		}
> 	}
> }
> ```
>
> 查询年龄大于30的男性user
>
> ```JSON
> POST /kase/user/_search
> {
> "query":{
>   "bool":{
>       "filter":{
>           "range":{
>               "age":{
>                   "gt":30
>               }
>           }
>       },
>       "must":{
>           "match":{
>               "sex":"男"
>           }
>       }
>   }
> }
> }
> #响应
> {
> "took": 30,
> "timed_out": false,
> "_shards": {
>   "total": 2,
>   "successful": 2,
>   "skipped": 0,
>   "failed": 0
> },
> "hits": {
>   "total": 0,
>   "max_score": null,
>   "hits": []
> }
> }
> ```
>
> 全文搜索匹配姓名为张三或者李四的文档
>
> ```JSON
> POST /kase/user/_search
> {
>  "query":{
>      "match":{
>          "name":"张三 李四"
>      }
>  }
> }
> #响应
> {
>  "took": 17,
>  "timed_out": false,
>  "_shards": {
>      "total": 2,
>      "successful": 2,
>      "skipped": 0,
>      "failed": 0
>  },
>  "hits": {
>      "total": 2,
>      "max_score": 1.3862944,
>      "hits": [
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "o1Dzwm8Bp_jVXyAu_0MD",
>              "_score": 1.3862944,  #匹配度
>              "_source": {
>                  "id": 1002,
>                  "name": "李四",
>                  "age": 30,
>                  "sex": "女"
>              }
>          },
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "1001",
>              "_score": 1.3862944, #匹配度
>              "_source": {
>                  "id": 1001,
>                  "name": "张三",
>                  "age": 33
>              }
>          }
>      ]
>  }
> }
> ```

#### 7.高亮显示

> 高亮显示是基于DSL搜索的，只需要在搜索的json参数中附带额外的高亮参数指定高亮的字段即可
>
> ```JSON
> POST /kase/user/_search
> {
>  "query":{
>      "match":{
>          "name":"张三 李四"
>      }
>  },
>  #高亮
>  "highlight":{
>  	"fields":{
>  		"name":{}
> 		}
> 	}
> }
> #响应中额外存在高亮的标签
> {
>  "took": 63,
>  "timed_out": false,
>  "_shards": {
>      "total": 2,
>      "successful": 2,
>      "skipped": 0,
>      "failed": 0
>  },
>  "hits": {
>      "total": 2,
>      "max_score": 1.3862944,
>      "hits": [
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "o1Dzwm8Bp_jVXyAu_0MD",
>              "_score": 1.3862944,
>              "_source": {
>                  "id": 1002,
>                  "name": "李四",
>                  "age": 30,
>                  "sex": "女"
>              },
>              "highlight": {
>                  "name": [
>                      "<em>李</em><em>四</em>"
>                  ]
>              }
>          },
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "1001",
>              "_score": 1.3862944,
>              "_source": {
>                  "id": 1001,
>                  "name": "张三",
>                  "age": 33
>              },
>              "highlight": {
>                  "name": [
>                      "<em>张</em><em>三</em>"
>                  ]
>              }
>          }
>      ]
>  }
> }
> ```

#### 8.聚合

> 聚合相当于关系型数据库中的 group by 分组
>
> ```JSON
> {
>  "aggs":{
>      "all_interests":{
>          "terms":{
>              "fields":"age"
>          }
>      }
>  }
> }
> #响应，最下面是聚合结果
> {
>  "took": 72,
>  "timed_out": false,
>  "_shards": {
>      "total": 2,
>      "successful": 2,
>      "skipped": 0,
>      "failed": 0
>  },
>  "hits": {
>      "total": 2,
>      "max_score": 1,
>      "hits": [
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "o1Dzwm8Bp_jVXyAu_0MD",
>              "_score": 1,
>              "_source": {
>                  "id": 1002,
>                  "name": "李四",
>                  "age": 30,
>                  "sex": "女"
>              }
>          },
>          {
>              "_index": "kase",
>              "_type": "user",
>              "_id": "1001",
>              "_score": 1,
>              "_source": {
>                  "id": 1001,
>                  "name": "张三",
>                  "age": 33
>              }
>          }
>      ]
>  },
>  "aggregations": {
>      "all_interests": {
>          "doc_count_error_upper_bound": 0,
>          "sum_other_doc_count": 0,
>          "buckets": [
>              {
>                  "key": 30,#聚合的值
>                  "doc_count": 1 #获取到的个数
>              },
>              {
>                  "key": 33,
>                  "doc_count": 1
>              }
>          ]
>      }
>  }
> }
> ```

### 三、核心详解

#### 1.文档

> 文档document指的就是存储在elastisearch 分片中的数据，它是以json格式进行存储的，可以存储复杂的结构。
>
> ```JSON
> {
> "_index": "kase",
> "_type": "user",
> "_id": "1001",
> "_version": 4,
> "found": true,
> "_source": {
>   "id": 1001,
>   "name": "张三",
>   "age": 33,
>   "car":{
>       "color":"red"
>   }
> }
> }
> ```
>
> 文档中不仅仅存储有自己的属性，还存储有元数据（metadata）
>
> | 节点   | 说明                   |
> | ------ | ---------------------- |
> | _index | 文档存储的地方（索引） |
> | _type  | 文档代表的对象的类型   |
> | _id    | 文档的唯一标识         |
>
> 索引 index 相当于mysql的数据库 , 而类型type 相当于表 , 区别在于当在type中定义了字段类型后,就不能修改了,而且是整个index通用的 . 比如 定义了一个 user类型,里面有一个 name字段是test的,那么在其他的类型中name也只能是test . 因而高版本的elasticSearch逐步取消了type 。可以将index理解为一个表 ， type属性废弃不用。

#### 2.响应

##### 2.1 美化响应

可以在查询url后面添加pretty参数，使得返回的json更易查看。不添加的话，在浏览器中就直接是显示一行。

GET /kase/user/3?pretty

```JSON
{
    "_index": "kase",
    "_type": "user",
    "_id": "3",
    "_version": 5,
    "found": true,
    "_source": {
        "name": "王五",
        "age": 30,
        "six": "男",
        "height": 10
    }
}
```

##### 2.2 指定响应字段

查询时每次返回所有字段，我们可以在url后增加_search参数来限定响应的字段

GET /kase/user/3?_source=name,age

```JSON
{
    "_index": "kase",
    "_type": "user",
    "_id": "3",
    "_version": 5,
    "found": true,
    "_source": {
        "name": "王五",
        "age": 30
    }
}
```

上面的返回值包含有元数据，能否去掉元数据的响应呢，也是可以的

GET /kase/user/3/_source? _source=name,age

```JSON
{
    "name": "王五",
    "age": 30
}
```

#### 3.判断文档是否存在

如果我们只需要判断文档是否存在，而不是查询文档内容，那么可以这样：

HEAD /kase/user/1

状态值 status 200 就表示文档存在，status 404则表示不存在。

#### 4.批量操作

##### 4.1 批量查询

POST /kase/user/_mget

```JSON
#请求参数
{
    "ids":["1","2","3"]
}
```

响应

```JSON
{
    "docs": [
        {
            "_index": "kase",
            "_type": "user",
            "_id": "1",
            "_version": 2,
            "found": true,
            "_source": {
                "name": "李四",
                "age": 30,
                "six": "女",
                "height": "165"
            }
        },
        {
            "_index": "kase",
            "_type": "user",
            "_id": "2",
            "_version": 1,
            "found": true,
            "_source": {
                "name": "张三",
                "age": 21,
                "six": "男",
                "height": "175"
            }
        },
        {
            "_index": "kase",
            "_type": "user",
            "_id": "3",
            "_version": 5,
            "found": true,
            "_source": {
                "name": "王五",
                "age": 30,
                "six": "男",
                "height": 10
            }
        }
    ]
}
```

如果请求的id中有不存在的，并不会影响其他的响应

```JSON
#请求参数
{
    "ids":["1","2","3","300"]
}
#响应
{
    "docs": [
        {
            "_index": "kase",
            "_type": "user",
            "_id": "1",
            "_version": 2,
            "found": true,
            "_source": {
                "name": "李四",
                "age": 30,
                "six": "女",
                "height": "165"
            }
        },
        {
            "_index": "kase",
            "_type": "user",
            "_id": "2",
            "_version": 1,
            "found": true,
            "_source": {
                "name": "张三",
                "age": 21,
                "six": "男",
                "height": "175"
            }
        },
        {
            "_index": "kase",
            "_type": "user",
            "_id": "3",
            "_version": 5,
            "found": true,
            "_source": {
                "name": "王五",
                "age": 30,
                "six": "男",
                "height": 10
            }
        },
        {
            "_index": "kase",
            "_type": "user",
            "_id": "300",
            "found": false
        }
    ]
}
```

##### 4.2 _bulk操作

在Elasticsearch中，支持批量的插入、修改、删除操作，都是通过_bulk的api完成的。
请求格式如下：（请求格式不符合json格式要求）

```JSON
{ action: { metadata }}\n #命令行
{ request body        }\n #参数行
{ action: { metadata }}\n 
{ request body        }\n 
```

批量插入，注意每行末尾的换行符。

命令 GET /_bulk  如果命令行已经指定了索引和类型，则路径中可以不附带。如果路径中附带了则命令行中也可以不指定。

```JSON
{"create":{"_index":"kase","_type":"user","_id":"6"}}
{"name":"唐僧","age":"23","six":"男","height":"174"}
{"create":{"_index":"kase","_type":"user","_id":"7"}}
{"name":"孙悟空","age":"1024","six":"男","height":"164"}
{"create":{"_index":"kase","_type":"user","_id":"8"}}
{"name":"猪八戒","age":"34","six":"男","height":"176"}
{"create":{"_index":"kase","_type":"user","_id":"9"}}
{"name":"沙悟净","age":"36","six":"男","height":"182"}

#响应
{
    "took": 46,
    "errors": false,
    "items": [
        {
            "create": {
                "_index": "kase",
                "_type": "user",
                "_id": "6",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 5,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "kase",
                "_type": "user",
                "_id": "7",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 5,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "kase",
                "_type": "user",
                "_id": "8",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 6,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "kase",
                "_type": "user",
                "_id": "9",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 7,
                "_primary_term": 1,
                "status": 201
            }
        }
    ]
}
```

批量删除  命令行为 delete，命令如下，此处不做测试。

```JSON
{"delete":{"_index":"kase","_type":"user","_id":"1"}}
{"delete":{"_index":"kase","_type":"user","_id":"2"}}
{"delete":{"_index":"kase","_type":"user","_id":"3"}}
```

其他操作类似。

一次请求多少性能高？

- 整个批量请求需要被加载到接受我们请求节点的内存里，所以请求越大，给其它请求可用的内存就越小。有一 个最佳的bulk请求大小。超过这个大小，性能不再提升而且可能降低。
- 最佳大小，当然并不是一个固定的数字。它完全取决于你的硬件、你文档的大小和复杂度以及索引和搜索的负载。
- 幸运的是，这个最佳点(sweetspot)还是容易找到的：试着批量索引标准的文档，随着大小的增长，当性能开始降低，说明你每个批次的大小太大了。开始的数量可以在1000~5000个文档之间，如果你的文档非常大，可以 使用较小的批次。
- 通常着眼于你请求批次的物理大小是非常有用的。一千个1kB的文档和一千个1MB的文档大不相同。一个好的批次好保持在5-15MB大小间。 

#### 5. 分页

默认的 _search 查询只会返回10条数据，和SQL使用 LIMIT 关键字返回只有一页的结果一样，Elasticsearch接受 from 和 size 参数用来进行分页查询：

size: 结果数，默认就是10。

from: 跳过开始的结果数，默认是 0 。

如果想每页显示5条数据，分别获取 1,2,3页码的数据

```JSON
GET /_search?size=5 
GET /_search?size=5&from=5 
GET /_search?size=5&from=10
```

**应该当心分页太深或者一次请求太多的结果。结果在返回前会被排序。但是记住，一个搜索请求常常涉及多个分片。每个分片生成自己排好序的结果，它们接着需要集中起来排序以确保整体排序正确。**

在集群系统中深度分页

为了理解为什么深度分页是有问题的，让我们假设在一个有5个主分片的索引中搜索。当我们请求结果的第一 （结果1到10）时，每个分片产生自己顶端10个结果然后返回它们给请求节点(requesting node)，它再 排序这所有的50个结果以选出顶端的10个结果。现在假设我们请求第1000页——结果10001到10010。工作方式都相同，不同的是每个分片都必须产生顶端的 10010个结果。然后请求节点排序这50050个结果并丢弃50040个！你可以看到在分布式系统中，排序结果的花费随着分页的深入而成倍增长。这也是为什么网络搜索引擎中任何 语句不能返回多于1000个结果的原因。 

#### 6.映射（mapping）

前面我们创建的索引以及插入数据，都是由Elasticsearch进行自动判断类型，有些时候我们是需要进行明确字段型 的，否则，自动判断的类型和实际需求是不相符的。

自动判断的规则如下：

| JSON type                        | Field type |
| -------------------------------- | ---------- |
| Boolean: true or false           | "boolean"  |
| Whole number: 123                | "long"     |
| Floating point: 123.45           | "double"   |
| String, valid date: "2014-09-15" | "date"     |
| String: "foo bar"                | "string"   |

Elasticsearch中支持的类型如下：

| 类型           | 表示的数据类型                |
| -------------- | ----------------------------- |
| String         | string , text , keyword       |
| Whole number   | byte , short , integer , long |
| Floating point | float , double                |
| Boolean        | boolean                       |
| Date           | date                          |

- string类型在ElasticSearch 旧版本中使用较多，从ElasticSearch 5.x开始不再支持string，由text和 keyword类型替代。
- text 类型，当一个字段是要被全文搜索的，比如Email内容、产品描述，应该使用text类型。设置text类型 以后，字段内容会被分析，在生成倒排索引以前，字符串会被分析器分成一个一个词项。text类型的字段 不用于排序，很少用于聚合。
- keyword类型适用于索引结构化的字段，比如email地址、主机名、状态码和标签。如果字段需要进行过 滤(比如查找已发布博客中status属性为published的文章)、排序、聚合。keyword类型的字段只能通过精 确值搜索到。

创建明确类型的索引

```JSON
PUT /chadan
{    
    "settings": {        
        "index": {            
            "number_of_shards": "2",
            "number_of_replicas": "0"
        }    
    },    
    "mappings": {        
        "person": {            
            "properties": {                
                "name": {                    
                    "type": "text"                
                },                
                "age": {                    
                    "type": "integer"                
                },                
                "mail": {                    
                    "type": "keyword"               
                },                
                "hobby": {                    
                    "type": "text"                
                }            
            }        
        }    
    } 
}
```

查看映射：

```JSON
GET /kase/_mapping
#响应
{
    "kase": {
        "mappings": {
            "user": {
                "properties": {
                    "age": {
                        "type": "long"
                    },
                    "height": {
                        "type": "text",
                        #默认的给text类型字段下的fields还会创建一个 keyword类型，只取
                        #长度256
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "name": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    },
                    "six": {
                        "type": "text",
                        "fields": {
                            "keyword": {
                                "type": "keyword",
                                "ignore_above": 256
                            }
                        }
                    }
                }
            }
        }
    }
}

GET /chadan/_mapping
#响应
{
    "chadan": {
        "mappings": {
            "person": {
                "properties": {
                    "age": {
                        "type": "integer"
                    },
                    "hobby": {
                        "type": "text"
                    },
                    "mail": {
                        "type": "keyword"
                    },
                    "name": {
                        "type": "text"
                    }
                }
            }
        }
    }
}
```

插入数据

```JSON
POST /chadan/person/_bulk
#请求
{"create":{"_id":"1"}}
{"name":"张三","age": 20,"mail": "111@qq.com","hobby":"羽毛球、乒乓球、足球"}
{"create":{"_id":"2"}}
{"name":"李四","age": 21,"mail": "222@qq.com","hobby":"羽毛球、乒乓球、足球、篮球"} 
{"create":{"_id":"3"}}
{"name":"王五","age": 22,"mail": "333@qq.com","hobby":"羽毛球、篮球、游泳、听音乐"} 
{"create":{"_id":"4"}}
{"name":"赵六","age": 23,"mail": "444@qq.com","hobby":"跑步、游泳"}
{"create":{"_id":"5"}}
{"name":"孙七","age": 24,"mail": "555@qq.com","hobby":"听音乐、看电影"}

#响应
{
    "took": 68,
    "errors": false,
    "items": [
        {
            "create": {
                "_index": "chadan",
                "_type": "person",
                "_id": "1",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 0,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "chadan",
                "_type": "person",
                "_id": "2",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 1,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 0,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "chadan",
                "_type": "person",
                "_id": "4",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 2,
                "_primary_term": 1,
                "status": 201
            }
        },
        {
            "create": {
                "_index": "chadan",
                "_type": "person",
                "_id": "5",
                "_version": 1,
                "result": "created",
                "_shards": {
                    "total": 1,
                    "successful": 1,
                    "failed": 0
                },
                "_seq_no": 3,
                "_primary_term": 1,
                "status": 201
            }
        }
    ]
}
```

测试搜索

```JSON
POST /chadan/_search
#请求
{
	"query":{
		"match":{
			"hobby":"音乐"
		}
	}
}
#响应
{
    "took": 4,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 2.5574043,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "5",
                "_score": 2.5574043,
                "_source": {
                    "name": "孙七",
                    "age": 24,
                    "mail": "555@qq.com",
                    "hobby": "听音乐、看电影"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 0.5753642,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            }
        ]
    }
}
```

#### 7.结构化查询

##### 7.1 term查询

`term` 主要用于精确匹配哪些值，比如数字，日期，布尔值或 `not_analyzed` 的字符串(未经分析的文本数据类型)：

```JSON
{ "term": { "age":    26           }}    
{ "term": { "date":   "2014-09-01" }}    
{ "term": { "public": true         }}    
{ "term": { "tag":    "full_text"  }}
```

```JSON
POST /chadan/_search
#请求
{
    "query":{
        "term":{
            "age":20
        }
    }
}
#响应
{
    "took": 13,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 1,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            }
        ]
    }
}
```

##### 7.2 terms查询

`terms` 跟 `term` 有点类似，但 `terms` 允许指定多个匹配条件。 如果某个字段指定了多个值，那么文档需要一起去做匹配：

```JSON
POST /chadan/_search
#请求
{
    "query":{
        "terms":{
            "age":[23,22,25]
        }
    }
}
#响应
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 2,
        "max_score": 1,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 1,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "4",
                "_score": 1,
                "_source": {
                    "name": "赵六",
                    "age": 23,
                    "mail": "444@qq.com",
                    "hobby": "跑步、游泳"
                }
            }
        ]
    }
}
```

##### 7.3 range查询

`range` 过滤允许我们按照指定范围查找一批数据：

```JSON
{
    "query":{
        "range":{
            "age":{
                "gte":20,
                "lt":30
            }
        }
    }
}
```

范围操作符包含：
gt :: 大于
gte :: 大于等于
lt :: 小于
lte :: 小于等于

```JSON
POST /chadan/_search
#请求
{
    "query":{
        "range":{
            "age":{
                "gte":22,
                "lte":25
            }
        }
    }
}
#响应
{
    "took": 1,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 3,
        "max_score": 1,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 1,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "4",
                "_score": 1,
                "_source": {
                    "name": "赵六",
                    "age": 23,
                    "mail": "444@qq.com",
                    "hobby": "跑步、游泳"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "5",
                "_score": 1,
                "_source": {
                    "name": "孙七",
                    "age": 24,
                    "mail": "555@qq.com",
                    "hobby": "听音乐、看电影"
                }
            }
        ]
    }
}
```

##### 7.4 exists 查询

`exists` 查询可以用于查找文档中是否包含指定字段或没有某个字段，类似于SQL语句中的 `IS_NULL` 条件

```JSON
{
    "exists":{
        "field":"title"
    }
}
```

这个查询只是针对已经查出一批数据来，但是想区分出某个字段是否存在的时候使用。

```JSON
POST /chadan/_search
#请求
{
	"query":{
		"exists":{
			"field":"age"
		}
	}
}
#响应
{
    "took": 13,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 5,
        "max_score": 1,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 1,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "2",
                "_score": 1,
                "_source": {
                    "name": "李四",
                    "age": 21,
                    "mail": "222@qq.com",
                    "hobby": "羽毛球、乒乓球、足球、篮球"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "4",
                "_score": 1,
                "_source": {
                    "name": "赵六",
                    "age": 23,
                    "mail": "444@qq.com",
                    "hobby": "跑步、游泳"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "5",
                "_score": 1,
                "_source": {
                    "name": "孙七",
                    "age": 24,
                    "mail": "555@qq.com",
                    "hobby": "听音乐、看电影"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "1",
                "_score": 1,
                "_source": {
                    "name": "张三",
                    "age": 20,
                    "mail": "111@qq.com",
                    "hobby": "羽毛球、乒乓球、足球",
                    "height": "哈哈哈"
                }
            }
        ]
    }
}
```

##### 7.5 match查询

`match` 查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。
如果你使用 `match` 查询一个全文本字段，它会在真正查询之前用分析器先分析 `match` 一下查询字符：

```JSON
{
	"query":{
		"match":{
			"hobby":"足球 音乐"
		}
	}
}
```

如果用 `match` 下指定了一个确切值，在遇到数字，日期，布尔值或者 `not_analyzed `的字符串时，它将为你搜索你 给定的值：

```JSON
{ "match": { "age":    26           }} 
{ "match": { "date":   "2014-09-01" }} 
{ "match": { "public": true         }} 
{ "match": { "tag":    "full_text"  }}
```

##### 7.6 bool查询

bool 查询可以用来合并多个条件查询结果的布尔逻辑，它包含一下操作符：
must :: 多个查询条件的完全匹配,相当于 and 。
must_not :: 多个查询条件的相反匹配，相当于 not 。
should :: 至少有一个查询条件匹配, 相当于 or 。
这些参数可以分别继承一个查询条件或者一个查询条件的数组：

```JSON
POST /chadan/_search
#请求
{
	"query":{
		"bool":{
			"must":{
				"range":{
					"age":{
						"gt":20
					}
				}
			},
			"must_not":{
				"term":{
					"age":"24"
				}
			},
			"should":[
				{
					"term":{
						"mail":"444@qq.com"
					}
				}
				]
		}
	}
}
#响应 
{
    "took": 5,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 3,
        "max_score": 2.3862944,
        "hits": [
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "4",
                "_score": 2.3862944,
                "_source": {
                    "name": "赵六",
                    "age": 23,
                    "mail": "444@qq.com",
                    "hobby": "跑步、游泳"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "3",
                "_score": 1,
                "_source": {
                    "name": "王五",
                    "age": 22,
                    "mail": "333@qq.com",
                    "hobby": "羽毛球、篮球、游泳、听音乐"
                }
            },
            {
                "_index": "chadan",
                "_type": "person",
                "_id": "2",
                "_score": 1,
                "_source": {
                    "name": "李四",
                    "age": 21,
                    "mail": "222@qq.com",
                    "hobby": "羽毛球、乒乓球、足球、篮球"
                }
            }
        ]
    }
}
```

