# Java Functional Programming Guide

*Published on March 30, 2024*

## Overview

This guide provides a comprehensive overview of Functional Programming in Java, covering everything from basic concepts to advanced patterns. It includes practical examples, common problems, and their solutions. Whether you're new to functional programming or looking to enhance your existing knowledge, this guide will help you understand and implement functional programming principles in Java.

## Table of Contents

1. [What is Functional Programming?](#what-is-functional-programming)
2. [Key Concepts](#key-concepts)
3. [Functional Interfaces](#functional-interfaces)
4. [Lambda Expressions](#lambda-expressions)
5. [Method References](#method-references)
6. [Streams and Functional Programming](#streams-and-functional-programming)
7. [Common Problems and Solutions](#common-problems-and-solutions)
8. [Best Practices](#best-practices)
9. [References](#references)

## What is Functional Programming?

Functional Programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data. In Java, functional programming features were introduced in Java 8.

### Key Principles:
- Immutability
- Pure functions
- First-class functions
- Higher-order functions
- Function composition
- Declarative programming

## Key Concepts

### 1. Immutability
```java
// Mutable approach
class MutablePerson {
    private String name;
    public void setName(String name) { this.name = name; }
}

// Immutable approach
class ImmutablePerson {
    private final String name;
    public ImmutablePerson(String name) { this.name = name; }
    public String getName() { return name; }
}
```

### 2. Pure Functions
```java
// Impure function (has side effects)
int counter = 0;
public int increment() {
    return ++counter;
}

// Pure function (no side effects)
public int add(int a, int b) {
    return a + b;
}
```

### 3. Function Composition
```java
Function<Integer, Integer> multiplyByTwo = x -> x * 2;
Function<Integer, Integer> addThree = x -> x + 3;

// Compose functions
Function<Integer, Integer> multiplyAndAdd = multiplyByTwo.andThen(addThree);
int result = multiplyAndAdd.apply(5); // Result: 13
```

## Functional Interfaces

### 1. Predicate
```java
Predicate<String> isLongerThan5 = str -> str.length() > 5;
boolean result = isLongerThan5.test("Hello"); // false
```

### 2. Function
```java
Function<String, Integer> stringLength = String::length;
int length = stringLength.apply("Hello"); // 5
```

### 3. Consumer
```java
Consumer<String> printUpperCase = str -> System.out.println(str.toUpperCase());
printUpperCase.accept("hello"); // Prints "HELLO"
```

### 4. Supplier
```java
Supplier<Double> randomNumber = Math::random;
double number = randomNumber.get();
```

## Lambda Expressions

### Basic Syntax
```java
// Traditional approach
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda approach
Runnable runnable = () -> System.out.println("Hello");
```

### Lambda with Parameters
```java
// Single parameter
Function<String, Integer> length = s -> s.length();

// Multiple parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

## Method References

### Types of Method References
```java
// Static method reference
Function<String, Integer> parseInt = Integer::parseInt;

// Instance method reference
Function<String, String> toUpperCase = String::toUpperCase;

// Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;
```

## Streams and Functional Programming

### 1. Filtering and Mapping
```java
List<String> names = Arrays.asList("John", "Jane", "Adam", "Eve");
List<String> filteredNames = names.stream()
    .filter(name -> name.startsWith("J"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 2. Reduction
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .reduce(0, Integer::sum);
```

## Common Problems and Solutions

### Problem 1: State Management
**Problem**: How to handle state in a functional way?
```java
// Non-functional approach
class Counter {
    private int count = 0;
    public int increment() {
        return ++count;
    }
}
```

**Solution**: Use immutable state and return new instances
```java
class Counter {
    private final int count;
    public Counter(int count) { this.count = count; }
    public Counter increment() {
        return new Counter(count + 1);
    }
    public int getCount() { return count; }
}
```

### Problem 2: Exception Handling
**Problem**: How to handle exceptions in functional pipelines?
```java
// Problematic approach
List<String> numbers = Arrays.asList("1", "2", "three", "4");
numbers.stream()
    .map(Integer::parseInt) // Throws NumberFormatException
    .collect(Collectors.toList());
```

**Solution**: Use Either pattern or Optional
```java
// Using Optional
List<String> numbers = Arrays.asList("1", "2", "three", "4");
List<Integer> validNumbers = numbers.stream()
    .map(str -> {
        try {
            return Optional.of(Integer.parseInt(str));
        } catch (NumberFormatException e) {
            return Optional.<Integer>empty();
        }
    })
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());
```

### Problem 3: Side Effects
**Problem**: How to handle operations with side effects?
```java
// Problematic approach
List<String> names = Arrays.asList("John", "Jane");
names.stream()
    .forEach(name -> System.out.println(name)); // Side effect
```

**Solution**: Separate side effects from pure operations
```java
List<String> names = Arrays.asList("John", "Jane");
String result = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.joining(", "));
System.out.println(result); // Side effect isolated
```

### Problem 4: Recursion
**Problem**: How to handle recursion without stack overflow?
```java
// Problematic approach
public int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

**Solution**: Use tail recursion
```java
public int factorial(int n) {
    return factorialHelper(n, 1);
}

private int factorialHelper(int n, int acc) {
    if (n <= 1) return acc;
    return factorialHelper(n - 1, n * acc);
}
```

### Problem 5: Parallel Processing
**Problem**: How to handle parallel processing safely?
```java
// Problematic approach
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.parallelStream()
    .reduce(0, (a, b) -> a + b);
```

**Solution**: Use appropriate collectors and ensure thread safety
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int sum = numbers.parallelStream()
    .collect(Collectors.summingInt(Integer::intValue));
```

## Best Practices

1. **Immutability**:
   - Use final fields
   - Return new instances instead of modifying state
   - Use immutable collections
   - Avoid side effects

2. **Function Design**:
   - Keep functions small and focused
   - Use pure functions when possible
   - Compose functions effectively
   - Use meaningful names

3. **Error Handling**:
   - Use Optional for nullable values
   - Implement Either pattern for error handling
   - Handle exceptions at appropriate levels
   - Use functional error handling patterns

4. **Performance**:
   - Use streams appropriately
   - Consider parallel streams for large datasets
   - Avoid unnecessary object creation
   - Use primitive streams when possible

## References

- [Java Functional Programming Documentation](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
- [Functional Programming in Java](https://www.baeldung.com/java-functional-programming)
- [Java 8 Stream Tutorial](https://www.baeldung.com/java-8-streams)
- [Java Functional Interfaces](https://www.baeldung.com/java-8-functional-interfaces)
- [Java Method References](https://www.baeldung.com/java-method-references)
