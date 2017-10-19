# Elasticsearch 学习笔记

- [Elasticsearch 学习笔记](#elasticsearch-%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0)
    - [（一）专业术语：](#%EF%BC%88%E4%B8%80%EF%BC%89%E4%B8%93%E4%B8%9A%E6%9C%AF%E8%AF%AD%EF%BC%9A)
        - [文档(Document)](#%E6%96%87%E6%A1%A3document)
        - [类型(Type)](#%E7%B1%BB%E5%9E%8Btype)
        - [索引(index)](#%E7%B4%A2%E5%BC%95index)
    - [（二）Java操作Elasticsearch的几种方式：](#%EF%BC%88%E4%BA%8C%EF%BC%89java%E6%93%8D%E4%BD%9Celasticsearch%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E5%BC%8F%EF%BC%9A)
        - [节点客户端(Node client)：](#%E8%8A%82%E7%82%B9%E5%AE%A2%E6%88%B7%E7%AB%AFnode-client%EF%BC%9A)
        - [传输客户端(Transport client)：](#%E4%BC%A0%E8%BE%93%E5%AE%A2%E6%88%B7%E7%AB%AFtransport-client%EF%BC%9A)
        - [Spring boot中使用ElasticsearchTemplate操作ES：](#spring-boot%E4%B8%AD%E4%BD%BF%E7%94%A8elasticsearchtemplate%E6%93%8D%E4%BD%9Ces%EF%BC%9A)
    - [（三）Elasticsearch Restful API操作](#%EF%BC%88%E4%B8%89%EF%BC%89elasticsearch-restful-api%E6%93%8D%E4%BD%9C)

## （一）专业术语：

### 文档(Document)

    特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）。通常， 我们可以认为对象(object)和文档(document)是等价相通的。

### 类型(Type)

    在Elasticsearch中每一条文档都归属于一种类型（Type），而这些类型存储在索引（index）中。

### 索引(index)

    Elasticsearch中存储数据的行为就叫做索引(indexing)，最终生成的索引集合（indices）成为索引（indices）。
我们可以简单地来类比传统关系型数据库来理解这些概念：
| Relational DB | Databases | Tables | Rows | Columns |
| --------------| --------- | -------| -----| --------|
| Elasticsearch | Indices   | Types  | Documents| Fields|
Elasticsearch集群可以包含多个索引(indices)（数据库） ， 每一个索引可以包含多个类型(types)（表） ， 每一个类型包含多个文档(documents)（行） ， 然后每个文档包含多个字段(Fields)（列）。

## （二）Java操作Elasticsearch的几种方式：

Elasticsearch为Javay用户提供了两种内置客户端:

### 节点客户端(Node client)：

    节点客户端以无数据节点(none data node)身份加入集群， 换言之， 它自己不存储任何数据， 但是它知道数据在集群中的具体位置， 并且能够直接转发请求到对应的节点上。
首先引入如下类：

``` java
import static org.elasticsearch.node.NodeBuilder.nodeBuilder;
import org.elasticsearch.client.Client;
import org.elasticsearch.node.Node;
```

我们声明了Client接口、Node接口和NodeBuilder，并用他们来建立到ElasticSearch的连接。下面的代码片段创建了一个到客户端的实例：

``` java
Node node = nodeBuilder().clusterName("escluster2").client(true).node();
Client client = node.client();
```

### 传输客户端(Transport client)：

    这个更轻量的传输客户端能够发送请求到远程集群。 它自己不加入集群， 只是简单转发请求给集群中的节点。
首先导入依赖包：

``` java
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.ImmutableSettings;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
```

接下来是建立连接的代码:

``` java
Settings settings = ImmutableSettings.settingsBuilder().put("cluster.name", "escluster2").build();
        TransportClient client = new TransportClient(settings);
        client.addTransportAddress(new InetSocketTransportAddress("127.0.0.1", 9300));
```

两个Java客户端都通过9300端口与集群交互， 使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。 集群中的节点之间也通过9300端口进行通信。 如果此端口未开放， 你的节点将不能组成集群。

### Spring boot中使用ElasticsearchTemplate操作ES：

    Spring boot 提供了封装好的ElasticsearchTemplate类来供我们方便地操作Elasticsearch。

在application.yml中配置Elasticsearch相关设置：

``` yml
spring:
   data:
        elasticsearch:
            #cluster-name: #默认为elasticsearch
            #cluster-nodes: 112.74.72.18:9300 #配置es节点信息，逗号分隔，如果没有指定，则启动ClientNode
            properties:
                path:
                  logs: ./elasticsearch/log #elasticsearch日志存储目录
                  data: ./elasticsearch/data #elasticsearch数据存储目录
```

来看一下cluster-nodes，这里如果直接注释掉，不配置nodes，那么默认就是本机！如果要使用远程服务器，或者局域网服务器，那就需要在这里配置IP：PORT。
可以配置多个，以逗号分隔，相当于集群。cluster-name如果想改为其他，那就需要在安装ElasticSearch时编辑配置文件，设置name。
配置好yml后，就可以直接使用template了！

在具体的代码中我们就可以直接注入ElasticsearchTemplate使用了:

``` java
public class Test () {
    @Autowired
        ElasticsearchTemplate elasticsearchTemplate;
        public void test() {
            SearchQuery searchQuery = new NativeSearchQueryBuilder().withQuery(queryStringQuery("spring boot OR 书籍")).build();
            List<Article> articles = elasticsearchTemplate.queryForList(searchQuery, Article.class);
            for (Article article : articles) {
                System.out.println(article.toString());
            }
        }
}
```

## （三）Elasticsearch Restful API操作

