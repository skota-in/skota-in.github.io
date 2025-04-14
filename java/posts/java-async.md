# Java Asynchronous Programming Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of asynchronous programming in Java, focusing on CompletableFuture introduced in Java 8. It covers everything from basic concepts to advanced patterns, including practical examples with expected outputs. Whether you're new to asynchronous programming or looking to enhance your existing knowledge, this guide will help you understand and implement asynchronous patterns effectively.

## Table of Contents

1. [What is Asynchronous Programming?](#what-is-asynchronous-programming)
2. [CompletableFuture Basics](#completablefuture-basics)
3. [Creating CompletableFuture](#creating-completablefuture)
4. [Chaining Operations](#chaining-operations)
5. [Combining Futures](#combining-futures)
6. [Error Handling](#error-handling)
7. [Advanced Patterns](#advanced-patterns)
8. [Best Practices](#best-practices)
9. [Common Use Cases](#common-use-cases)
10. [References](#references)

## What is Asynchronous Programming?

Asynchronous programming allows tasks to run independently of the main program flow, enabling better resource utilization and improved application responsiveness.

### Key Concepts:
- Non-blocking operations
- Callback-based programming
- Future-based programming
- Reactive programming
- Parallel execution

## CompletableFuture Basics

CompletableFuture is a class that implements the Future interface and provides a rich API for asynchronous programming.

### Basic Example:
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        Thread.sleep(1000); // Simulate long-running task
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Hello, World!";
});

// Output: (after 1 second)
// Result: Hello, World!
future.thenAccept(result -> System.out.println("Result: " + result));
```

## Creating CompletableFuture

### 1. Using supplyAsync
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Task completed";
});

// Output:
// Task completed
future.thenAccept(System.out::println);
```

### 2. Using runAsync
```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("Running async task");
});

// Output:
// Running async task
future.join();
```

### 3. Using completedFuture
```java
CompletableFuture<String> future = CompletableFuture.completedFuture("Immediate result");

// Output:
// Immediate result
future.thenAccept(System.out::println);
```

## Chaining Operations

### 1. thenApply (Transform result)
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenApply(String::toUpperCase);

// Output:
// HELLO WORLD
future.thenAccept(System.out::println);
```

### 2. thenCompose (Chain futures)
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + " World"));

// Output:
// Hello World
future.thenAccept(System.out::println);
```

### 3. thenAccept (Consume result)
```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenAccept(System.out::println);

// Output:
// Hello World
```

### 4. thenRun (Run after completion)
```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenRun(() -> System.out.println("Task completed"));

// Output:
// Task completed
```

## Combining Futures

### 1. thenCombine (Combine two futures)
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

CompletableFuture<String> combined = future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2);

// Output:
// Hello World
combined.thenAccept(System.out::println);
```

### 2. allOf (Wait for all futures)
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Task 1");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "Task 2");
CompletableFuture<String> future3 = CompletableFuture.supplyAsync(() -> "Task 3");

CompletableFuture<Void> allFutures = CompletableFuture.allOf(future1, future2, future3);

// Output:
// All tasks completed
allFutures.thenRun(() -> System.out.println("All tasks completed"));
```

### 3. anyOf (Wait for any future)
```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try { Thread.sleep(2000); } catch (InterruptedException e) {}
    return "Task 1";
});
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try { Thread.sleep(1000); } catch (InterruptedException e) {}
    return "Task 2";
});

CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(future1, future2);

// Output: (after 1 second)
// First completed: Task 2
anyFuture.thenAccept(result -> System.out.println("First completed: " + result));
```

## Error Handling

### 1. exceptionally (Handle exceptions)
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (true) throw new RuntimeException("Error occurred");
    return "Success";
}).exceptionally(ex -> "Recovered from: " + ex.getMessage());

// Output:
// Recovered from: java.lang.RuntimeException: Error occurred
future.thenAccept(System.out::println);
```

### 2. handle (Handle both success and failure)
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    if (Math.random() > 0.5) throw new RuntimeException("Random error");
    return "Success";
}).handle((result, ex) -> {
    if (ex != null) return "Recovered from: " + ex.getMessage();
    return result;
});

// Output: (random)
// Either: Success
// Or: Recovered from: java.lang.RuntimeException: Random error
future.thenAccept(System.out::println);
```

## Advanced Patterns

### 1. Timeout Handling
```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try { Thread.sleep(2000); } catch (InterruptedException e) {}
    return "Result";
}).orTimeout(1, TimeUnit.SECONDS)
  .exceptionally(ex -> "Timeout occurred");

// Output: (after 1 second)
// Timeout occurred
future.thenAccept(System.out::println);
```

### 2. Retry Pattern
```java
public CompletableFuture<String> retryOperation(int maxRetries) {
    return CompletableFuture.supplyAsync(() -> {
        if (Math.random() > 0.7) return "Success";
        throw new RuntimeException("Operation failed");
    }).exceptionally(ex -> {
        if (maxRetries > 0) return retryOperation(maxRetries - 1).join();
        return "All retries failed";
    });
}

// Output: (random)
// Either: Success
// Or: All retries failed
retryOperation(3).thenAccept(System.out::println);
```

### 3. Parallel Processing
```java
List<String> items = Arrays.asList("1", "2", "3", "4", "5");

List<CompletableFuture<String>> futures = items.stream()
    .map(item -> CompletableFuture.supplyAsync(() -> {
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        return "Processed " + item;
    }))
    .collect(Collectors.toList());

CompletableFuture<Void> allFutures = CompletableFuture.allOf(
    futures.toArray(new CompletableFuture[0]));

// Output: (after 1 second)
// Processed 1
// Processed 2
// Processed 3
// Processed 4
// Processed 5
allFutures.thenRun(() -> {
    futures.stream()
        .map(CompletableFuture::join)
        .forEach(System.out::println);
});
```

## Best Practices

1. **Error Handling**:
   - Always handle exceptions
   - Use appropriate recovery strategies
   - Log errors appropriately
   - Consider retry mechanisms

2. **Resource Management**:
   - Use appropriate thread pools
   - Avoid blocking operations
   - Clean up resources properly
   - Monitor resource usage

3. **Performance**:
   - Use appropriate timeouts
   - Consider parallel processing
   - Avoid unnecessary blocking
   - Monitor execution time

4. **Code Organization**:
   - Keep async operations focused
   - Use meaningful names
   - Document complex flows
   - Consider testing strategies

## Common Use Cases

1. **Web Applications**:
   - Asynchronous API calls
   - Parallel data processing
   - Background tasks
   - Real-time updates

2. **Data Processing**:
   - Parallel data transformation
   - Batch processing
   - Stream processing
   - Data aggregation

3. **Microservices**:
   - Service composition
   - Circuit breaking
   - Retry mechanisms
   - Timeout handling

4. **UI Applications**:
   - Background tasks
   - Progress updates
   - Responsive UI
   - Error handling

## References

- [Java CompletableFuture Documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
- [Java Concurrency Tutorial](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- [CompletableFuture Guide](https://www.baeldung.com/java-completablefuture)
- [Asynchronous Programming in Java](https://www.baeldung.com/java-asynchronous-programming)
- [Java Concurrency Patterns](https://www.baeldung.com/java-concurrency-patterns)
