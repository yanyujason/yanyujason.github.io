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

本文主要介绍如何在Apple Mac操作系统以及[AWS(Amazon Web Services)](https://aws.amazon.com/cn/)上快速搭建一套ElasticSearch与Kibana的系统。

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

__需要注意的是：__

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

AWS提供了一套封装好的ElasticSearch服务，可以直接登录AWS Console或者使用CloudFormation配置。

### AWS Console配置

#### 登录AWS Console

登录AWS Console并选择ElasticSearch Service。部分地区比如Canada(Central)和EU(London)不支持ElasticSearch Service服务，所以需要选择合适的AWS地区。这里选择Asia Pacific(Tokyo)进行配置。

如下图所示，填写域名(ElasticSearch domain name)，比如`es-testing`，并选择版本号，这里选择`5.1`（AWS目前只支持ElasticSearch5.1，2.3和1.5版本）。

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/aws_es_first.png" width = "800" height = "600" />
</div>

#### 配置集群

在`集群配置（Configure cluster）`页面填写结点配置(Node configuration)，存储配置(Storage configuration)，快照配置（Snapshot configuration）。

__1.结点配置:__

__实例个数(Instance count):__ 集群结点个数

__实例类型(Instance type):__ 配置数据结点机器的大小，不同大小的机器运算速率、内存、硬盘大小都是不同的，需要考虑ElasticSearch的索引大小、备份和搜索查询的类型来决定机器的大小。这里选择`t2.small.elasticsearch(Free tier eligible)`。这种结点类型的机器在这里是免费的，包含每个月750小时的`t2.small`类型机器的使用，或者上限为10GB弹性块存储(EBS)。

__开启专属主结点:__ 配置专用结点作为主结点，一般只在产品环境上使用。默认推荐值为3。

__开启地域限制:__ 配置ElasticSearch是否可以被别的AWS地区的服务访问。

__2.存储配置:__

__存储类型(Storage type):__ 选择数据结点的存储类型，实例默认存储或者EBS。

__EBS容量类型(EBS volume type)__ EBS容量可以通过计算资源对存储资源进行扩容。EBS容量主要用于对不需要大量计算的大数据集合的存储。

__EBS容量大小(EBS volume size)__ 配置EBS容量大小，默认为10GB。可以使用Minimum {MIN} GB与Maximum {MAX} GB来配置上下限。

__3.快照配置:__

__自动快照开始时间(Automated snapshot start hour)__ 用于配置ElasticSearch做快照的时间，为UTC时间。默认值0为午夜。例如：设置为`19：00 UTC`是北京时间凌晨3点。

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/aws_es_second.png" width = "800" height = "600" />
</div>

#### 配置访问权限

点击`Next`进入`设置访问权限(Set up access policy)`页面，访问权限的设定定义了账户、请求类型对ElasticSearch的访问权限。比如在下拉菜单中选择`Allow open access to the domain`选项，表示放开所有访问权限，也就是任何服务都可以对ElasticSearch进行任何（增删改查）的操作。如下图：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/aws_es_third.png" width = "800" height = "600" />
</div>

当然，也可以根据自己需要，自定义访问权限。以下图为例：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/aws_es_access.png" width = "800" height = "600" />
</div>

图中：

> `Effect`表示对权限是“允许”还是“拒绝”状态。

> `Principal`设置了哪些IAM下的用户可以访问该ElasticSearch，这里的IAM账号为“MY_ACCOUNT_IDENTITY”，因为安全问题这里做了隐藏，正常情况下为12位数字。

> `Action`表示允许哪种请求方式访问，这里设置为所有的HTTP请求，包括：Delete，GET，HEAD，POST，PUT等请求。

> `Resource`表示哪种资源可以被请求，这里表示es-testing下的所有资源都可以被请求。

#### 完成配置

点击`Next`进入`复审(Review)`页面，里面罗列了所有之前关于ElasticSearch的配置信息，检查无误后点击`Confirm and create`按钮完成配置。

这是页面会有创建成功的消息提示并且显示`Domain status: Loading`，需要花一定时间(一般为10分钟)来创建ElasticSearch服务。最后，创建好的ElasticSearch服务会如下显示：

<div style="text-align:center" markdown="1">
<img src="/images/blogs/elasticsearch/aws_es_success.png" width = "800" height = "600" />
</div>

图中的`Endpoint`显示ElasticSearch服务的访问终端，`Kibana`为AWS为ElasticSearch服务提供的Kibana访问终端，点击即可进入。

可以点击`Configure cluster`, `Modify access policy`和`Manage tags`来对之前的配置做修改。也可以点击`Delete ElasticSearch domain`来删除服务。


### 使用CloudFormation配置

如果对AWS CloudFormation非常熟悉，并且需要自动化创建或者修改ElasticSearch服务，可以直接使用CloudFormation对ElasticSearch进行配置，配置内容与在AWS Console的配置一致，这里直接采用`yml`格式写出所有ElasticSearch配置。

```
ElasticsearchDomain:
    Type: AWS::Elasticsearch::Domain
    Properties:
      DomainName: es-testing
      ElasticsearchVersion: '5.1'
      ElasticsearchClusterConfig:
        DedicatedMasterEnabled: false
        InstanceCount: 1
        ZoneAwarenessEnabled: false
        InstanceType: t2.small.elasticsearch
        DedicatedMasterType: t2.micro.elasticsearch
        DedicatedMasterCount: 1
      EBSOptions:
        EBSEnabled: 'true'
        VolumeSize: 10
        VolumeType: gp2
        Iops: 0
      SnapshotOptions:
        AutomatedSnapshotStartHour: '19'
      AccessPolicies:
        Version: '2012-10-17'
        Statement:
        - Action: es:ESHttp*
          Resource: !Sub 'arn:aws:es:ap-northeast-1:${MY_ACCOUNT_IDENTITY}:domain/es-testing/*'
          Effect: Allow
          Principal:
            AWS: 'arn:aws:iam::${MY_ACCOUNT_IDENTITY}:root'
      AdvancedOptions:
        rest.action.multi.allow_explicit_index: 'true'
```

详细关于AWS ElasticSearch服务的配置参见[官方文档](http://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-createupdatedomains.html)
