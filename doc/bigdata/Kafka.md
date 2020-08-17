# Kafka

[TOC]

## 部署 Kafka 环境

### 1. 安装 zookeeper 

```shell
$ docker pull wurstmeister/zookeeper
$ docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
```

### 2. 安装 Kafka

```shell
$ ifconfig | grep inet
	inet 127.0.0.1 netmask 0xff000000 
	inet6 ::1 prefixlen 128 
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
	inet 192.168.10.54 netmask 0xffffff00 broadcast 192.168.10.255

$ docker pull wurstmeister/kafka
$ docker run -d --name kafka -p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=192.168.10.54:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.10.54:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-t wurstmeister/kafka
```

### 3. 测试消息

```shell
# 进入容器
$ docker exec -it kafka bash
$ cd opt/kafka/bin/
# 启动生产者
$ ./kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka
# 启动消费者
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mykafka --from-beginning
```

## Java 连接 Kafka 

### 1. 配置 pom.xml

> kafka-clients 的版本必须和 kafka 安装的版本一致

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>1.1.0</version>
</dependency>
```

### 2. 消费者代码

```java
package demo;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

public class Consumer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.10.60:9092");
        props.put("group.id", "group-1");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("auto.offset.reset", "earliest");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        // 创建消费者并注册 topic
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList("topic01"));
        System.out.println("准备接收。。。");

        // 接收消息
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("offset = %d, value = %s", record.offset(), record.value());
            }
        }

    }
}
```

### 3. 生产者代码

```java
package demo;

import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;


public class Producter {
    public static void main(String[] args) {
        // Kafka 配置
        Properties props = new Properties();
        props.put("bootstrap.servers", "192.168.10.54:9092");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        try {
            // 创建生产者
            Producer<String, String> producer = new KafkaProducer<String, String>(props);
            System.out.println("准备发送。。。");

            for (int i = 0; i < 100; i++) {
                producer.send(new ProducerRecord<String, String>("topic01",  "This is Message " + i));
                System.out.println("已发送。。。" + i);
            }
        } catch (Exception e) {
            e.printStackTrace();

        }
    }
}
```

