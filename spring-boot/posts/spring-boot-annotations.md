# Spring Boot Annotations: A Comprehensive Guide

*Published on November 25, 2024*

Spring Boot provides a rich set of annotations that help in configuring and managing Spring applications. This guide covers the most commonly used annotations in Spring Boot.

## Core Spring Boot Annotations

### @SpringBootApplication
This is the main annotation that combines three other annotations:
- @Configuration
- @EnableAutoConfiguration
- @ComponentScan

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

## Spring MVC Annotations

### @Controller
Marks a class as a Spring MVC controller.

```java
@Controller
public class MyController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

### @RestController
A specialized version of @Controller that includes @ResponseBody.

```java
@RestController
public class MyRestController {
    @GetMapping("/api/hello")
    public String hello() {
        return "Hello, World!";
    }
}
```

### Request Mapping Annotations
- @GetMapping
- @PostMapping
- @PutMapping
- @DeleteMapping
- @PatchMapping

```java
@RestController
public class UserController {
    @GetMapping("/users")
    public List<User> getUsers() {
        // implementation
    }
    
    @PostMapping("/users")
    public User createUser(@RequestBody User user) {
        // implementation
    }
}
```

## Dependency Injection Annotations

### @Autowired
Automatically injects bean dependencies.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### @Component
Marks a class as a Spring component.

```java
@Component
public class MyComponent {
    // implementation
}
```

### @Service
Specialized @Component for service layer.

```java
@Service
public class UserService {
    // implementation
}
```

### @Repository
Specialized @Component for data access layer.

```java
@Repository
public class UserRepository {
    // implementation
}
```

## Configuration Annotations

### @Configuration
Indicates that a class declares one or more @Bean methods.

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

### @Bean
Indicates that a method produces a bean to be managed by Spring.

```java
@Configuration
public class AppConfig {
    @Bean
    public DataSource dataSource() {
        // implementation
    }
}
```

### @Value
Injects values from properties files.

```java
@Component
public class MyComponent {
    @Value("${app.name}")
    private String appName;
}
```

## Database Annotations

### @Entity
Specifies that the class is an entity.

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}
```

### @Repository
Marks the interface as a JPA repository.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    // implementation
}
```

## Security Annotations

### @EnableWebSecurity
Enables Spring Security's web security support.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // implementation
}
```

### @Secured
Specifies security roles for methods.

```java
@Service
public class UserService {
    @Secured("ROLE_ADMIN")
    public void adminOperation() {
        // implementation
    }
}
```

## Testing Annotations

### @SpringBootTest
Used for Spring Boot integration tests.

```java
@SpringBootTest
public class MyIntegrationTest {
    // implementation
}
```

### @Test
Marks a method as a test method.

```java
@SpringBootTest
public class UserServiceTest {
    @Test
    public void testUserCreation() {
        // implementation
    }
}
```

## Scheduling Annotations

### @EnableScheduling
Enables Spring's scheduled task execution capability.

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {
    // implementation
}
```

### @Scheduled
Marks a method to be scheduled.

```java
@Component
public class ScheduledTasks {
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        // implementation
    }
}
```

## Caching Annotations

### @EnableCaching
Enables Spring's caching capability.

```java
@Configuration
@EnableCaching
public class CacheConfig {
    // implementation
}
```

### @Cacheable
Marks a method as cacheable.

```java
@Service
public class UserService {
    @Cacheable("users")
    public User getUser(Long id) {
        // implementation
    }
}
```

## Validation Annotations

### @Valid
Marks a property, method parameter, or method return type for validation.

```java
@RestController
public class UserController {
    @PostMapping("/users")
    public User createUser(@Valid @RequestBody User user) {
        // implementation
    }
}
```

### @Validated
Marks a class for validation.

```java
@Service
@Validated
public class UserService {
    public void validateUser(@Valid User user) {
        // implementation
    }
}
```

## Conclusion

These annotations form the foundation of Spring Boot development. Understanding and using them effectively can help you build robust and maintainable Spring Boot applications. Remember that this is not an exhaustive list, and Spring Boot provides many more annotations for specific use cases.
