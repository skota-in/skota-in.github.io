# Spring Boot REST API Best Practices Guide

*Published on March 15, 2024*

## Overview

This guide provides comprehensive best practices for building robust, secure, and maintainable REST APIs using Spring Boot. It covers everything from API design principles to implementation details, security considerations, and performance optimization. Whether you're building a new API or improving an existing one, these practices will help you create high-quality, production-ready REST services.

## Table of Contents

1. [What is REST?](#what-is-rest)
2. [API Design Principles](#api-design-principles)
3. [Request/Response Structure](#requestresponse-structure)
4. [Error Handling](#error-handling)
5. [Versioning Strategies](#versioning-strategies)
6. [Security Best Practices](#security-best-practices)
7. [Documentation](#documentation)
8. [Performance Optimization](#performance-optimization)
9. [Testing](#testing)
10. [Best Practices](#best-practices)
11. [References](#references)

## What is REST?

REST (Representational State Transfer) is an architectural style for designing networked applications. It uses:

- Stateless communication
- Standard HTTP methods
- Resource-based URLs
- JSON/XML data formats
- Cacheable responses

### Key Principles:
- Client-server architecture
- Statelessness
- Cacheability
- Layered system
- Uniform interface
- Code on demand (optional)

### Common Use Cases:
- Web services
- Mobile applications
- Microservices
- Public APIs
- Internal services
- Integration points

## API Design Principles

### 1. Resource Naming
```java
// Good examples
@GetMapping("/api/users")
@GetMapping("/api/users/{id}/orders")
@GetMapping("/api/products/{id}/reviews")

// Bad examples
@GetMapping("/api/getUsers")
@GetMapping("/api/fetchOrders")
@GetMapping("/api/retrieveReviews")
```

### 2. HTTP Methods Usage
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping
    public List<User> getUsers() {
        // Retrieve all users
    }
    
    @PostMapping
    public User createUser(@RequestBody User user) {
        // Create new user
    }
    
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody User user) {
        // Update existing user
    }
    
    @PatchMapping("/{id}")
    public User partialUpdateUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
        // Partial update
    }
    
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        // Delete user
    }
}
```

### 3. Status Codes
```java
@RestController
public class UserController {
    
    @PostMapping("/users")
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User savedUser = userService.createUser(user);
        return ResponseEntity
            .created(URI.create("/api/users/" + savedUser.getId()))
            .body(savedUser);
    }
    
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

## Request/Response Structure

### 1. DTOs and Validation
```java
@Data
public class UserDTO {
    private Long id;
    private String username;
    private String email;
    private LocalDateTime createdAt;
}

@Data
public class UserCreateRequest {
    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50)
    private String username;
    
    @Email(message = "Email should be valid")
    @NotBlank(message = "Email is required")
    private String email;
    
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserCreateRequest request) {
        UserDTO user = userService.createUser(request);
        return ResponseEntity.created(URI.create("/api/users/" + user.getId()))
            .body(user);
    }
}
```

### 2. Pagination and Sorting
```java
@GetMapping
public Page<UserDTO> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String direction
) {
    Sort.Direction sortDirection = Sort.Direction.fromString(direction);
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortDirection, sortBy));
    return userService.findAll(pageable);
}
```

## Error Handling

### 1. Global Exception Handler
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    private final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
        ResourceNotFoundException ex) {
        logger.error("Resource not found", ex);
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(
        MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("VALIDATION_FAILED", "Validation failed", errors));
    }
}
```

### 2. Custom Exceptions
```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class BusinessException extends RuntimeException {
    private final String errorCode;
    
    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public String getErrorCode() {
        return errorCode;
    }
}
```

## Versioning Strategies

### 1. URI Versioning
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 {
    // V1 implementation
}

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    // V2 implementation
}
```

### 2. Header Versioning
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

### 1. HTTPS Configuration
```yaml
server:
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: ${KEYSTORE_PASSWORD}
    key-store-type: PKCS12
    key-alias: tomcat
  port: 8443
```

### 2. Rate Limiting
```java
@Configuration
public class RateLimitingConfig {
    
    @Bean
    public Bucket createBucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(20, Refill.intervally(20, Duration.ofMinutes(1))))
            .build();
    }
}

@RestController
@RequestMapping("/api")
public class ApiController {
    
    private final Bucket bucket;
    
    @PostMapping("/search")
    public ResponseEntity<?> search(@RequestBody SearchRequest request) {
        if (bucket.tryConsume(1)) {
            return ResponseEntity.ok(searchService.search(request));
        }
        return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
            .body(new ErrorResponse("RATE_LIMIT_EXCEEDED", "Rate limit exceeded"));
    }
}
```

## Documentation

### 1. OpenAPI Configuration
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
                    .email("support@example.com")))
            .components(new Components()
                .addSecuritySchemes("bearerAuth", 
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")));
    }
}
```

### 2. API Documentation
```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Operation(summary = "Get all users", description = "Retrieves a paginated list of users")
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Successfully retrieved users"),
        @ApiResponse(responseCode = "401", description = "Unauthorized"),
        @ApiResponse(responseCode = "403", description = "Forbidden")
    })
    @GetMapping
    public Page<UserDTO> getUsers(Pageable pageable) {
        return userService.findAll(pageable);
    }
}
```

## Performance Optimization

### 1. Caching
```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products");
    }
}

@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public UserDTO findById(Long id) {
        // Implementation
    }
}
```

### 2. Compression
```yaml
server:
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
    min-response-size: 1024
```

## Testing

### 1. Unit Tests
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testCreateUser() {
        UserCreateRequest request = new UserCreateRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        
        when(userRepository.save(any(User.class)))
            .thenReturn(new User(1L, "testuser", "test@example.com"));
        
        UserDTO result = userService.createUser(request);
        
        assertNotNull(result);
        assertEquals("testuser", result.getUsername());
    }
}
```

### 2. Integration Tests
```java
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void testCreateUser() throws Exception {
        UserCreateRequest request = new UserCreateRequest();
        request.setUsername("testuser");
        request.setEmail("test@example.com");
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(asJsonString(request)))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.username").value("testuser"));
    }
}
```

## Best Practices

1. **API Design**:
   - Use nouns for resource names
   - Follow REST principles
   - Implement proper versioning
   - Use appropriate HTTP methods
   - Return proper status codes

2. **Security**:
   - Use HTTPS in production
   - Implement authentication
   - Validate all inputs
   - Use rate limiting
   - Implement proper authorization

3. **Performance**:
   - Implement caching
   - Use compression
   - Optimize database queries
   - Implement pagination
   - Monitor performance

4. **Maintenance**:
   - Document your API
   - Write comprehensive tests
   - Follow coding standards
   - Implement proper logging
   - Monitor API usage

## References

- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [REST API Design Best Practices](https://www.mulesoft.com/resources/api/rest-api-design-best-practices)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/)
- [Spring Testing Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html)
