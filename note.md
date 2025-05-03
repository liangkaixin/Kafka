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

