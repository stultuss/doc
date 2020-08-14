# Docker 部署本地测试环境

1. 安装 zookeeper 

```shell
$ docker pull wurstmeister/zookeeper
$ docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
```

2. 安装 Kafka

```shell
$ docker pull wurstmeister/kafka
$ docker run -d --name kafka -p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.10.60:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.10.60:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-t wurstmeister/kafka
```

3. 测试消息

```shell
# 进入容器
$ docker exec -it kafka bash
$ cd opt/kafka/bin/
# 启动生产者
$ ./kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
# 启动消费者
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning
```

