![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2ccd794db76e482680f72d60959cf368~tplv-k3u1fbpfcp-zoom-1.image)

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Author-3y-orange.svg" alt="作者"></a>
  <a href="https://gitee.com/zhongfucheng/austin"><img src="https://gitee.com/zhongfucheng/austin/badge/star.svg?theme=dark" alt="gitee Starts"></a>
  <a href="https://gitee.com/zhongfucheng/austin"><img src="https://gitee.com/zhongfucheng/austin/badge/fork.svg?theme=dark" alt="gitee Starts"></a>
  <a href="https://github.com/ZhongFuCheng3y/austin"><img src="https://img.shields.io/github/forks/ZhongFuCheng3y/austin.svg?style=flat&label=GithubFork"></a> 
  <a href="https://github.com/ZhongFuCheng3y/austin"><img src="https://img.shields.io/github/stars/ZhongFuCheng3y/austin.svg?style=flat&label=GithubStars"></a> 
  <a href="#如何准备面试"><img src="https://img.shields.io/badge/如何准备-面试-yellow.svg" alt="对线面试官"></a>
</p>


## 项目介绍

austin项目**核心功能**：发送消息

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5436b2e3d6cd471db9aafbd436198ca7~tplv-k3u1fbpfcp-zoom-1.image)

**项目出现意义**：只要公司内有发送消息的需求，都应该要有类似`austin`的项目，对各类消息进行统一发送处理。这有利于对功能的收拢，以及提高业务需求开发的效率

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c267ebb2ff234243b8665312dbb46310~tplv-k3u1fbpfcp-zoom-1.image)

## 系统项目架构

austin项目**核心流程**：`austin-api`接收到发送消息请求，直接将请求进`MQ`。`austin-handler`消费`MQ`消息后由各类消息的Handler进行发送处理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cce7979291e740a39c5f00f1cee8c214~tplv-k3u1fbpfcp-zoom-1.image)

**Question** ：为什么发个消息需要MQ？

**Answer**：发送消息实际上是调用各个服务提供的API，假设某消息的服务超时，`austin-api`如果是直接调用服务，那存在**超时**风险，拖垮整个接口性能。MQ在这是为了做异步和解耦，并且在一定程度上抗住业务流量。

**Question**：能简单说下接入层做了什么事吗？

**Answer**：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c94059a008784a69bd10b98caa46d683~tplv-k3u1fbpfcp-zoom-1.image)

**Question**：`austin-stream`和`austin-datahouse`的作用？

**Answer**：`austin-handler`在发送消息的过程中会做些**通用业务处理**以及**发送消息**，这个过程会产生大量的日志数据。日志数据会被收集至MQ，由`austin-stream`流式处理模块进行消费并最后将数据写入至`austin-datahouse`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e4bd420001c549ebad922637f7b2e38a~tplv-k3u1fbpfcp-zoom-1.image)

**Question**：`austin-admin`和`austin-cron`的作用？

**Answer**：`autsin-admin`是`austin`项目的**管理后台**，负责管理消息以及查看消息下发的情况。业务方可根据通过`austin-admin`管理后台直接**定时**发送消息，而`austin-cron`就是承载着定时任务的工作了。

## 使用姿势

目前引用的中间件教程的安装姿势均基于`Centos 7.6`，austin项目**强依赖**`MySQL`/`Redis`/`Kafka`/`apollo`，**弱依赖**`prometheus`/`graylog`。如果缺少相关的组件可戳：[安装相关组件教程](INSTALL.md)。

**1**、austin使用的MySQL版本**5.7x**。如果目前使用的MySQL版本8.0，注意改变`pom.xml`所依赖的版本

**2**、适配`application.yml`的配置信息(`srping.datasource`)

**3**、执行`sql`文件夹下的`austin.sql`创建对应的表

**4**、填写Kafka配置的`bootstrap-servers`地址和端口以及对应的`topicName`

**5**、填写Redis的`host`、`port`和`password`

**6**、填写apollo的`appid`/`namespace`

**7**、以上配置信息都在`application.yml`文件中修改。

**8**、由于使用了Apollo且我是在云服务器上安装的，我这边会直接跳过`metaserver`服务发现，在`AustinApplication`需要配置对应的apollo地址

**9**、目前短信和邮件账号的信息都配置在**apollo**，配置的示例参照`com.java3y.austin.utils.AccountUtils#getAccount`中的注释

**10**、调用http接口`com.java3y.austin.controller.SendController#send`给自己发一条短信或者邮件感受

```shell

curl -XPOST "127.0.0.1:8080/send"  -H 'Content-Type: application/json'  -d '{"code":"send","messageParam":{"receiver":"13788888888","variables":{"title":"yyyyyy","contentValue":"6666164180"}},"messageTemplateId":1}'

```

## 将要实现的项目架构模块

2021-11~2021-12实现功能：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2edbaed45c3471d946c09bd829d936b~tplv-k3u1fbpfcp-zoom-1.image)

实现功能所需引入的技术栈：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4b200a3e5ab2450f9123756ac7cc7cf2~tplv-k3u1fbpfcp-zoom-1.image)

## 已完成内容


![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f99631fe25c42b39cbfb6e59cccec85~tplv-k3u1fbpfcp-watermark.image?)

**近期更新时间**：2022年1月17日

**近期更新功能**：完成接入graylog

## 项目交流

可以添加我的**个人微信**备注：austin拉进项目交流群


<img align="center" src='https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5eae548196934599a7cb3637aedf381d~tplv-k3u1fbpfcp-zoom-1.image' width=300px height=300px />

**Java3y**公众号在持续更新austin系列文章，**保姆级**讲解搭建项目的过程（包括技术选型以及一些业务的探讨）以及相关环境的搭建。**扫下面的码直接关注，带你了解整个项目**


如果你需要用这个项目写在简历上，**强烈建议关注公众号看实现细节的思路**。如果⽂档中有任何的不懂的问题，都可以直接来找我询问，我乐意帮助你们！公众号下有我的联系方式

<img align="center" src='https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e109cdb8d064c1e87541d7b6c17957d~tplv-k3u1fbpfcp-zoom-1.image' width=300px height=300px />

## 如何准备面试？

**对线面试官**公众号持续更新**面试系列**文章（对线面试官系列），深受各大开发的好评，已有不少的同学通过对线面试官系列得到BATTMD等一线大厂的的offer。一个**讲人话的面试系列**，八股文不再是背诵。

<img align="center" src='https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f87f574e93964921a4d02146bf3ccdac~tplv-k3u1fbpfcp-zoom-1.image' width=300px height=300px />


