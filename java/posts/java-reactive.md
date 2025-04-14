# Java Reactive Programming Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of Reactive Programming in Java, focusing on Project Reactor and Spring WebFlux. It covers everything from basic concepts to advanced patterns, including practical examples and best practices. Whether you're new to reactive programming or looking to enhance your existing knowledge, this guide will help you understand and implement reactive patterns effectively.

## Table of Contents

1. [What is Reactive Programming?](#what-is-reactive-programming)
2. [Reactive Streams](#reactive-streams)
3. [Project Reactor](#project-reactor)
4. [Mono and Flux](#mono-and-flux)
5. [Operators](#operators)
6. [Error Handling](#error-handling)
7. [Backpressure](#backpressure)
8. [Spring WebFlux](#spring-webflux)
9. [Testing Reactive Code](#testing-reactive-code)
10. [Best Practices](#best-practices)
11. [Common Use Cases](#common-use-cases)
12. [References](#references)

## What is Reactive Programming?

Reactive Programming is a programming paradigm that deals with asynchronous data streams and the propagation of change. It's based on the Observer pattern and provides a way to handle asynchronous data flows.

### Key Principles:
- Responsive: Systems respond in a timely manner
- Resilient: Systems stay responsive in the face of failure
- Elastic: Systems stay responsive under varying workload
- Message-driven: Systems rely on asynchronous message-passing

## Reactive Streams

Reactive Streams is a specification for asynchronous stream processing with non-blocking backpressure.

### Key Components:
- Publisher: Emits data
- Subscriber: Consumes data
- Subscription: Controls the flow
- Processor: Transforms data

```java
// Publisher
Publisher<String> publisher = new Publisher<String>() {
    @Override
    public void subscribe(Subscriber<? super String> subscriber) {
        subscriber.onSubscribe(new Subscription() {
            @Override
            public void request(long n) {
                subscriber.onNext("Data");
                subscriber.onComplete();
            }
            
            @Override
            public void cancel() {}
        });
    }
};

// Subscriber
Subscriber<String> subscriber = new Subscriber<String>() {
    @Override
    public void onSubscribe(Subscription subscription) {
        subscription.request(1);
    }
    
    @Override
    public void onNext(String item) {
        System.out.println("Received: " + item);
    }
    
    @Override
    public void onError(Throwable throwable) {
        System.err.println("Error: " + throwable.getMessage());
    }
    
    @Override
    public void onComplete() {
        System.out.println("Completed");
    }
};

// Connect publisher and subscriber
publisher.subscribe(subscriber);
```

## Project Reactor

Project Reactor is a fully non-blocking reactive programming foundation for the JVM.

### Dependencies:
```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.4.0</version>
</dependency>
```

## Mono and Flux

### 1. Mono (0-1 items)
```java
// Create Mono
Mono<String> mono = Mono.just("Hello");

// Subscribe to Mono
mono.subscribe(
    value -> System.out.println("Received: " + value),
    error -> System.err.println("Error: " + error),
    () -> System.out.println("Completed")
);

// Output:
// Received: Hello
// Completed
```

### 2. Flux (0-N items)
```java
// Create Flux
Flux<String> flux = Flux.just("A", "B", "C");

// Subscribe to Flux
flux.subscribe(
    value -> System.out.println("Received: " + value),
    error -> System.err.println("Error: " + error),
    () -> System.out.println("Completed")
);

// Output:
// Received: A
// Received: B
// Received: C
// Completed
```

## Operators

### 1. Transform Operators
```java
Flux<Integer> numbers = Flux.just(1, 2, 3, 4, 5);

// Map
numbers.map(n -> n * 2)
    .subscribe(System.out::println);

// Output:
// 2
// 4
// 6
// 8
// 10

// Filter
numbers.filter(n -> n % 2 == 0)
    .subscribe(System.out::println);

// Output:
// 2
// 4
```

### 2. Combine Operators
```java
Flux<String> flux1 = Flux.just("A", "B");
Flux<String> flux2 = Flux.just("C", "D");

// Merge
Flux.merge(flux1, flux2)
    .subscribe(System.out::println);

// Output:
// A
// B
// C
// D

// Zip
Flux.zip(flux1, flux2, (s1, s2) -> s1 + s2)
    .subscribe(System.out::println);

// Output:
// AC
// BD
```

### 3. Time-based Operators
```java
// Delay elements
Flux.just("A", "B", "C")
    .delayElements(Duration.ofSeconds(1))
    .subscribe(System.out::println);

// Output: (with 1-second delay between each)
// A
// B
// C

// Timeout
Flux.just("A")
    .delayElements(Duration.ofSeconds(2))
    .timeout(Duration.ofSeconds(1))
    .subscribe(
        System.out::println,
        error -> System.err.println("Timeout: " + error)
    );

// Output: (after 1 second)
// Timeout: java.util.concurrent.TimeoutException
```

## Error Handling

### 1. onErrorReturn
```java
Flux.just(1, 2, 0, 4)
    .map(i -> 10 / i)
    .onErrorReturn(-1)
    .subscribe(System.out::println);

// Output:
// 10
// 5
// -1
```

### 2. onErrorResume
```java
Flux.just(1, 2, 0, 4)
    .map(i -> 10 / i)
    .onErrorResume(e -> Flux.just(-1, -2, -3))
    .subscribe(System.out::println);

// Output:
// 10
// 5
// -1
// -2
// -3
```

### 3. retry
```java
Flux.just(1, 2, 0, 4)
    .map(i -> 10 / i)
    .retry(2)
    .subscribe(
        System.out::println,
        error -> System.err.println("Error: " + error)
    );

// Output:
// 10
// 5
// 10
// 5
// 10
// 5
// Error: java.lang.ArithmeticException: / by zero
```

## Backpressure

### 1. Request Strategy
```java
Flux.range(1, 100)
    .subscribe(
        new BaseSubscriber<Integer>() {
            @Override
            protected void hookOnSubscribe(Subscription subscription) {
                request(5); // Request 5 items initially
            }
            
            @Override
            protected void hookOnNext(Integer value) {
                System.out.println("Received: " + value);
                if (value % 5 == 0) {
                    request(5); // Request 5 more items
                }
            }
        }
    );
```

### 2. Backpressure Strategies
```java
// Buffer
Flux.range(1, 1000)
    .onBackpressureBuffer(10)
    .subscribe(System.out::println);

// Drop
Flux.range(1, 1000)
    .onBackpressureDrop()
    .subscribe(System.out::println);

// Latest
Flux.range(1, 1000)
    .onBackpressureLatest()
    .subscribe(System.out::println);
```

## Spring WebFlux

### 1. Controller Example
```java
@RestController
public class UserController {
    
    @GetMapping("/users")
    public Flux<User> getUsers() {
        return userRepository.findAll();
    }
    
    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable String id) {
        return userRepository.findById(id);
    }
    
    @PostMapping("/users")
    public Mono<User> createUser(@RequestBody Mono<User> user) {
        return user.flatMap(userRepository::save);
    }
}
```

### 2. Router Function Example
```java
@Configuration
public class RouterConfig {
    
    @Bean
    public RouterFunction<ServerResponse> routes(UserHandler handler) {
        return RouterFunctions.route()
            .GET("/users", handler::getUsers)
            .GET("/users/{id}", handler::getUser)
            .POST("/users", handler::createUser)
            .build();
    }
}
```

## Testing Reactive Code

### 1. StepVerifier
```java
@Test
public void testFlux() {
    Flux<String> flux = Flux.just("A", "B", "C");
    
    StepVerifier.create(flux)
        .expectNext("A")
        .expectNext("B")
        .expectNext("C")
        .verifyComplete();
}
```

### 2. TestPublisher
```java
@Test
public void testWithTestPublisher() {
    TestPublisher<String> testPublisher = TestPublisher.create();
    
    Flux<String> flux = testPublisher.flux();
    
    StepVerifier.create(flux)
        .then(() -> testPublisher.next("A"))
        .expectNext("A")
        .then(() -> testPublisher.complete())
        .verifyComplete();
}
```

## Best Practices

1. **Error Handling**:
   - Use appropriate error handling operators
   - Implement fallback mechanisms
   - Log errors appropriately
   - Consider retry strategies

2. **Backpressure**:
   - Implement proper backpressure strategies
   - Monitor system resources
   - Use appropriate buffering
   - Consider system capacity

3. **Performance**:
   - Use appropriate schedulers
   - Minimize blocking operations
   - Optimize operator chains
   - Monitor system metrics

4. **Testing**:
   - Use StepVerifier for testing
   - Test error scenarios
   - Verify backpressure
   - Test time-based operations

## Common Use Cases

1. **Web Applications**:
   - Reactive REST APIs
   - WebSocket communication
   - Server-sent events
   - Real-time updates

2. **Data Processing**:
   - Stream processing
   - Data transformation
   - Event processing
   - Batch processing

3. **Microservices**:
   - Service communication
   - Circuit breaking
   - Rate limiting
   - Load balancing

4. **IoT Applications**:
   - Sensor data processing
   - Real-time analytics
   - Event-driven systems
   - Device management

## References

- [Project Reactor Documentation](https://projectreactor.io/docs/core/release/reference/)
- [Spring WebFlux Documentation](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)
- [Reactive Streams Specification](https://www.reactive-streams.org/)
- [Reactive Programming Guide](https://www.baeldung.com/reactor-core)
- [Spring WebFlux Guide](https://www.baeldung.com/spring-webflux)
