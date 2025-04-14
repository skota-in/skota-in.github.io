# Spring Boot Response Types

*Published on January 30, 2025*

Spring Boot provides excellent support for handling different types of responses in your REST APIs. This guide covers various response types and how to implement them.

## 1. JSON Response

JSON is the most common response type in modern web applications. Spring Boot automatically handles JSON serialization/deserialization using Jackson.

```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
}
```

## 2. XML Response

To enable XML responses, add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

Then you can specify XML response using `produces` attribute:

```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    @GetMapping(value = "/users", produces = MediaType.APPLICATION_XML_VALUE)
    public List<User> getUsers() {
        return userService.findAll();
    }
}
```

## 3. Binary Response (PDF, Images, etc.)

For binary responses like PDFs, images, or other files:

```java
@RestController
@RequestMapping("/api")
public class FileController {
    
    @GetMapping(value = "/pdf", produces = MediaType.APPLICATION_PDF_VALUE)
    public ResponseEntity<Resource> getPdf() {
        InputStreamResource resource = new InputStreamResource(new FileInputStream("report.pdf"));
        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, "attachment;filename=report.pdf")
            .body(resource);
    }
    
    @GetMapping(value = "/image", produces = MediaType.IMAGE_JPEG_VALUE)
    public ResponseEntity<Resource> getImage() {
        InputStreamResource resource = new InputStreamResource(new FileInputStream("image.jpg"));
        return ResponseEntity.ok()
            .body(resource);
    }
}
```

## 4. Custom Response Types

You can create custom response types by implementing your own `HttpMessageConverter`:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        converters.add(new CustomMessageConverter());
    }
}

public class CustomMessageConverter extends AbstractHttpMessageConverter<CustomObject> {
    
    public CustomMessageConverter() {
        super(new MediaType("application", "custom"));
    }
    
    @Override
    protected boolean supports(Class<?> clazz) {
        return CustomObject.class.isAssignableFrom(clazz);
    }
    
    @Override
    protected CustomObject readInternal(Class<? extends CustomObject> clazz, 
                                      HttpInputMessage inputMessage) {
        // Implement reading logic
    }
    
    @Override
    protected void writeInternal(CustomObject object, HttpOutputMessage outputMessage) {
        // Implement writing logic
    }
}
```

## 5. Response Entity

`ResponseEntity` provides more control over the HTTP response:

```java
@RestController
@RequestMapping("/api")
public class UserController {
    
    @GetMapping("/users/{id}")
    public ResponseEntity<?> getUser(@PathVariable Long id) {
        try {
            User user = userService.findById(id);
            return ResponseEntity.ok(user);
        } catch (UserNotFoundException e) {
            return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse("User not found"));
        }
    }
}
```

## 6. Streaming Response

For large files or streaming data:

```java
@GetMapping(value = "/stream", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
public StreamingResponseBody streamData() {
    return outputStream -> {
        // Write data to outputStream
        for (int i = 0; i < 1000; i++) {
            outputStream.write(("Data " + i + "\n").getBytes());
            outputStream.flush();
        }
    };
}
```

## Best Practices

1. Always specify the correct `Content-Type` header
2. Use appropriate HTTP status codes
3. Consider implementing proper error handling
4. Use compression for large responses
5. Implement proper caching strategies
6. Consider using pagination for large datasets

## Conclusion

Spring Boot provides flexible options for handling different response types. Choose the appropriate response type based on your application's requirements and client needs.
