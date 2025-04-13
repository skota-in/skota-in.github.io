# Configuring Apache Kafka with Spring Boot

*Published on May 15, 2023*

## Introduction

Apache Kafka is a distributed streaming platform that allows you to build real-time data pipelines and streaming applications. When combined with Spring Boot, it provides a powerful foundation for building event-driven microservices. This guide will walk you through the process of configuring and using Apache Kafka with Spring Boot.

## Table of Contents

1. [Understanding Kafka Concepts](#understanding-kafka-concepts)
2. [Setting Up Kafka](#setting-up-kafka)
3. [Spring Boot Kafka Integration](#spring-boot-kafka-integration)
4. [Configuring Kafka Producer](#configuring-kafka-producer)
5. [Configuring Kafka Consumer](#configuring-kafka-consumer)
6. [Working with Kafka Topics](#working-with-kafka-topics)
7. [Serialization and Deserialization](#serialization-and-deserialization)
8. [Error Handling](#error-handling)
9. [Testing Kafka Components](#testing-kafka-components)
10. [Monitoring and Management](#monitoring-and-management)
11. [Best Practices](#best-practices)
12. [Advanced Topics](#advanced-topics)

## Understanding Kafka Concepts

Before diving into the implementation, let's understand some key Kafka concepts:

### Topics

A topic is a category or feed name to which records are published. Topics in Kafka are always multi-subscriber; that is, a topic can have zero, one, or many consumers that subscribe to the data written to it.

### Partitions

Topics are divided into partitions, which allow for parallel processing of data. Each partition is an ordered, immutable sequence of records.

### Producers

Producers publish data to topics of their choice. The producer is responsible for choosing which record to assign to which partition within the topic.

### Consumers

Consumers read data from topics. Consumers belong to a consumer group, and each record published to a topic is delivered to one consumer instance within each subscribing consumer group.

### Brokers

Kafka is run as a cluster of one or more servers, each of which is called a broker. Brokers are responsible for maintaining published data.

## Setting Up Kafka

### Local Development Environment

For local development, you can use Docker to run Kafka:

```yaml
# docker-compose.yml
version: '3'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: localhost
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: "example-topic:1:1"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Start the containers:

```bash
docker-compose up -d
```

### Kafka CLI Tools

Kafka comes with command-line tools for management:

```bash
# Create a topic
kafka-topics.sh --create --topic example-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# List topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# Describe a topic
kafka-topics.sh --describe --topic example-topic --bootstrap-server localhost:9092

# Produce messages
kafka-console-producer.sh --topic example-topic --bootstrap-server localhost:9092

# Consume messages
kafka-console-consumer.sh --topic example-topic --from-beginning --bootstrap-server localhost:9092
```

## Spring Boot Kafka Integration

### Dependencies

Add the following dependencies to your `pom.xml`:

```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
    <!-- Spring Kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
    
    <!-- For JSON serialization/deserialization -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
    
    <!-- Lombok (optional) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### Basic Configuration

Configure Kafka in your `application.properties` or `application.yml`:

```properties
# Kafka Producer Configuration
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

# Kafka Consumer Configuration
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=my-group
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

Or using YAML:

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      bootstrap-servers: localhost:9092
      group-id: my-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

## Configuring Kafka Producer

### Basic Producer

Create a simple Kafka producer:

```java
@Service
public class KafkaProducerService {
    
    private static final Logger logger = LoggerFactory.getLogger(KafkaProducerService.class);
    
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    public void sendMessage(String topic, String message) {
        logger.info("Sending message to topic {}: {}", topic, message);
        kafkaTemplate.send(topic, message);
    }
}
```

### Producer with Callback

Add a callback to handle the result of the send operation:

```java
public void sendMessageWithCallback(String topic, String message) {
    logger.info("Sending message to topic {}: {}", topic, message);
    
    ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, message);
    
    future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
        @Override
        public void onSuccess(SendResult<String, String> result) {
            logger.info("Message sent successfully to topic {} with offset {}", 
                    topic, result.getRecordMetadata().offset());
        }
        
        @Override
        public void onFailure(Throwable ex) {
            logger.error("Failed to send message to topic {}: {}", topic, ex.getMessage());
        }
    });
}
```

### Producer with Key

Send messages with a key to ensure that messages with the same key go to the same partition:

```java
public void sendMessageWithKey(String topic, String key, String message) {
    logger.info("Sending message to topic {} with key {}: {}", topic, key, message);
    kafkaTemplate.send(topic, key, message);
}
```

### Custom Producer Configuration

For more control, you can create a custom producer configuration:

```java
@Configuration
public class KafkaProducerConfig {
    
    @Value("${spring.kafka.producer.bootstrap-servers}")
    private String bootstrapServers;
    
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // Additional configuration
        configProps.put(ProducerConfig.ACKS_CONFIG, "all"); // Strongest guarantee
        configProps.put(ProducerConfig.RETRIES_CONFIG, 3); // Retry on failure
        configProps.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384); // Batch size in bytes
        configProps.put(ProducerConfig.LINGER_MS_CONFIG, 1); // Linger up to 1ms before sending
        configProps.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // 32MB buffer
        
        return new DefaultKafkaProducerFactory<>(configProps);
    }
    
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

## Configuring Kafka Consumer

### Basic Consumer

Create a simple Kafka consumer:

```java
@Service
public class KafkaConsumerService {
    
    private static final Logger logger = LoggerFactory.getLogger(KafkaConsumerService.class);
    
    @KafkaListener(topics = "example-topic", groupId = "my-group")
    public void listen(String message) {
        logger.info("Received message: {}", message);
        // Process the message
    }
}
```

### Consumer with Topic Partitions

Listen to specific partitions of a topic:

```java
@KafkaListener(topicPartitions = {
    @TopicPartition(topic = "example-topic", partitions = {"0", "1"})
})
public void listenToPartitions(String message) {
    logger.info("Received message from partitions 0 or 1: {}", message);
    // Process the message
}
```

### Consumer with Concurrency

Process messages concurrently:

```java
@KafkaListener(topics = "example-topic", groupId = "my-group", concurrency = "3")
public void listenConcurrently(String message) {
    logger.info("Received message (concurrent): {}", message);
    // Process the message
}
```

### Manual Acknowledgment

Manually acknowledge messages:

```java
@KafkaListener(topics = "example-topic", groupId = "my-group", 
               containerFactory = "kafkaListenerContainerFactory")
public void listenWithAck(String message, Acknowledgment acknowledgment) {
    try {
        logger.info("Received message: {}", message);
        // Process the message
        acknowledgment.acknowledge(); // Acknowledge the message
    } catch (Exception e) {
        logger.error("Error processing message: {}", e.getMessage());
        // Handle the error (retry, dead letter queue, etc.)
    }
}
```

### Custom Consumer Configuration

For more control, you can create a custom consumer configuration:

```java
@Configuration
public class KafkaConsumerConfig {
    
    @Value("${spring.kafka.consumer.bootstrap-servers}")
    private String bootstrapServers;
    
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    
    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // Additional configuration
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false); // Disable auto-commit
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 100); // Max records per poll
        
        return new DefaultKafkaConsumerFactory<>(props);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        factory.setConcurrency(3); // Number of concurrent threads
        return factory;
    }
}
```

## Working with Kafka Topics

### Creating Topics Programmatically

Create Kafka topics programmatically:

```java
@Configuration
public class KafkaTopicConfig {
    
    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;
    
    @Bean
    public KafkaAdmin kafkaAdmin() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        return new KafkaAdmin(configs);
    }
    
    @Bean
    public NewTopic exampleTopic() {
        return TopicBuilder.name("example-topic")
                .partitions(3)
                .replicas(1)
                .build();
    }
    
    @Bean
    public NewTopic dltTopic() {
        return TopicBuilder.name("example-topic.DLT")
                .partitions(1)
                .replicas(1)
                .build();
    }
}
```

### Using Spring Boot's Auto-Configuration

Spring Boot can automatically create topics defined in the configuration:

```properties
spring.kafka.admin.auto-create=true
spring.kafka.admin.properties.cleanup.policy=compact
spring.kafka.admin.properties.min.insync.replicas=2
```

## Serialization and Deserialization

### JSON Serialization

Configure JSON serialization for complex objects:

```properties
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example.model
```

### Message Model

Create a model class for your messages:

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
    private String email;
}
```

### JSON Producer

Create a producer for JSON messages:

```java
@Service
public class UserKafkaProducer {
    
    private static final Logger logger = LoggerFactory.getLogger(UserKafkaProducer.class);
    
    @Autowired
    private KafkaTemplate<String, User> kafkaTemplate;
    
    public void sendUser(String topic, User user) {
        logger.info("Sending user to topic {}: {}", topic, user);
        kafkaTemplate.send(topic, user);
    }
}
```

### JSON Consumer

Create a consumer for JSON messages:

```java
@Service
public class UserKafkaConsumer {
    
    private static final Logger logger = LoggerFactory.getLogger(UserKafkaConsumer.class);
    
    @KafkaListener(topics = "user-topic", groupId = "user-group")
    public void listen(User user) {
        logger.info("Received user: {}", user);
        // Process the user
    }
}
```

### Custom Serializer/Deserializer

For more control, you can create custom serializers and deserializers:

```java
public class CustomUserSerializer implements Serializer<User> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public byte[] serialize(String topic, User data) {
        try {
            return objectMapper.writeValueAsBytes(data);
        } catch (Exception e) {
            throw new SerializationException("Error serializing User", e);
        }
    }
}

public class CustomUserDeserializer implements Deserializer<User> {
    
    private final ObjectMapper objectMapper = new ObjectMapper();
    
    @Override
    public User deserialize(String topic, byte[] data) {
        try {
            return objectMapper.readValue(data, User.class);
        } catch (Exception e) {
            throw new SerializationException("Error deserializing User", e);
        }
    }
}
```

## Error Handling

### Error Handler

Configure an error handler for consumer exceptions:

```java
@Configuration
public class KafkaErrorHandlingConfig {
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
            ConsumerFactory<String, String> consumerFactory,
            KafkaTemplate<String, String> kafkaTemplate) {
        
        ConcurrentKafkaListenerContainerFactory<String, String> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory);
        
        // Configure error handler
        factory.setErrorHandler(new SeekToCurrentErrorHandler(
            new DeadLetterPublishingRecoverer(kafkaTemplate,
                (record, ex) -> new TopicPartition(record.topic() + ".DLT", record.partition())),
            new FixedBackOff(1000L, 3))); // Retry 3 times with 1s delay
        
        return factory;
    }
}
```

### Exception Handling in Listener

Handle exceptions in the listener method:

```java
@KafkaListener(topics = "example-topic", groupId = "my-group")
public void listen(String message, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                  @Header(KafkaHeaders.OFFSET) long offset) {
    try {
        logger.info("Received message: {}", message);
        // Process the message
        if (message.contains("error")) {
            throw new RuntimeException("Simulated error");
        }
    } catch (Exception e) {
        logger.error("Error processing message from topic {}, partition {}, offset {}: {}",
                topic, partition, offset, e.getMessage());
        // The error handler will handle retries and DLT publishing
    }
}
```

### Dead Letter Topic

Configure a dead letter topic for failed messages:

```java
@Bean
public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> 
        kafkaListenerContainerFactory(ConsumerFactory<String, String> consumerFactory,
                                     KafkaTemplate<String, String> kafkaTemplate) {
    
    ConcurrentKafkaListenerContainerFactory<String, String> factory = 
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory);
    
    // Configure DLT
    DeadLetterPublishingRecoverer recoverer = new DeadLetterPublishingRecoverer(kafkaTemplate,
        (record, ex) -> {
            if (ex.getCause() instanceof SerializationException) {
                return new TopicPartition("deserialization-errors", record.partition());
            } else {
                return new TopicPartition(record.topic() + ".DLT", record.partition());
            }
        });
    
    // Configure error handler with DLT
    SeekToCurrentErrorHandler errorHandler = new SeekToCurrentErrorHandler(
        recoverer, new FixedBackOff(1000L, 3));
    
    factory.setErrorHandler(errorHandler);
    
    return factory;
}
```

## Testing Kafka Components

### Testing with Embedded Kafka

Use an embedded Kafka broker for testing:

```java
@SpringBootTest
@EmbeddedKafka(partitions = 1, topics = {"example-topic"})
public class KafkaProducerServiceTest {
    
    @Autowired
    private KafkaProducerService producerService;
    
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;
    
    @Test
    public void testSendMessage() throws InterruptedException {
        // Arrange
        String message = "Test message";
        CountDownLatch latch = new CountDownLatch(1);
        AtomicReference<String> receivedMessage = new AtomicReference<>();
        
        // Create a consumer
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps(groupId, "true", 
                                                                       embeddedKafka);
        ConsumerFactory<String, String> cf = new DefaultKafkaConsumerFactory<>(consumerProps);
        Consumer<String, String> consumer = cf.createConsumer();
        embeddedKafka.consumeFromAnEmbeddedTopic(consumer, "example-topic");
        
        // Act
        producerService.sendMessage("example-topic", message);
        
        // Assert
        ConsumerRecord<String, String> record = KafkaTestUtils.getSingleRecord(consumer, "example-topic");
        assertThat(record).isNotNull();
        assertThat(record.value()).isEqualTo(message);
        
        consumer.close();
    }
}
```

### Testing with Mock Kafka Template

Use mocking for unit testing:

```java
@ExtendWith(MockitoExtension.class)
public class KafkaProducerServiceUnitTest {
    
    @Mock
    private KafkaTemplate<String, String> kafkaTemplate;
    
    @InjectMocks
    private KafkaProducerService producerService;
    
    @Test
    public void testSendMessage() {
        // Arrange
        String topic = "example-topic";
        String message = "Test message";
        
        when(kafkaTemplate.send(topic, message))
            .thenReturn(mock(ListenableFuture.class));
        
        // Act
        producerService.sendMessage(topic, message);
        
        // Assert
        verify(kafkaTemplate, times(1)).send(topic, message);
    }
}
```

## Monitoring and Management

### Kafka Metrics

Spring Boot automatically exposes Kafka metrics when Micrometer is on the classpath:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

Configure Actuator to expose Kafka metrics:

```properties
management.endpoints.web.exposure.include=health,info,metrics,prometheus
management.metrics.export.prometheus.enabled=true
```

### Kafka Admin Client

Use the Kafka Admin Client for management operations:

```java
@Service
public class KafkaAdminService {
    
    private final AdminClient adminClient;
    
    public KafkaAdminService(@Value("${spring.kafka.bootstrap-servers}") String bootstrapServers) {
        Map<String, Object> configs = new HashMap<>();
        configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        this.adminClient = AdminClient.create(configs);
    }
    
    public Set<String> listTopics() throws ExecutionException, InterruptedException {
        ListTopicsResult topics = adminClient.listTopics();
        return topics.names().get();
    }
    
    public void createTopic(String topicName, int numPartitions, short replicationFactor) {
        NewTopic newTopic = new NewTopic(topicName, numPartitions, replicationFactor);
        adminClient.createTopics(Collections.singleton(newTopic));
    }
    
    public void deleteTopic(String topicName) {
        adminClient.deleteTopics(Collections.singleton(topicName));
    }
    
    public TopicDescription describeTopic(String topicName) throws ExecutionException, InterruptedException {
        DescribeTopicsResult result = adminClient.describeTopics(Collections.singleton(topicName));
        Map<String, TopicDescription> topicDescriptions = result.all().get();
        return topicDescriptions.get(topicName);
    }
}
```

## Best Practices

### 1. Use Appropriate Serialization

- Use JSON serialization for complex objects
- Consider using Avro or Protocol Buffers for schema evolution

### 2. Configure Proper Acknowledgments

```properties
spring.kafka.producer.acks=all
```

### 3. Handle Errors Properly

- Use error handlers and dead letter topics
- Implement proper retry mechanisms

### 4. Monitor Your Kafka Cluster

- Use Prometheus and Grafana for monitoring
- Set up alerts for important metrics

### 5. Secure Your Kafka Cluster

```properties
# SSL Configuration
spring.kafka.security.protocol=SSL
spring.kafka.ssl.key-store-location=classpath:keystore.jks
spring.kafka.ssl.key-store-password=password
spring.kafka.ssl.trust-store-location=classpath:truststore.jks
spring.kafka.ssl.trust-store-password=password
```

### 6. Use Appropriate Partitioning

- Choose appropriate keys for partitioning
- Ensure even distribution of messages across partitions

### 7. Configure Consumer Groups Properly

- Use meaningful consumer group IDs
- Configure appropriate number of consumers

### 8. Implement Idempotent Consumers

- Handle duplicate messages gracefully
- Use unique message IDs for deduplication

## Advanced Topics

### Kafka Streams

Kafka Streams is a client library for building applications and microservices that process and analyze data stored in Kafka:

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

```java
@Configuration
@EnableKafkaStreams
public class KafkaStreamsConfig {
    
    @Bean
    public KStream<String, String> kStream(StreamsBuilder streamsBuilder) {
        KStream<String, String> stream = streamsBuilder.stream("input-topic");
        
        stream.filter((key, value) -> value.contains("important"))
              .mapValues(value -> value.toUpperCase())
              .to("output-topic");
        
        return stream;
    }
    
    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration kStreamsConfig() {
        Map<String, Object> props = new HashMap<>();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-app");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        
        return new KafkaStreamsConfiguration(props);
    }
}
```

### Transactional Messages

Configure transactional producers:

```properties
spring.kafka.producer.transaction-id-prefix=tx-
```

```java
@Transactional
public void sendMessagesInTransaction(String topic, List<String> messages) {
    messages.forEach(message -> kafkaTemplate.send(topic, message));
}
```

### Batch Processing

Process messages in batches:

```java
@KafkaListener(topics = "example-topic", groupId = "batch-group", 
               containerFactory = "batchFactory")
public void listenBatch(List<String> messages) {
    logger.info("Received batch of {} messages", messages.size());
    // Process the batch
}

@Bean
public ConcurrentKafkaListenerContainerFactory<String, String> batchFactory(
        ConsumerFactory<String, String> consumerFactory) {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = 
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory);
    factory.setBatchListener(true);
    factory.setBatchMessageConverter(new BatchMessagingMessageConverter());
    return factory;
}
```

### Schema Registry

Use a schema registry for schema evolution:

```xml
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>
    <version>7.0.1</version>
</dependency>
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>1.11.0</version>
</dependency>
```

```properties
spring.kafka.producer.value-serializer=io.confluent.kafka.serializers.KafkaAvroSerializer
spring.kafka.consumer.value-deserializer=io.confluent.kafka.serializers.KafkaAvroDeserializer
spring.kafka.properties.schema.registry.url=http://localhost:8081
```

## Conclusion

Integrating Apache Kafka with Spring Boot provides a powerful foundation for building event-driven applications. By following the guidelines and best practices outlined in this guide, you can create robust, scalable, and maintainable Kafka-based systems.

Remember that Kafka is a complex system with many configuration options. It's important to understand the trade-offs involved in different configuration choices and to monitor your Kafka cluster to ensure it's performing as expected.

## References

- [Spring for Apache Kafka Documentation](https://docs.spring.io/spring-kafka/reference/html/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Confluent Documentation](https://docs.confluent.io/platform/current/overview.html)
- [Kafka Streams Documentation](https://kafka.apache.org/documentation/streams/)
