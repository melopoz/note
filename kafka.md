# kafka

百度网盘有pdf

### Topic 

kafka是发布/订阅模式的消息队列 (另一种是点对点消息队列，即一条消息被一个消费者消费一次就没了)

Kafka中消息是以topic进行分类的，生产者生产消息，消费者消费消息，都是面向topic的。topic是逻辑上的概念，

### Partition 分区

而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费。

由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位
效率低下，Kafka 采取了分片和索引机制，将每个 partition 分为多个 segment。每个 segment
对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名
规则为：topic 名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first-
0,first-1,first-2

index 和 log 文件以当前 segment 的第一条消息的 offset 命名:

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

“.index”文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元数据指向对应数据文件中 message 的物理偏移地址。

### Consumer group 组

一个topic的消息只能被同一个消费者组中的一个消费者消费。组内的消费者是竞争关系。

##### 作用

可以提高kafka的消费能力，当消费者组中的消费者个数(例如解析项目的线程个数)等于分区数的时候效率最高。再多了也消费不到消息。

## 配置

kafka下的config/server.properties是配置文件

```
log.dirs=/tmp/kafka-logs #这个目录下是可以看到kafka中的topic-分区号的 也就是说kafka的数据存储在硬盘
```

待补充待补充待补充待补充待补充待补充待补充待补充

## 命令

```
bin/kafka-server-start.sh config/server.properties #启动
bin/kafka-server-start.sh -daemon config/server.properties #后台启动
```

命令均在kafka的bin目录下

```
#创建topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
#查看topic列表
bin/kafka-topics.sh --list --zookeeper localhost:2181
#产生消息，创建消息生产者
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
#消费消息，创建消息消费者
bin/kafka-console-consumer.sh --broker-list localhost:9092 --topic test
#查看topic的消息
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
#删除topic
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
#修改分区数为6
bin/kafka-topics.sh --alter --partitions 6 --zookeeper localhost:2181 --topic test
```

