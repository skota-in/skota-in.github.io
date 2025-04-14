# Java 11 Features Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of Java 11 features, which was released in September 2018 as a Long-Term Support (LTS) release. Java 11 introduced several significant improvements, including new APIs, performance enhancements, and language features. This guide covers all major features with practical examples and best practices.

## Table of Contents

1. [What's New in Java 11?](#whats-new-in-java-11)
2. [Local-Variable Syntax for Lambda Parameters](#local-variable-syntax-for-lambda-parameters)
3. [New String Methods](#new-string-methods)
4. [New File Methods](#new-file-methods)
5. [HTTP Client API](#http-client-api)
6. [Nest-Based Access Control](#nest-based-access-control)
7. [Dynamic Class-File Constants](#dynamic-class-file-constants)
8. [Epsilon Garbage Collector](#epsilon-garbage-collector)
9. [Flight Recorder](#flight-recorder)
10. [Best Practices](#best-practices)
11. [Common Use Cases](#common-use-cases)
12. [References](#references)

## What's New in Java 11?

Java 11 introduced several major features and improvements:

- Local-Variable Syntax for Lambda Parameters
- New String Methods
- New File Methods
- Standard HTTP Client API
- Nest-Based Access Control
- Dynamic Class-File Constants
- Epsilon Garbage Collector
- Flight Recorder
- ZGC (Experimental)
- Removal of Java EE and CORBA Modules
- Unicode 10 Support
- TLS 1.3 Support

## Local-Variable Syntax for Lambda Parameters

Java 11 allows the use of `var` in lambda parameters, making the syntax more consistent with local variable declarations.

```java
// Before Java 11
BiFunction<String, String, String> concat = (String a, String b) -> a + b;

// With Java 11
BiFunction<String, String, String> concat = (var a, var b) -> a + b;
```

### Benefits:
- Consistent syntax with local variables
- Better readability in some cases
- Type inference still works
- Maintains backward compatibility

## New String Methods

Java 11 added several useful methods to the String class:

### 1. `isBlank()`
```java
String str = "   ";
boolean isBlank = str.isBlank(); // true
```

### 2. `lines()`
```java
String text = "Line 1\nLine 2\nLine 3";
text.lines().forEach(System.out::println);
```

### 3. `strip()`, `stripLeading()`, `stripTrailing()`
```java
String str = "  Hello World  ";
String stripped = str.strip(); // "Hello World"
String leadingStripped = str.stripLeading(); // "Hello World  "
String trailingStripped = str.stripTrailing(); // "  Hello World"
```

### 4. `repeat()`
```java
String str = "Java";
String repeated = str.repeat(3); // "JavaJavaJava"
```

## New File Methods

Java 11 added new methods to the `Files` class for better file handling:

### 1. `readString()` and `writeString()`
```java
// Reading a file
String content = Files.readString(Path.of("file.txt"));

// Writing to a file
Files.writeString(Path.of("output.txt"), "Hello World");
```

### 2. `isSameFile()` with Path
```java
Path path1 = Path.of("file1.txt");
Path path2 = Path.of("file2.txt");
boolean isSame = Files.isSameFile(path1, path2);
```

## HTTP Client API

Java 11 standardized the HTTP Client API that was introduced in Java 9 as an incubating feature:

```java
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .connectTimeout(Duration.ofSeconds(5))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .GET()
    .build();

HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());

System.out.println(response.statusCode());
System.out.println(response.body());
```

### Features:
- HTTP/2 support
- WebSocket support
- Asynchronous requests
- Connection pooling
- Request/response interceptors

## Nest-Based Access Control

Java 11 introduced nest-based access control, which allows classes that are logically part of the same code entity to access each other's private members:

```java
public class Outer {
    private String outerField = "Outer";
    
    class Inner {
        private String innerField = "Inner";
        
        void accessOuter() {
            System.out.println(outerField); // Now works without synthetic accessors
        }
    }
    
    void accessInner() {
        Inner inner = new Inner();
        System.out.println(inner.innerField); // Now works without synthetic accessors
    }
}
```

## Dynamic Class-File Constants

Java 11 introduced a new constant pool form, `CONSTANT_Dynamic`, which allows the creation of constants that are computed at runtime:

```java
public class DynamicConstants {
    public static void main(String[] args) {
        // Example of dynamic constant usage
        var constant = new DynamicConstant();
        System.out.println(constant.getValue());
    }
}

class DynamicConstant {
    private static final String VALUE = computeValue();
    
    private static String computeValue() {
        return "Computed at runtime: " + System.currentTimeMillis();
    }
    
    public String getValue() {
        return VALUE;
    }
}
```

## Epsilon Garbage Collector

Java 11 introduced the Epsilon garbage collector, which is a no-op garbage collector:

```bash
# Enable Epsilon GC
java -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC MyApplication
```

### Use Cases:
- Performance testing
- Short-lived applications
- Memory pressure testing
- Benchmarking

## Flight Recorder

Java Flight Recorder (JFR) was made available in OpenJDK:

```bash
# Enable Flight Recorder
java -XX:StartFlightRecording MyApplication
```

### Features:
- Low overhead
- Continuous recording
- Detailed performance data
- Memory usage tracking
- Thread analysis

## Best Practices

1. **HTTP Client**:
   - Use connection pooling
   - Implement proper error handling
   - Set appropriate timeouts
   - Use HTTP/2 when possible

2. **String Operations**:
   - Use new string methods for cleaner code
   - Prefer `strip()` over `trim()`
   - Use `isBlank()` for whitespace checking
   - Leverage `lines()` for text processing

3. **File Operations**:
   - Use new file methods for simplicity
   - Handle exceptions properly
   - Close resources appropriately
   - Use try-with-resources

4. **Performance**:
   - Use Flight Recorder for profiling
   - Consider Epsilon GC for specific use cases
   - Monitor memory usage
   - Profile before optimization

## Common Use Cases

1. **Web Applications**:
   - HTTP client for API calls
   - String processing for data handling
   - File operations for resource management

2. **Data Processing**:
   - Text file processing
   - String manipulation
   - Data transformation

3. **Performance Monitoring**:
   - Flight Recorder for profiling
   - Memory usage analysis
   - Performance optimization

4. **Testing**:
   - Epsilon GC for memory testing
   - Performance benchmarking
   - Resource usage monitoring

## References

- [Java 11 Documentation](https://docs.oracle.com/en/java/javase/11/)
- [Java 11 Release Notes](https://www.oracle.com/java/technologies/javase/11-relnote-issues.html)
- [Java HTTP Client Guide](https://openjdk.java.net/groups/net/httpclient/intro.html)
- [Java Flight Recorder Documentation](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm)
- [Java 11 Features](https://www.baeldung.com/java-11-new-features)
