---
layout: post
title: 'kafka入门使用'
subtitle:'kafka安装教程及在spring boot中的使用'
categories: 技术
tags:java kafka
---

## 1.kafka简介
kafka 是 Linkedin 公司用于日志处理的分布式消息队列，现归 Apache 维护，官网地址：[http://kafka.apache.org/](http://kafka.apache.org/)

kafka中几个消息系统术语：
* topic：用来分类消息
* producer：发布消息到kafka主题的进程
* consumer：订阅主题，消费消息的进程
* broker：kafka由多个服务器集群时，每个服务器为一个代理有自己的id

producer通过tcp协议发送消息到kafka集群，kafka集群向消费者提供消息，如下图所示：
![](http://www.aboutyun.com/data/attachment/forum/201505/02/225851j2s4eq67aq9llaol.png)

### 主题（topic）

一个topic相当于是一类消息，每个topic可以分为多个partition。任何发布到此partition的消息都会直接追加到分区尾部，每个消息有一个序列号称为offset。消息是有序且不可变的。
![](http://www.aboutyun.com/data/attachment/forum/201505/02/225851kqq1pnqbq81kblln.png)

kafka中消息可以设置保存时间，在时间期限内可以被消费，到期后会被删除释放空间。

consumer需要维护它在读取消息的位置即offset，可以随着offset增加顺序读取消息，也可以重置offset重读之前的消息。

### 分布式

每个分区在Kafka集群的若干服务中都有副本，这样这些持有副本的服务可以共同处理数据和请求，副本数量是可以配置的。副本使Kafka具备了容错能力。

每个分区都由一个服务器作为“leader”，零或若干服务器作为“followers”,leader负责处理消息的读和写，followers则去复制leader.如果leader down了，followers中的一台则会自动成为leader。集群中的每个服务都会同时扮演两个角色：作为它所持有的一部分分区的leader，同时作为其他分区的followers，这样集群就会据有较好的负载均衡

### consumer

kafka 中的发布-订阅模式消费者是以组为单位订阅服务的，同一组中，同一个分区只能被一个消费者消费。所以组中的消费者数量不能大于分区数量。
![](http://www.aboutyun.com/data/attachment/forum/201505/02/225852ng3ur3gmtc9v489o.png)
如上图，由两个机器组成的集群拥有4个分区 (P0-P3) 2个consumer组. A组有两个consumerB组有4个。

## 2.Linux下安装kafka

### 安装
可以从官网下载:[https://kafka.apache.org/downloads](https://kafka.apache.org/downloads)

命令下载：
```
wget http://mirrors.shuosc.org/apache/kafka/1.0.0/kafka_2.11-1.0.0.tgz
```
找个地方解压：
```
tar -zxvf kafka_2.11-1.0.0.tgz
```
解压文件夹中有个 config 文件夹，里面有相关配置文件。
可以修改 kafka-server 的配置文件
```
broker.id=1
log.dir=自己定义一个地址
```
### 启动

#### 1. 启动zookeeper

```
 bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```
#### 2. 启动kafka

```
bin/kafka-server-start.sh  config/server.properties
```
后台启动方式
```
bin/kafka-server-start.sh  config/server.properties 1>/dev/null 2>&1 &
```
## 3.kafka常用命令

### 创建 topic

创建 topic 主题
```
 bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
查看 topic 列表
```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```
### 发送消息

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```
### 消费消息

接收消息并打印
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
``` 

### 查看 topic 描述信息

```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
```

## 4.spring boot 整合 kafka 基本使用
> 参考该篇博客[http://www.54tianzhisheng.cn/2018/01/05/SpringBoot-Kafka/](http://www.54tianzhisheng.cn/2018/01/05/SpringBoot-Kafka/)

### 1.创建项目引入相关依赖

kafka 的依赖，json转换工具等

### 2.创建消息实体类
```java
@Data
public class Message {
    private Long id;    //id

    private String msg; //消息

    private Date sendTime;  //时间戳

}
```

### 3.创建消息发送者
```java
@Component
@Slf4j
public class KafkaSender {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private Gson gson = new GsonBuilder().create();

    //发送消息方法
    public void send() {
        Message message = new Message();
        message.setId(System.currentTimeMillis());
        message.setMsg(UUID.randomUUID().toString());
        message.setSendTime(new Date());
        log.info("+++++++++++++++++++++  message = {}", gson.toJson(message));
        kafkaTemplate.send("zhisheng", gson.toJson(message));
    }
}
```

### 4.创建消息接收者
```java
@Component
@Slf4j
public class KafkaReceiver {

    @KafkaListener(topics = {"zhisheng"})
    public void listen(ConsumerRecord<?, ?> record) {

        Optional<?> kafkaMessage = Optional.ofNullable(record.value());

        if (kafkaMessage.isPresent()) {

            Object message = kafkaMessage.get();

            log.info("----------------- record =" + record);
            log.info("------------------ message =" + message);
        }

    }
}

```
### 5.配置文件

```
#============== kafka ===================
# 指定kafka 代理地址，可以多个
spring.kafka.bootstrap-servers=192.168.153.135:9092

#=============== provider  =======================

spring.kafka.producer.retries=0
# 每次批量发送消息的数量
spring.kafka.producer.batch-size=16384
spring.kafka.producer.buffer-memory=33554432

# 指定消息key和消息体的编解码方式
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

#=============== consumer  =======================
# 指定默认消费者group id
spring.kafka.consumer.group-id=test-consumer-group

spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=true
spring.kafka.consumer.auto-commit-interval=100

# 指定消息key和消息体的编解码方式
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```





