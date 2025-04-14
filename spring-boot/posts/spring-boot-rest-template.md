# Spring Boot RestTemplate Guide

*Published on March 25, 2024*

## Overview

This guide provides a comprehensive overview of Spring Boot's RestTemplate, a synchronous HTTP client for consuming RESTful web services. While RestTemplate is now in maintenance mode (with WebClient being the recommended alternative), it remains widely used in existing applications. This guide covers everything from basic setup to advanced configuration, error handling, and best practices for using RestTemplate effectively.

## Table of Contents

1. [What is RestTemplate?](#what-is-resttemplate)
2. [Getting Started](#getting-started)
3. [Basic Configuration](#basic-configuration)
4. [HTTP Methods](#http-methods)
5. [Advanced Features](#advanced-features)
6. [Error Handling](#error-handling)
7. [Security](#security)
8. [Testing](#testing)
9. [Best Practices](#best-practices)
10. [Migration to WebClient](#migration-to-webclient)
11. [References](#references)

## What is RestTemplate?

RestTemplate is a synchronous HTTP client that provides a convenient way to consume RESTful web services. It's part of the Spring Web module and offers:

- Simple API for HTTP operations
- Built-in message conversion
- Error handling
- URI template support
- Request/response interceptors

### Key Features:
- Synchronous HTTP client
- Template-based API
- Built-in message converters
- Exception translation
- URI template support
- Request/response interceptors

### Common Use Cases:
- Consuming REST APIs
- Microservice communication
- Third-party API integration
- Legacy system integration
- Data synchronization
- Service-to-service communication

## Getting Started

### Prerequisites
- Java 8 or higher
- Spring Boot 2.x or higher
- Spring Web module

### Dependencies

#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

## Basic Configuration

### 1. Simple Configuration
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 2. Advanced Configuration
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(3000);
        factory.setReadTimeout(3000);
        
        RestTemplate restTemplate = new RestTemplate(factory);
        restTemplate.setErrorHandler(new CustomResponseErrorHandler());
        restTemplate.setInterceptors(Collections.singletonList(new CustomClientHttpRequestInterceptor()));
        
        return restTemplate;
    }
}
```

## HTTP Methods

### 1. GET Requests
```java
@Service
public class UserService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public User getUser(String userId) {
        String url = "http://api.example.com/users/{id}";
        return restTemplate.getForObject(url, User.class, userId);
    }
    
    public ResponseEntity<User> getUserWithResponse(String userId) {
        String url = "http://api.example.com/users/{id}";
        return restTemplate.getForEntity(url, User.class, userId);
    }
    
    public User getUserWithHeaders(String userId, String token) {
        String url = "http://api.example.com/users/{id}";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + token);
        headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
        
        HttpEntity<String> entity = new HttpEntity<>(headers);
        
        return restTemplate.exchange(
            url,
            HttpMethod.GET,
            entity,
            User.class,
            userId
        ).getBody();
    }
}
```

### 2. POST Requests
```java
@Service
public class UserService {
    
    public User createUser(User user) {
        String url = "http://api.example.com/users";
        return restTemplate.postForObject(url, user, User.class);
    }
    
    public ResponseEntity<User> createUserWithResponse(User user) {
        String url = "http://api.example.com/users";
        return restTemplate.postForEntity(url, user, User.class);
    }
    
    public User createUserWithHeaders(User user, String token) {
        String url = "http://api.example.com/users";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", "Bearer " + token);
        
        HttpEntity<User> request = new HttpEntity<>(user, headers);
        
        return restTemplate.exchange(
            url,
            HttpMethod.POST,
            request,
            User.class
        ).getBody();
    }
}
```

### 3. PUT and PATCH Requests
```java
@Service
public class UserService {
    
    public void updateUser(String userId, User user) {
        String url = "http://api.example.com/users/{id}";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<User> request = new HttpEntity<>(user, headers);
        
        restTemplate.exchange(
            url,
            HttpMethod.PUT,
            request,
            Void.class,
            userId
        );
    }
    
    public void partialUpdateUser(String userId, Map<String, Object> updates) {
        String url = "http://api.example.com/users/{id}";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        
        HttpEntity<Map<String, Object>> request = new HttpEntity<>(updates, headers);
        
        restTemplate.exchange(
            url,
            HttpMethod.PATCH,
            request,
            Void.class,
            userId
        );
    }
}
```

### 4. DELETE Requests
```java
@Service
public class UserService {
    
    public void deleteUser(String userId) {
        String url = "http://api.example.com/users/{id}";
        restTemplate.delete(url, userId);
    }
    
    public void deleteUserWithHeaders(String userId, String token) {
        String url = "http://api.example.com/users/{id}";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + token);
        
        HttpEntity<String> entity = new HttpEntity<>(headers);
        
        restTemplate.exchange(
            url,
            HttpMethod.DELETE,
            entity,
            Void.class,
            userId
        );
    }
}
```

## Advanced Features

### 1. Custom Message Converters
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        List<HttpMessageConverter<?>> converters = new ArrayList<>();
        converters.add(new MappingJackson2HttpMessageConverter());
        converters.add(new StringHttpMessageConverter());
        
        restTemplate.setMessageConverters(converters);
        return restTemplate;
    }
}
```

### 2. Request/Response Interceptors
```java
@Component
public class CustomClientHttpRequestInterceptor implements ClientHttpRequestInterceptor {
    
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) 
            throws IOException {
        // Add custom headers
        request.getHeaders().set("X-Custom-Header", "value");
        
        // Log request
        logRequest(request, body);
        
        // Execute request
        ClientHttpResponse response = execution.execute(request, body);
        
        // Log response
        logResponse(response);
        
        return response;
    }
}
```

### 3. URI Template Handling
```java
@Service
public class UserService {
    
    public User getUserWithUriTemplate(String userId) {
        String url = "http://api.example.com/users/{id}";
        Map<String, String> params = new HashMap<>();
        params.put("id", userId);
        
        return restTemplate.getForObject(url, User.class, params);
    }
}
```

## Error Handling

### 1. Custom Error Handler
```java
public class CustomResponseErrorHandler extends DefaultResponseErrorHandler {
    
    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        if (response.getStatusCode() == HttpStatus.NOT_FOUND) {
            throw new ResourceNotFoundException("Resource not found");
        }
        if (response.getStatusCode() == HttpStatus.BAD_REQUEST) {
            throw new BadRequestException("Invalid request");
        }
        if (response.getStatusCode() == HttpStatus.UNAUTHORIZED) {
            throw new UnauthorizedException("Authentication required");
        }
        super.handleError(response);
    }
}
```

### 2. Exception Handling
```java
@RestControllerAdvice
public class RestTemplateExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", ex.getMessage()));
    }
    
    @ExceptionHandler(BadRequestException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(BadRequestException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(new ErrorResponse("BAD_REQUEST", ex.getMessage()));
    }
}
```

## Security

### 1. SSL Configuration
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() throws Exception {
        SSLContext sslContext = SSLContextBuilder
            .create()
            .loadTrustMaterial((chain, authType) -> true)
            .build();
        
        HttpClient client = HttpClients.custom()
            .setSSLContext(sslContext)
            .build();
        
        HttpComponentsClientHttpRequestFactory factory = 
            new HttpComponentsClientHttpRequestFactory(client);
        
        return new RestTemplate(factory);
    }
}
```

### 2. Basic Authentication
```java
@Service
public class UserService {
    
    public User getUserWithBasicAuth(String userId) {
        String url = "http://api.example.com/users/{id}";
        
        HttpHeaders headers = new HttpHeaders();
        headers.setBasicAuth("username", "password");
        
        HttpEntity<String> entity = new HttpEntity<>(headers);
        
        return restTemplate.exchange(
            url,
            HttpMethod.GET,
            entity,
            User.class,
            userId
        ).getBody();
    }
}
```

## Testing

### 1. Unit Tests
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private RestTemplate restTemplate;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void testGetUser() {
        User expectedUser = new User("1", "John Doe");
        when(restTemplate.getForObject(anyString(), eq(User.class), anyString()))
            .thenReturn(expectedUser);
        
        User result = userService.getUser("1");
        
        assertThat(result).isEqualTo(expectedUser);
    }
}
```

### 2. Integration Tests
```java
@SpringBootTest
class UserServiceIntegrationTest {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @Test
    void testGetUser() {
        User user = userService.getUser("1");
        
        assertThat(user).isNotNull();
        assertThat(user.getId()).isEqualTo("1");
    }
}
```

## Best Practices

1. **Configuration**:
   - Use dependency injection
   - Configure timeouts
   - Set up error handling
   - Use interceptors
   - Configure message converters

2. **Error Handling**:
   - Implement custom error handlers
   - Use proper exception handling
   - Log errors appropriately
   - Provide meaningful error messages
   - Handle different HTTP status codes

3. **Security**:
   - Use HTTPS
   - Implement proper authentication
   - Handle SSL certificates
   - Secure sensitive data
   - Validate responses

4. **Performance**:
   - Configure timeouts
   - Use connection pooling
   - Implement retry mechanisms
   - Cache responses when appropriate
   - Monitor performance

## Migration to WebClient

As of Spring 5.0, WebClient is the recommended alternative to RestTemplate. Here's how to migrate:

```java
// RestTemplate
User user = restTemplate.getForObject(url, User.class, userId);

// WebClient
User user = webClient.get()
    .uri(url, userId)
    .retrieve()
    .bodyToMono(User.class)
    .block();
```

Key benefits of WebClient:
- Reactive programming support
- Non-blocking I/O
- Better performance
- Modern API
- Better error handling

## References

- [Spring RestTemplate Documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)
- [Spring WebClient Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [HTTP Client Best Practices](https://www.baeldung.com/rest-template)
- [Spring Security Documentation](https://docs.spring.io/spring-security/reference/)
