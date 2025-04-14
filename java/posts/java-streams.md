# Java Streams Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of Java Streams, introduced in Java 8. It covers everything from basic concepts to advanced operations, including stream creation, intermediate and terminal operations, parallel streams, and best practices. Whether you're new to Java Streams or looking to optimize your existing stream operations, this guide will help you understand and implement best practices for working with Java Streams.

## Table of Contents

1. [What are Java Streams?](#what-are-java-streams)
2. [Getting Started](#getting-started)
3. [Stream Creation](#stream-creation)
4. [Intermediate Operations](#intermediate-operations)
5. [Terminal Operations](#terminal-operations)
6. [Parallel Streams](#parallel-streams)
7. [Collectors](#collectors)
8. [Best Practices](#best-practices)
9. [Common Use Cases](#common-use-cases)
10. [References](#references)

## What are Java Streams?

Java Streams are a sequence of elements supporting sequential and parallel aggregate operations. They provide:

- Functional-style operations
- Lazy evaluation
- Parallel processing
- Immutable data processing
- Chainable operations

### Key Features:
- Declarative programming style
- Internal iteration
- Parallel processing support
- Immutable data processing
- Chainable operations
- Lazy evaluation

### Common Use Cases:
- Data filtering and transformation
- Aggregation operations
- Data grouping and partitioning
- Parallel data processing
- Complex data pipelines
- Collection processing

## Getting Started

### Prerequisites
- Java 8 or later
- Basic understanding of Java collections
- Familiarity with lambda expressions
- Understanding of functional programming concepts

### Basic Example
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");

// Traditional approach
List<String> filteredNames = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("J")) {
        filteredNames.add(name.toUpperCase());
    }
}

// Stream approach
List<String> filteredNames = names.stream()
    .filter(name -> name.startsWith("J"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

## Stream Creation

### 1. From Collections
```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
```

### 2. From Arrays
```java
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);
```

### 3. From Values
```java
Stream<String> stream = Stream.of("a", "b", "c");
```

### 4. From Functions
```java
// Infinite stream
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);

// Limited stream
Stream<Integer> limitedStream = Stream.iterate(0, n -> n + 2)
    .limit(10);
```

## Intermediate Operations

### 1. Filter
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> filteredNames = names.stream()
    .filter(name -> name.startsWith("J"))
    .collect(Collectors.toList());
```

### 2. Map
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<Integer> nameLengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

### 3. FlatMap
```java
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);
List<String> flattenedList = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
```

### 4. Distinct
```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3);
List<Integer> distinctNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
```

### 5. Sorted
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> sortedNames = names.stream()
    .sorted()
    .collect(Collectors.toList());
```

## Terminal Operations

### 1. Collect
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> result = names.stream()
    .filter(name -> name.startsWith("J"))
    .collect(Collectors.toList());
```

### 2. ForEach
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
names.stream()
    .forEach(System.out::println);
```

### 3. Reduce
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .reduce(0, Integer::sum);
```

### 4. Count
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
long count = names.stream()
    .filter(name -> name.startsWith("J"))
    .count();
```

### 5. AnyMatch/AllMatch/NoneMatch
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
boolean anyStartsWithJ = names.stream()
    .anyMatch(name -> name.startsWith("J"));
```

## Parallel Streams

### 1. Basic Usage
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> result = names.parallelStream()
    .filter(name -> name.startsWith("J"))
    .collect(Collectors.toList());
```

### 2. Performance Considerations
```java
// Use parallel streams for large datasets
List<Integer> largeList = // ... large list
List<Integer> result = largeList.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

## Collectors

### 1. ToList/ToSet
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> result = names.stream()
    .collect(Collectors.toList());

Set<String> uniqueNames = names.stream()
    .collect(Collectors.toSet());
```

### 2. ToMap
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
Map<String, Integer> nameLengthMap = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        String::length
    ));
```

### 3. GroupingBy
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
Map<Integer, List<String>> namesByLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
```

### 4. Joining
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
String joinedNames = names.stream()
    .collect(Collectors.joining(", "));
```

## Best Practices

1. **Stream Creation**:
   - Use appropriate stream sources
   - Consider parallel streams for large datasets
   - Avoid creating unnecessary streams
   - Use primitive streams when possible
   - Consider stream characteristics

2. **Operations**:
   - Chain operations effectively
   - Use appropriate intermediate operations
   - Choose correct terminal operations
   - Consider operation order
   - Avoid stateful operations

3. **Performance**:
   - Use parallel streams judiciously
   - Consider stream characteristics
   - Avoid unnecessary operations
   - Use appropriate collectors
   - Consider memory usage

4. **Maintenance**:
   - Write readable stream operations
   - Document complex operations
   - Test stream operations
   - Consider error handling
   - Follow functional principles

## Common Use Cases

1. **Data Filtering**:
```java
List<Person> adults = people.stream()
    .filter(person -> person.getAge() >= 18)
    .collect(Collectors.toList());
```

2. **Data Transformation**:
```java
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

3. **Data Aggregation**:
```java
double averageAge = people.stream()
    .mapToInt(Person::getAge)
    .average()
    .orElse(0);
```

4. **Data Grouping**:
```java
Map<String, List<Person>> peopleByCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));
```

## References

- [Java Stream Documentation](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/stream/package-summary.html)
- [Java 8 Stream Tutorial](https://www.baeldung.com/java-8-streams)
- [Java Stream Best Practices](https://www.oracle.com/technical-resources/articles/java/ma14-java-se-8-streams.html)
- [Java Functional Programming](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Java Collectors Guide](https://www.baeldung.com/java-8-collectors)
