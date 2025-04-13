# Spring Boot REST API Best Practices

*Published on April 10, 2023*

## Introduction

Building robust and maintainable REST APIs is crucial for modern application development. Spring Boot provides an excellent framework for creating RESTful services, but following best practices ensures your APIs are secure, performant, and developer-friendly.

## Table of Contents

1. [API Design Principles](#api-design-principles)
2. [Request/Response Structure](#requestresponse-structure)
3. [Error Handling](#error-handling)
4. [Versioning Strategies](#versioning-strategies)
5. [Security Best Practices](#security-best-practices)
6. [Documentation](#documentation)
7. [Performance Optimization](#performance-optimization)
8. [Testing](#testing)

## API Design Principles

### Use Nouns, Not Verbs in Endpoints

```
✅ Good: /api/users
❌ Bad: /api/getUsers
```

### Use HTTP Methods Appropriately

- `GET`: Retrieve resources
- `POST`: Create resources
- `PUT`: Update resources (full update)
- `PATCH`: Partial update of resources
- `DELETE`: Remove resources

### Use Proper HTTP Status Codes

- `200 OK`: Successful request
- `201 Created`: Resource created successfully
- `204 No Content`: Successful request with no response body
- `400 Bad Request`: Invalid request
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Authenticated but not authorized
- `404 Not Found`: Resource not found
- `500 Internal Server Error`: Server-side error

### Implement Pagination for Collections

```java
@GetMapping("/users")
public Page<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return userRepository.findAll(pageable);
}
```

## Request/Response Structure

### Use DTOs (Data Transfer Objects)

```java
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    // Getters and setters
}
```

### Implement Request Validation

```java
@PostMapping("/users")
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserCreateRequest request) {
    // Implementation
}

public class UserCreateRequest {
    @NotBlank(message = "Username is required")
    private String username;
    
    @Email(message = "Email should be valid")
    @NotBlank(message = "Email is required")
    private String email;
    
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
    
    // Getters and setters
}
```

## Error Handling

### Create a Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("NOT_FOUND", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ErrorResponse error = new ErrorResponse("VALIDATION_FAILED", "Validation failed", errors);
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
    
    // Other exception handlers
}
```

### Create Custom Exceptions

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

## Versioning Strategies

### URI Versioning

```
/api/v1/users
/api/v2/users
```

### Header Versioning

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping(headers = "API-Version=1")
    public List<UserV1DTO> getUsersV1() {
        // V1 implementation
    }
    
    @GetMapping(headers = "API-Version=2")
    public List<UserV2DTO> getUsersV2() {
        // V2 implementation
    }
}
```

## Security Best Practices

### Use HTTPS

Configure your application to use HTTPS in production:

```properties
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=your-password
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat
server.port=8443
```

### Implement Rate Limiting

Use libraries like Bucket4j or Spring Cloud Gateway for rate limiting:

```java
@Configuration
public class RateLimitingConfig {
    
    @Bean
    public Bucket createBucket() {
        long capacity = 20;
        long refillTokens = 20;
        Duration refillDuration = Duration.ofMinutes(1);
        
        RefillStrategy refillStrategy = Refill.intervally(refillTokens, refillDuration);
        Bandwidth bandwidth = Bandwidth.classic(capacity, refillStrategy);
        
        return Bucket.builder()
                .addLimit(bandwidth)
                .build();
    }
}
```

### Input Validation

Always validate user input to prevent injection attacks:

```java
@PostMapping("/search")
public List<Product> searchProducts(@RequestParam @Pattern(regexp = "^[a-zA-Z0-9 ]+$") String query) {
    return productService.search(query);
}
```

## Documentation

### Use Springdoc OpenAPI (formerly Swagger)

Add the dependency:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.9</version>
</dependency>
```

Configure OpenAPI:

```java
@Configuration
public class OpenApiConfig {
    
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("My API")
                        .version("1.0")
                        .description("API Documentation")
                        .contact(new Contact()
                                .name("API Support")
                                .email("support@example.com")));
    }
}
```

Document your endpoints:

```java
@Operation(summary = "Get user by ID", description = "Returns a user based on ID")
@ApiResponses(value = {
    @ApiResponse(responseCode = "200", description = "User found"),
    @ApiResponse(responseCode = "404", description = "User not found")
})
@GetMapping("/users/{id}")
public ResponseEntity<UserDTO> getUserById(
    @Parameter(description = "User ID") @PathVariable Long id
) {
    // Implementation
}
```

## Performance Optimization

### Use Caching

```java
@Configuration
@EnableCaching
public class CachingConfig {
    
    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("users"),
            new ConcurrentMapCache("products")
        ));
        return cacheManager;
    }
}

@Service
public class UserService {
    
    @Cacheable("users")
    public UserDTO getUserById(Long id) {
        // Implementation
    }
    
    @CacheEvict(value = "users", key = "#user.id")
    public void updateUser(UserDTO user) {
        // Implementation
    }
}
```

### Use Asynchronous Processing for Long-Running Tasks

```java
@Service
public class ReportService {
    
    @Async
    public CompletableFuture<Report> generateReport(ReportRequest request) {
        // Long-running task
        Report report = // generate report
        return CompletableFuture.completedFuture(report);
    }
}
```

## Testing

### Unit Testing Controllers

```java
@WebMvcTest(UserController.class)
public class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    public void testGetUserById() throws Exception {
        UserDTO user = new UserDTO();
        user.setId(1L);
        user.setUsername("testuser");
        
        when(userService.getUserById(1L)).thenReturn(user);
        
        mockMvc.perform(get("/api/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.username").value("testuser"));
    }
}
```

### Integration Testing

```java
@SpringBootTest
@AutoConfigureMockMvc
public class UserApiIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @BeforeEach
    public void setup() {
        userRepository.deleteAll();
        // Setup test data
    }
    
    @Test
    public void testCreateAndGetUser() throws Exception {
        // Test creating a user
        String userJson = "{\"username\":\"newuser\",\"email\":\"user@example.com\",\"password\":\"password123\"}";
        
        MvcResult result = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(userJson))
                .andExpect(status().isCreated())
                .andReturn();
        
        String response = result.getResponse().getContentAsString();
        Long userId = JsonPath.parse(response).read("$.id", Long.class);
        
        // Test getting the created user
        mockMvc.perform(get("/api/users/" + userId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.username").value("newuser"))
                .andExpect(jsonPath("$.email").value("user@example.com"));
    }
}
```

## Conclusion

Following these best practices will help you build robust, secure, and maintainable REST APIs with Spring Boot. Remember that the specific implementation details may vary based on your project requirements, but these principles provide a solid foundation for any RESTful service.

## References

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring REST Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)
- [REST API Design Best Practices](https://restfulapi.net/)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
