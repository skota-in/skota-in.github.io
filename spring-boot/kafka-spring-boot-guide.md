# Configuring Apache Kafka with Spring Boot

This guide provides instructions on how to integrate and configure Apache Kafka with a Spring Boot application.

## Prerequisites

- Java 8 or higher
- Spring Boot 2.x or higher
- Apache Kafka (local installation or cloud service)
- Maven or Gradle

## Dependencies

### Maven

Add the following dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

### Gradle

Add the following to your `build.gradle`:

```gradle
dependencies {
    implementation 'org.springframework.kafka:spring-kafka'
}
```

## Configuration Properties

Add the following properties to your `application.properties` or `application.yml` file:

### application.properties

```properties
# Kafka Producer Configuration
spring.kafka.producer.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

# Kafka Consumer Configuration
spring.kafka.consumer.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=my-group-id
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

### application.yml

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      bootstrap-servers: localhost:9092
      group-id: my-group-id
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

## Creating a Kafka Producer

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class KafkaProducerService {

    private static final String TOPIC = "example-topic";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String message) {
        kafkaTemplate.send(TOPIC, message);
        System.out.println("Message sent: " + message);
    }
}
```

## Creating a Kafka Consumer

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class KafkaConsumerService {

    @KafkaListener(topics = "example-topic", groupId = "my-group-id")
    public void listen(String message) {
        System.out.println("Received message: " + message);
        // Process the message
    }
}
```

## Configuring Kafka Topics Programmatically

```java
import org.apache.kafka.clients.admin.NewTopic;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.config.TopicBuilder;

@Configuration
public class KafkaTopicConfig {

    @Bean
    public NewTopic exampleTopic() {
        return TopicBuilder.name("example-topic")
                .partitions(3)
                .replicas(1)
                .build();
    }
}
```

## Using JSON Serialization/Deserialization

For sending and receiving JSON objects, you'll need to configure JSON serializers:

### Configuration

```properties
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example.model
```

### Example Model Class

```java
package com.example.model;

public class User {
    private String name;
    private int age;

    // Constructors, getters, and setters
    public User() {}

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + "}";
    }
}
```

### JSON Producer

```java
@Service
public class JsonKafkaProducerService {

    private static final String TOPIC = "user-topic";

    @Autowired
    private KafkaTemplate<String, User> kafkaTemplate;

    public void sendUser(User user) {
        kafkaTemplate.send(TOPIC, user);
        System.out.println("User sent: " + user);
    }
}
```

### JSON Consumer

```java
@Service
public class JsonKafkaConsumerService {

    @KafkaListener(topics = "user-topic", groupId = "json-group-id")
    public void listen(User user) {
        System.out.println("Received user: " + user);
        // Process the user object
    }
}
```

## Error Handling

```java
@Service
public class KafkaConsumerWithErrorHandling {

    @KafkaListener(topics = "example-topic", groupId = "error-handling-group")
    public void listen(String message, 
                      @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                      @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                      @Header(KafkaHeaders.OFFSET) long offset) {
        try {
            // Process the message
            System.out.println("Received message: " + message);
        } catch (Exception e) {
            System.err.println("Error processing message: " + message);
            System.err.println("From topic: " + topic + ", partition: " + partition + ", offset: " + offset);
            e.printStackTrace();
            // You might want to send to a dead letter queue here
        }
    }
}
```

## Common Issues and Troubleshooting

1. **Connection Issues**
   - Ensure Kafka broker is running and accessible
   - Check bootstrap server addresses
   - Verify network connectivity

2. **Serialization/Deserialization Errors**
   - Ensure serializers and deserializers match the data format
   - For JSON, make sure trusted packages are configured correctly

3. **Consumer Not Receiving Messages**
   - Check group ID configuration
   - Verify topic name is correct
   - Check auto-offset-reset property

4. **Producer Not Sending Messages**
   - Verify topic exists
   - Check for exceptions in logs

## Additional Resources

- [Spring for Apache Kafka Documentation](https://docs.spring.io/spring-kafka/reference/html/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Spring Boot Kafka Starter Guide](https://spring.io/guides/gs/messaging-kafka/)

## Advanced Configuration

For production environments, consider these additional configurations:

```properties
# Producer additional configurations
spring.kafka.producer.acks=all
spring.kafka.producer.retries=3
spring.kafka.producer.batch-size=16384
spring.kafka.producer.buffer-memory=33554432

# Consumer additional configurations
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.listener.ack-mode=MANUAL
spring.kafka.listener.concurrency=3
```

This will help you get started with integrating Apache Kafka with your Spring Boot application. Adjust the configurations based on your specific requirements.
