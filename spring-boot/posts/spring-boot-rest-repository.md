# Spring Boot REST Repository Guide

*Published on July 8, 2023*

## Introduction

Spring Boot REST repositories provide a powerful way to create RESTful APIs with minimal code. They leverage Spring Data JPA to automatically generate CRUD operations and expose them as REST endpoints. This guide will walk you through setting up and using Spring Boot REST repositories.

## Prerequisites

- Java 8 or higher
- Maven or Gradle
- Spring Boot 2.x or higher
- Spring Data JPA
- A database (H2, MySQL, PostgreSQL, etc.)

## Setting Up the Project

### Maven Dependencies

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### Gradle Dependencies

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'com.h2database:h2'
}
```

## Creating a REST Repository

### 1. Define the Entity

```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    private String author;
    private String isbn;
    
    // Getters and Setters
}
```

### 2. Create the Repository Interface

```java
@RepositoryRestResource(collectionResourceRel = "books", path = "books")
public interface BookRepository extends JpaRepository<Book, Long> {
    List<Book> findByAuthor(@Param("author") String author);
    List<Book> findByTitleContaining(@Param("title") String title);
}
```

### 3. Configure the Application

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Available Endpoints

Once you've set up your repository, Spring Boot automatically creates the following REST endpoints:

- `GET /books` - Get all books
- `GET /books/{id}` - Get a specific book
- `POST /books` - Create a new book
- `PUT /books/{id}` - Update a book
- `DELETE /books/{id}` - Delete a book
- `GET /books/search/findByAuthor?author={author}` - Find books by author
- `GET /books/search/findByTitleContaining?title={title}` - Find books by title

## Customizing the API

### Pagination and Sorting

```java
// Example request with pagination and sorting
GET /books?page=0&size=10&sort=title,asc
```

### Custom Query Methods

```java
@RepositoryRestResource
public interface BookRepository extends JpaRepository<Book, Long> {
    @Query("SELECT b FROM Book b WHERE b.publicationYear > :year")
    List<Book> findRecentBooks(@Param("year") int year);
}
```

### Customizing Response Format

```java
@Configuration
public class RepositoryConfig extends RepositoryRestConfigurerAdapter {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        config.setBasePath("/api");
        config.setDefaultPageSize(20);
        config.setMaxPageSize(50);
    }
}
```

## Security Considerations

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .antMatchers("/books/**").authenticated()
            .and()
            .httpBasic();
    }
}
```

## Testing the API

### Using cURL

```bash
# Get all books
curl -X GET http://localhost:8080/books

# Create a new book
curl -X POST http://localhost:8080/books \
  -H "Content-Type: application/json" \
  -d '{"title":"Spring Boot in Action","author":"Craig Walls","isbn":"978-1617292545"}'

# Update a book
curl -X PUT http://localhost:8080/books/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated Title","author":"Same Author","isbn":"978-1617292545"}'
```

## Best Practices

1. Always use proper validation
2. Implement proper error handling
3. Use appropriate HTTP status codes
4. Implement pagination for large datasets
5. Secure your endpoints
6. Use proper documentation (Swagger/OpenAPI)
7. Implement proper logging
8. Use DTOs for complex operations

## Common Issues and Solutions

1. **CORS Issues**
   ```java
   @Configuration
   public class WebConfig implements WebMvcConfigurer {
       @Override
       public void addCorsMappings(CorsRegistry registry) {
           registry.addMapping("/**")
               .allowedOrigins("*")
               .allowedMethods("GET", "POST", "PUT", "DELETE");
       }
   }
   ```

2. **N+1 Query Problem**
   - Use `@EntityGraph` for eager loading
   - Implement proper indexing
   - Use DTO projections

## Conclusion

Spring Boot REST repositories provide a powerful and efficient way to create RESTful APIs. By following this guide, you can quickly set up a robust API with minimal code while maintaining flexibility and control over your endpoints.

## Additional Resources

- [Spring Data JPA Documentation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [REST API Best Practices](https://www.moesif.com/blog/technical/api-design/REST-API-Design-Best-Practices-for-Sub-and-Nested-Resources/)
