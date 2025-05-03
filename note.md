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
## 参考资料
[kafka是什么--小白debug](https://mp.weixin.qq.com/s/SNMmCMV-gqkHtWS0Ca3j4g)