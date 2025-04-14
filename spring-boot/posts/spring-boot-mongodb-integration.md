# Spring Boot MongoDB Integration Guide

*Published on March 5, 2024*

## Step 1: Add Dependencies
Add these dependencies to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## Step 2: Configure MongoDB Connection
Add these properties to `application.properties` or `application.yml`:

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/your_database
      # For authentication:
      # uri: mongodb://username:password@localhost:27017/your_database
```

## Step 3: Create a Model/Entity
```java
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

@Document(collection = "users")
public class User {
    @Id
    private String id;
    private String name;
    private String email;
    
    // Constructors
    // Getters and Setters
}
```

## Step 4: Create Repository Interface
```java
import org.springframework.data.mongodb.repository.MongoRepository;

public interface UserRepository extends MongoRepository<User, String> {
    User findByEmail(String email);
    List<User> findByName(String name);
}
```

## Step 5: Create Service Layer
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User createUser(User user) {
        return userRepository.save(user);
    }
    
    public User getUserById(String id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("User not found"));
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
}
```

## Step 6: Create REST Controller
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @Autowired
    private UserService userService;
    
    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable String id) {
        return userService.getUserById(id);
    }
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.getAllUsers();
    }
    
    @PutMapping("/{id}")
    public User updateUser(@PathVariable String id, @RequestBody User user) {
        user.setId(id);
        return userService.updateUser(user);
    }
    
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable String id) {
        userService.deleteUser(id);
    }
}
```

## Step 7: Advanced MongoDB Operations

### Custom Queries
```java
@Repository
public interface UserRepository extends MongoRepository<User, String> {
    // Query by example
    List<User> findByNameLike(String name);
    
    // Custom query
    @Query("{ 'age' : { $gt: ?0 } }")
    List<User> findUsersByAgeGreaterThan(int age);
    
    // Regex query
    @Query("{ 'email' : { $regex: ?0 } }")
    List<User> findByEmailRegex(String emailPattern);
}
```

### Aggregation Example
```java
@Service
public class UserService {
    @Autowired
    private MongoTemplate mongoTemplate;
    
    public List<UserStats> getUserStats() {
        TypedAggregation<User> aggregation = Aggregation.newAggregation(
            User.class,
            Aggregation.group("age")
                .count().as("total")
                .avg("salary").as("averageSalary"),
            Aggregation.project("total", "averageSalary")
                .andExpression("_id").as("age")
        );
        
        AggregationResults<UserStats> results = mongoTemplate
            .aggregate(aggregation, UserStats.class);
        return results.getMappedResults();
    }
}
```

## Step 8: Exception Handling
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            e.getMessage()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
    
    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(NotFoundException e) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            e.getMessage()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
}
```

## Step 9: Testing
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

## Common Issues and Solutions

### Connection Issues
- Verify MongoDB is running: `mongo --eval "db.serverStatus()"`
- Check connection string format
- Ensure network connectivity and firewall settings
- Verify credentials if authentication is enabled

### Performance Tips
1. Use indexes for frequently queried fields:
```java
@Document(collection = "users")
@CompoundIndex(def = "{'name': 1, 'email': 1}")
public class User {
    // ...
}
```

2. Use projection to limit returned fields:
```java
@Query(value = "{ 'name' : ?0 }", fields = "{ 'name' : 1, 'email' : 1}")
List<User> findByName(String name);
```

3. Implement pagination:
```java
Page<User> findAll(Pageable pageable);
```

### Security Best Practices
1. Use environment variables for sensitive data:
```yaml
spring:
  data:
    mongodb:
      uri: ${MONGODB_URI}
```

2. Enable authentication and SSL:
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://username:password@localhost:27017/database?ssl=true
```

## Additional Resources
- [Spring Data MongoDB Documentation](https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/)
- [MongoDB Java Driver Documentation](https://mongodb.github.io/mongo-java-driver/)
- [Spring Boot MongoDB Tutorial](https://spring.io/guides/gs/accessing-data-mongodb/)