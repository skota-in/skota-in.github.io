# Spring Boot MongoDB Integration Guide

*Published on March 5, 2024*

## Overview

This guide provides a comprehensive overview of integrating MongoDB with Spring Boot applications. MongoDB is a popular NoSQL database that offers flexibility, scalability, and high performance. This guide covers everything from basic setup to advanced features, including practical examples and best practices for production environments.

## Table of Contents

1. [What is MongoDB?](#what-is-mongodb)
2. [Prerequisites](#prerequisites)
3. [Dependencies](#dependencies)
4. [Configuration](#configuration)
5. [Implementation](#implementation)
6. [Advanced Features](#advanced-features)
7. [Error Handling](#error-handling)
8. [Testing](#testing)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)
11. [References](#references)

## What is MongoDB?

MongoDB is a document-oriented NoSQL database that provides:

- Flexible document model
- High performance
- Horizontal scalability
- Rich query language
- Automatic sharding
- Built-in replication

### Key Benefits:
- Schema flexibility
- High performance
- Scalability
- Rich query capabilities
- Easy integration
- Active community

### Common Use Cases:
- Content management
- Real-time analytics
- Internet of Things
- Mobile applications
- User data management
- Logging and event data

## Prerequisites

- Java 8 or higher
- Spring Boot 2.x or 3.x
- MongoDB server (local or cloud)
- Basic understanding of NoSQL concepts
- Familiarity with Spring Boot

## Dependencies

Add these dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

## Configuration

### Application Properties
Add these properties to `application.properties` or `application.yml`:

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/your_database
      # For authentication:
      # uri: mongodb://username:password@localhost:27017/your_database
      # Additional configuration:
      auto-index-creation: true
      authentication-database: admin
      connection-pool-size: 100
      connect-timeout: 5000
      socket-timeout: 3000
      max-wait-time: 1500
```

## Implementation

### 1. Model/Entity
```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.index.Indexed;
import lombok.Data;

@Document(collection = "users")
@Data
public class User {
    @Id
    private String id;
    
    @Indexed(unique = true)
    private String email;
    
    private String name;
    private int age;
    private List<String> roles;
    private Address address;
    
    @Data
    public static class Address {
        private String street;
        private String city;
        private String country;
    }
}
```

### 2. Repository Interface
```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
import java.util.List;

public interface UserRepository extends MongoRepository<User, String> {
    User findByEmail(String email);
    List<User> findByName(String name);
    
    @Query("{ 'age' : { $gt: ?0 } }")
    List<User> findUsersByAgeGreaterThan(int age);
    
    @Query("{ 'email' : { $regex: ?0 } }")
    List<User> findByEmailRegex(String emailPattern);
}
```

### 3. Service Layer
```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final MongoTemplate mongoTemplate;
    
    public UserService(UserRepository userRepository, MongoTemplate mongoTemplate) {
        this.userRepository = userRepository;
        this.mongoTemplate = mongoTemplate;
    }
    
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    public User getUserById(String id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("User not found"));
    }
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    public void deleteUser(String id) {
        userRepository.deleteById(id);
    }
    
    public List<UserStats> getUserStats() {
        TypedAggregation<User> aggregation = Aggregation.newAggregation(
            User.class,
            Aggregation.group("age")
                .count().as("total")
                .avg("salary").as("averageSalary"),
            Aggregation.project("total", "averageSalary")
                .andExpression("_id").as("age")
        );
        
        return mongoTemplate.aggregate(aggregation, UserStats.class)
            .getMappedResults();
    }
}
```

### 4. REST Controller
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        return ResponseEntity.ok(userService.createUser(user));
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable String id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable String id, @RequestBody User user) {
        user.setId(id);
        return ResponseEntity.ok(userService.updateUser(user));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable String id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Advanced Features

### 1. Custom Queries
```java
@Repository
public interface UserRepository extends MongoRepository<User, String> {
    // Text search
    @Query("{ $text: { $search: ?0 } }")
    List<User> searchUsers(String searchTerm);
    
    // Geospatial queries
    @Query("{ 'address.location' : { $near : { $geometry : { type : 'Point' , coordinates : [ ?0, ?1 ] } , $maxDistance : ?2 } } }")
    List<User> findUsersNearLocation(double longitude, double latitude, double maxDistance);
}
```

### 2. Index Management
```java
@Configuration
public class MongoConfig {
    @Bean
    public MongoTemplate mongoTemplate(MongoDatabaseFactory mongoDbFactory) {
        MongoTemplate template = new MongoTemplate(mongoDbFactory);
        template.indexOps(User.class).ensureIndex(
            new Index().on("email", Sort.Direction.ASC).unique()
        );
        return template;
    }
}
```

### 3. Change Streams
```java
@Service
public class UserChangeStreamService {
    private final MongoTemplate mongoTemplate;
    
    public void watchUserChanges() {
        mongoTemplate.getCollection("users")
            .watch()
            .forEach(change -> {
                // Handle change events
                System.out.println("Change detected: " + change);
            });
    }
}
```

## Error Handling

### 1. Global Exception Handler
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    private final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(NotFoundException e) {
        logger.error("Resource not found", e);
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(HttpStatus.NOT_FOUND.value(), e.getMessage()));
    }
    
    @ExceptionHandler(DuplicateKeyException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateKeyException(DuplicateKeyException e) {
        logger.error("Duplicate key violation", e);
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(new ErrorResponse(HttpStatus.CONFLICT.value(), "Resource already exists"));
    }
}
```

### 2. Custom Exceptions
```java
public class NotFoundException extends RuntimeException {
    public NotFoundException(String message) {
        super(message);
    }
}
```

## Testing

### 1. Unit Tests
```java
@SpringBootTest
class UserServiceTest {
    @Autowired
    private UserService userService;
    
    @Test
    void testCreateUser() {
        User user = new User();
        user.setName("Test User");
        user.setEmail("test@example.com");
        
        User savedUser = userService.createUser(user);
        assertNotNull(savedUser.getId());
        assertEquals("Test User", savedUser.getName());
    }
}
```

### 2. Integration Tests
```java
@SpringBootTest
class UserControllerTest {
    @Autowired
    private WebTestClient webTestClient;
    
    @Test
    void testCreateUser() {
        User user = new User();
        user.setName("Test User");
        user.setEmail("test@example.com");
        
        webTestClient.post()
            .uri("/api/users")
            .bodyValue(user)
            .exchange()
            .expectStatus().isOk()
            .expectBody(User.class)
            .value(u -> assertEquals("Test User", u.getName()));
    }
}
```

## Best Practices

1. **Data Modeling**:
   - Design documents based on access patterns
   - Use embedded documents when appropriate
   - Consider document size limits
   - Implement proper indexing
   - Use appropriate data types

2. **Performance Optimization**:
   - Create appropriate indexes
   - Use projection to limit returned fields
   - Implement pagination
   - Use bulk operations
   - Monitor query performance

3. **Security**:
   - Use environment variables for credentials
   - Implement proper authentication
   - Use SSL/TLS for connections
   - Implement proper access control
   - Regular security audits

4. **Monitoring and Maintenance**:
   - Monitor database performance
   - Regular backups
   - Index maintenance
   - Query optimization
   - Capacity planning

## Troubleshooting

### Common Issues and Solutions:

1. **Connection Issues**:
   - Verify MongoDB is running
   - Check connection string format
   - Validate credentials
   - Check network connectivity
   - Verify firewall settings

2. **Performance Issues**:
   - Check index usage
   - Monitor query performance
   - Review document structure
   - Check resource utilization
   - Optimize queries

3. **Data Consistency**:
   - Implement proper error handling
   - Use transactions when needed
   - Handle concurrent updates
   - Implement proper validation
   - Monitor data integrity

## References

- [MongoDB Documentation](https://docs.mongodb.com/)
- [Spring Data MongoDB Documentation](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/)
- [MongoDB Best Practices](https://www.mongodb.com/basics/best-practices)
- [Spring Boot MongoDB Guide](https://spring.io/guides/gs/accessing-data-mongodb/)
- [MongoDB Security Guide](https://www.mongodb.com/docs/manual/security/)