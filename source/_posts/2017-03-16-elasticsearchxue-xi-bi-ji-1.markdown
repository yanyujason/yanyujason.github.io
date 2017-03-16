---
layout: post
title: "ElasticSearch学习笔记(一)"
date: 2017-03-16 22:22:07 +0800
comments: true
categories: ElasticSearch
---

# 从0开始搭建ElasticSearch与Kibana

[ElasticSearch](https://www.elastic.co/products/elasticsearch)是基于[Lucene](http://lucene.apache.org/)的搜索引擎，它提供了一个分布式的全文搜索引擎，是当前流行的企业级搜索引擎。ElasticSearch趋近达到实时搜索，稳定、可靠、安装方便。

[Kibana](https://www.elastic.co/products/kibana)是一个开源免费的工具，用于提供有好的web界面帮助用户做数据展示、汇总以及数据日志搜索。

本文主要介绍如何在Apple Mac操作系统以及AWS([Amazon Web Services](https://aws.amazon.com/cn/))上快速搭建一套ElasticSearch与Kibana的系统。

<br />

## 本地环境

### ElasticSearch安装

目前最新的ElasticSearch版本为5.2.2。5.0以上版本ES需要JAVA 8的支持，所以需要Oracle JDK 1.8以上。如果使用ES V5.0之下的版本（比如V2.4），只需要JAVA 7支持即可。本文主要介绍V5.1.1版本的安装。安装ElasticSearch非常简单，只需要以下3个步骤：

首先，根据操作系统，确保环境已安装合适的[Oracle JDK 1.8](http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html)。进入命令行，使用下列命令确保JAVA版本正确。

```
java -version
echo $JAVA_HOME
```

以下为安装成功结果:

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/java_success.png" width = "600" height = "400" />
</div>

其次，在Elastic官网官方下载页面下载[5.1.1版本](https://www.elastic.co/downloads/past-releases/elasticsearch-5-1-1)。也可通过命令行直接下载：

```
cd DIR_YOU_WANT_TO_PUT_ElasticSearch_IN
mkdir ElasticSearch && cd ElasticSearch
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.1.1.tar.gz
tar -xvf elasticsearch-5.1.1.tar.gz
cd elasticsearch-5.1.1
ls
```

以下为成功下载ElasticSearch结果：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/es_success.png" width = "600" height = "400" />
</div>

最后，进入`bin`目录，启动ElasticSearch：

```
cd bin
./elasticsearch
```

以下是启动结果：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/start_es.png" width = "800" height = "600" />
</div>

如上图所示，ElasticSearch已经成功启动，一个集群只有一个结点。图中[53F-VlU]为ElasticSearch启动成功后主结点(Master)的名字。ElasticSearch是去中心化的，但是对于外界来说，一个ElasticSearch就是一个集群，对外的逻辑暴露为一个整体，外界与一个ElasticSearch集群的任何结点的交互都是等价的。默认的启动只有一个结点。

同时，我们也可以自己定义结点名称：

```
./elasticsearch -Ecluster.name=fancyCluster -Enode.name=fancyNode
```

此时，主节点的名字就会变成`fancyNode`。

所有关于ElasticSearch的配置都在`config/elasticsearch.yml`，可以通过修改该文件来对ElasticSearch作出相应配置。比如：集群名称、节点名称、启动端口等，这里不做一一介绍。

<br />

### Kibana安装

Kibana的安装与ElasticSearch类似，也是属于傻瓜式安装。但是需要注意的是Kibana的版本需要与ElasticSearch匹配，例如：ElasticSearch的版本为5.1.1时，Kibana的版本最好也为5.1.1，否则会有一定的兼容性问题。尤其是当ElasticSearch为5.0以上版本，而Kibana只有5.0以下版本时，反之亦然。安装过程如下：

首先，在Elastic官网官方下载页面下载[Kibana 5.1.1](https://www.elastic.co/downloads/past-releases/kibana-5-1-1)。也可通过命令行直接下载：

```
cd DIR_YOU_WANT_TO_PUT_KIBANA_IN
mkdir Kibana && cd Kibana
curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.1.1-darwin-x86_64.tar.gz
tar -xvf kibana-5.1.1-darwin-x86_64.tar.gz
cd kibana-5.1.1-darwin-x86_64
ls
```

以下为成功下载Kibana结果：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/kibana_success.png" width = "600" height = "400" />
</div>

然后进入`bin`目录启动Kibana即可：

```
cd bin
./kibana
```

以下是启动结果：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/start_kibana.png" width = "800" height = "600" />
</div>

Kibana的默认启动是5601，可以直接在浏览器通过访问`http://localhost:5601`进入Kibana页面，如下图所示：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/kibana.png" width = "800" height = "600" />
</div>

Kibana的配置文件为`config/kibana.yml`，可以通过修改该文件来对Kibana作出相应配置，比如：监听ElasticSearch的URL，启动Kibana的端口以及一些与ElasticSearch服务相关的配置等。

需要注意的是：

ElasticSearch与Kibana的启动先后顺序没有影响，因为Kibana会监听ElasticSearch的启动端口，如果ElasticSearch没有启动，或者ElasticSearch没有在Kibana所监听的端口启动，Kibana的在命令行会显示出`error`状态，如下图：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/kibana_error.png" width = "800" height = "600" />
</div>

上图显示，当ElasticSearch服务不可用时，Kibana会显示[error]说明连接错误，并且尝试重新连接，如果始终无法连接，会持续输出

```
[warning][elasticsearch] Unable to revive connection: http://localhost:9292/
[warning][elasticsearch] No living connections
```

<br />

## 在AWS上使用ElasticSearch Service搭建

(TBC)
