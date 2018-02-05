---
title: Docker下的日志管理
tags: Docker
---
今天来记录下Docker中日志的管理问题，作为一个可用的系统，日志是必须要有的。在Docker中是，日志的输出形式有两种，一种是标准输出`STDOUT`，一种是文件输出。

文件输出，是指输出到日志文件，每个应用的日志文件保存在不一样的地方。

标准输出`STDOUT`，Unix和Linux在程序运行的时候，都会产生[三个文件](Linux Documentation Project article on I/O redirection.)，分别是`STDIN`、`STDOUT`和`STDERR`，默认情况下，docker是打开`STDOUT`和`STDERR`的。

容器有一个处理标准输出的模块LogDriver来处理日志，[Docker本身支持很多的LogDriver](https://docs.docker.com/engine/admin/logging/overview/#supported-logging-drivers)。容器默认支持的`json-file`的方式所谓的LogDriver就是一个日志的转发模块，转发到其他地方进行处理，之所以这样子做，是为了不在所有的容器或者节点机器上去配置日志的处理，这样可有做到统一处理了。我这里讲两种配置方式，一种是直接配置容器的`daemon.josn`文件![image.png](http://upload-images.jianshu.io/upload_images/8433-515b929b67fbc355.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另一种是在Dockerfile中增加
```docker
CMD run --log-driver syslog --log-opt syslog-address=udp://1.2.3.4:1111
```
# 在阿里云的容器服务中
在阿里云的容器服务中，可以直接打开容器集群的日志服务，然后就会直接投递到阿里云上面的日志服务，见[连接](https://help.aliyun.com/document_detail/26036.html?spm=5176.doc55339.6.650.lrKXid)，需要注意的是，在编排文件中添加标签的格式为 `aliyun.log_store_{name}: {logpath}`。其中 `name` 为阿里云日志服务 `logstore` 的名字，实际创建的 `logstore` 的名字为`acslog-${app}-${name}`。`app` 为应用名称；`logpath`为容器中日志的路径；`stdout` 是一个特殊的 `logpath`，表示标准输出。然后有了这个标签之后，日志会自动投递到日志服务中。

# ELK（Elasticsearch, Logstash, Kibana）

这个没有去试过，后续更新
