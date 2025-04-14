# Spring Boot RestTemplate Guide

*Published on February 10, 2025*

RestTemplate is a synchronous client to perform HTTP requests. It's part of the Spring Web module and provides a convenient way to consume RESTful web services.

## Setup

First, add the required dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## Basic Configuration

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

## HTTP Methods Examples

### 1. GET Request

#### Simple GET Request
```java
@Autowired
private RestTemplate restTemplate;

public User getUser(String userId) {
    String url = "http://api.example.com/users/{id}";
    return restTemplate.getForObject(url, User.class, userId);
}
```

#### GET Request with Headers
```java
public User getUserWithHeaders(String userId) {
    String url = "http://api.example.com/users/{id}";
    
    HttpHeaders headers = new HttpHeaders();
    headers.set("Authorization", "Bearer " + token);
    headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));
    
    HttpEntity<String> entity = new HttpEntity<>(headers);
    
    ResponseEntity<User> response = restTemplate.exchange(
        url,
        HttpMethod.GET,
        entity,
        User.class,
        userId
    );
    
    return response.getBody();
}
```

### 2. POST Request

#### Simple POST Request
```java
public User createUser(User user) {
    String url = "http://api.example.com/users";
    return restTemplate.postForObject(url, user, User.class);
}
```

#### POST Request with Headers
```java
public User createUserWithHeaders(User user) {
    String url = "http://api.example.com/users";
    
    HttpHeaders headers = new HttpHeaders();
    headers.setContentType(MediaType.APPLICATION_JSON);
    headers.set("Authorization", "Bearer " + token);
    
    HttpEntity<User> request = new HttpEntity<>(user, headers);
    
    ResponseEntity<User> response = restTemplate.exchange(
        url,
        HttpMethod.POST,
        request,
        User.class
    );
    
    return response.getBody();
}
```

### 3. PUT Request

```java
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
```

### 4. PATCH Request

```java
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
```

### 5. DELETE Request

```java
public void deleteUser(String userId) {
    String url = "http://api.example.com/users/{id}";
    restTemplate.delete(url, userId);
}
```

## Error Handling

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        restTemplate.setErrorHandler(new DefaultResponseErrorHandler() {
            @Override
            public void handleError(ClientHttpResponse response) throws IOException {
                if (response.getStatusCode() == HttpStatus.NOT_FOUND) {
                    throw new ResourceNotFoundException("Resource not found");
                }
                if (response.getStatusCode() == HttpStatus.BAD_REQUEST) {
                    throw new BadRequestException("Invalid request");
                }
                super.handleError(response);
            }
        });
        
        return restTemplate;
    }
}
```

## Best Practices

1. **Use Dependency Injection**: Always inject RestTemplate using `@Autowired` rather than creating new instances.
2. **Handle Exceptions**: Implement proper error handling using `ResponseErrorHandler`.
3. **Use Headers**: Always set appropriate headers for authentication and content type.
4. **Use URI Variables**: Use URI variables instead of string concatenation for URLs.
5. **Consider Timeouts**: Configure timeouts for better error handling:
   ```java
   @Bean
   public RestTemplate restTemplate() {
       SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
       factory.setConnectTimeout(3000);
       factory.setReadTimeout(3000);
       return new RestTemplate(factory);
   }
   ```

## Note

As of Spring 5.0, RestTemplate is in maintenance mode, with only requests for bugs and minor features being considered. For new projects, consider using the WebClient interface, which offers a more modern, reactive approach to HTTP requests.
