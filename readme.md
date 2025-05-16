## kafka简述
Apache Kafka是一个分布式流平台。一个分布式的流平台应该包含3点关键的能力：  
1.**发布和订阅**流数据流，类似于消息队列或者是企业消息传递系统  
2.以容错的持久化方式存储数据流  
3.处理数据流

## kafka队列
![img.png](img.png)
partition内部是一个FIFO队列， 所以消息在partition内有序。  
但无法保证数据整体有序

## kafka架构
![img_1.png](img_1.png)

## kafka幂等性原理
![img_3.png](img_3.png)
消费者端引入了序列信息来保证重复到达的数据的一致性  
·  PID：每个Producer在初始化时，都会分配一个唯一的PID，这个PID对用户来说，是透明的。  
·  Sequence Number：针对每个生产者（对应PID）发送到指定主题分区的消息都对应一个从0开始递增的Sequence Number。  
## kafka实战
### 生产者
1. 配置编写
2. 创建生产者
3. 定义发送消息的主题和消息内容
4. 调用 ProducerRecord.send(topic, message)发送消息
5. 关闭生产者
```java
public class KafkaProducerTest {

    public static void main(String[] args) {
        // Configuration for connecting to Kafka
        Properties properties = new Properties();

        // We create the object Producer
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        // We are creating a message for sending to Kafka.
        String topic = "your_topic";  // The topic to which we are sending the message
        String message = "your_message"; // Message text

        // Creating and sending a message
        ProducerRecord<String, String> record = new ProducerRecord<>(topic, message);
        producer.send(record);

        producer.close();
    }
}
```
### 消费者
#### 直接消费
1. 配置编写
2. 创建消费者
3. 调用 KafkaConsumer.subscribe(topic)订阅消息的主题
4. 调用 KafkaConsumer.poll(wait duration)接收消息
5. 关闭消费者
```java
public class KafkaConsumerTest {
    public static void main(String[] args) {
        // Kafka Consumer configuration
        Properties properties = new Properties();

        // Create Kafka Consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // Subscribe to topic
        String topic = "your_topic";
        consumer.subscribe(Collections.singletonList(topic));
        
        while (true) {
            // Poll for messages
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(5000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf(
                        "Received message: key = %s, value = %s, partition = %d, offset = %d%n",
                        record.key(), record.value(), record.partition(), record.offset()
                );
            }
        }
        consumer.close();
    }
    
}
```
Received message: key = null, value = your_message, partition = 0, offset = 0  
Received message: key = null, value = your_message, partition = 0, offset = 1

### 通过offset消费
1. 配置编写
2. 创建消费者
3. 定义 具体分区TopicPartition和对应偏移量offset
4. 调用 KafkaConsumer.assign(TopicPartition)订阅消息的主题
5. 设置开始消费的位置 KafkaConsumer.seek(partition, offset);
6. 调用 KafkaConsumer.poll(wait duration)接收消息
7. 关闭消费者
```java
public class KafkaConsumerTest {
    public static void main(String[] args) {
        // Kafka Consumer configuration
        Properties properties = new Properties();
        // Create Kafka Consumer
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // Subscribe to topic
        String topic = "your_topic";

        TopicPartition partition = new TopicPartition(topic, 0); // Read from partition 0
        long offset = 3; // Specify the offset from which to start reading

        // Subscribe to a specific partition
        consumer.assign(Collections.singletonList(partition));

        // Set the starting offset
        consumer.seek(partition, offset);
        
        while (true) {
            // Poll for messages
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(5000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf(
                        "Received message: key = %s, value = %s, partition = %d, offset = %d%n",
                        record.key(), record.value(), record.partition(), record.offset()
                );
            }
        }
        // Close the consumer
        consumer.close();
    }
}
```
Received message: key = null, value = your_message, partition = 0, offset = 3  
Received message: key = null, value = your_message, partition = 0, offset = 4

## Kafka + bend-ingest
![img_7.png](bend_ingest架构图.png)
1. docker 搭建kafka集群
    ```shell
    docker rm -f kafka && docker run -d --name kafka -p 9092:9092 -e KAFKA_ENABLE_KRAFT=yes -e KAFKA_CFG_NODE_ID=1 -e KAFKA_CFG_PROCESS_ROLES=broker,controller -e KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER -e KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093 -e KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT -e KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 -e KAFKA_BROKER_ID=1 -e KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@localhost:9093 -e ALLOW_PLAINTEXT_LISTENER=yes bitnami/kafka:3.5.1
    ```
   步骤 2：创建 Topic 并生产消息
    创建一个名为 topic 的新 Kafka topic：
    ```shell
   kafka-console-producer --bootstrap-server localhost:9092 --topic test
   ```
   输入JSON格式的消息
   ```shell
    {"id": 1, "name": "Alice", "age": 30}
    {"id": 2, "name": "Bob", "age": 25}
    ```
   消费者
    ```shell
    kafka-console-consumer --bootstrap-server localhost:9092 --topic test --from-beginning    
    ```
2. 安装bend-ingest, 配置写入config/conf.json
   ```sh
    go install  github.com/databendcloud/bend-ingest-kafka@latest
    ```
   ```
    {
    "kafkaBootstrapServers": "localhost:9092",
    "kafkaTopic": "test",
    "KafkaConsumerGroup": "test",
    "isJsonTransform": false,
    "databendDSN": "databend://ENGINEERPIPELINE:rDxRFskx3ouu9aNGo2Um@tn9tdlpwf--bigdata-ta-event-cdc-xsmall-prod.gw.aws-us-east-1.aviagames.databend.com:443/production_ta_event?warehouse=bigdata-ta-event-cdc-xsmall-prod",
    "databendTable": "poc.kfk_test",
    "batchSize": 100000,
    "batchMaxInterval": 5,
    "dataFormat": "json",
    "workers": 1,
    "copyPurge": false,
    "copyForce": false,
    "MaxBytes": 100000000
    }       
   ```
3. 启动bend-ingest 消费kafka的数据，自动落表
    ```shell
    ./bend-ingest-kafka
    ```



## 面试题
1. 零拷贝  
    利用了DMA<sup>[1]</sup>机制，能让网卡直接访问内存中的数据。  
    发送数据时，不需要通过cpu调度，避免了数据从  
    内核态(read_buffer)->用户态->内核态(socket buffer)->网卡的多次复制。  
    ![img_4.png](img_4.png)

## 参考资料
[1] DMA，全称叫Direct Memory Access，一种可让某些硬件子系统去直接访问系统主内存，而不用依赖CPU的计算机系统的功能。 跳过CPU，直接访问主内存。
<div style="text-align: center;">
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="img_5.png" alt="">
    <br>
    <div style="border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">传统的文件拷贝</div>
</div>


[kafka是什么--小白debug](https://mp.weixin.qq.com/s/SNMmCMV-gqkHtWS0Ca3j4g)
[bend-ingest 项目代码](https://github.com/databendcloud/bend-ingest-kafka)