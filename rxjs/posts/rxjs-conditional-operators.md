# RxJS Conditional Operators: Examples and Use Cases

*Published on August 15, 2023*

## Introduction

Conditional operators in RxJS allow you to make decisions based on the behavior of Observables. These operators help you control the flow of your reactive streams based on specific conditions, making your code more declarative and easier to reason about. This post explores the conditional operators in RxJS with practical examples.

## Table of Contents

1. [defaultIfEmpty](#defaultifempty)
2. [every](#every)
3. [iif](#iif)
4. [sequenceEqual](#sequenceequal)
5. [Comparison of Conditional Operators](#comparison-of-conditional-operators)

## defaultIfEmpty

**Purpose**: Emits a default value if the source Observable completes without emitting any values.

```typescript
import { from, of } from 'rxjs';
import { defaultIfEmpty } from 'rxjs/operators';

// Example 1: With empty source
const emptySource$ = from([]);

emptySource$.pipe(
  defaultIfEmpty('No data available')
).subscribe(
  value => console.log(`Result: ${value}`)
);
// Output: "Result: No data available"

// Example 2: With non-empty source
const nonEmptySource$ = of(1, 2, 3);

nonEmptySource$.pipe(
  defaultIfEmpty('No data available')
).subscribe(
  value => console.log(`Result: ${value}`)
);
// Output: "Result: 1", "Result: 2", "Result: 3"
```

**When to use**:
- When you need to provide a fallback value for empty Observables
- For handling "no data" scenarios gracefully
- When you want to ensure subscribers always receive at least one value
- In UI components to display default content when data is missing

## every

**Purpose**: Determines whether all items emitted by an Observable satisfy a specified condition, emitting a boolean result.

```typescript
import { from } from 'rxjs';
import { every } from 'rxjs/operators';

// Example 1: All values pass the condition
const allEven$ = from([2, 4, 6, 8]);

allEven$.pipe(
  every(value => value % 2 === 0)
).subscribe(
  result => console.log(`Are all values even? ${result}`)
);
// Output: "Are all values even? true"

// Example 2: Not all values pass the condition
const mixedNumbers$ = from([2, 4, 5, 8]);

mixedNumbers$.pipe(
  every(value => value % 2 === 0)
).subscribe(
  result => console.log(`Are all values even? ${result}`)
);
// Output: "Are all values even? false"
// Note: The result is emitted as soon as a value fails the condition
```

**When to use**:
- For validation scenarios where all items must meet a condition
- When you need to check if a collection uniformly satisfies a predicate
- For implementing "all or nothing" logic
- In form validation where all fields must be valid

## iif

**Purpose**: Decides at subscription time which Observable to subscribe to based on a condition.

```typescript
import { iif, of, throwError } from 'rxjs';

// Function that returns an Observable based on a condition
function getDataObservable(hasPermission: boolean) {
  return iif(
    () => hasPermission,
    // If condition is true, emit data
    of('User data', 'Settings', 'Preferences'),
    // If condition is false, emit error
    throwError(() => new Error('Permission denied'))
  );
}

// Example 1: With permission
console.log('With permission:');
getDataObservable(true).subscribe({
  next: data => console.log(`Data received: ${data}`),
  error: err => console.error(`Error: ${err.message}`)
});
// Output:
// "Data received: User data"
// "Data received: Settings"
// "Data received: Preferences"

// Example 2: Without permission
console.log('Without permission:');
getDataObservable(false).subscribe({
  next: data => console.log(`Data received: ${data}`),
  error: err => console.error(`Error: ${err.message}`)
});
// Output: "Error: Permission denied"
```

**When to use**:
- For conditional subscription logic
- When you need to choose between different Observable sources at runtime
- For feature toggling or permission-based content
- To implement if/else logic in reactive streams

## sequenceEqual

**Purpose**: Compares the current Observable sequence with another Observable sequence, emitting a boolean value indicating whether they emit the same sequence of items.

```typescript
import { from, of } from 'rxjs';
import { sequenceEqual } from 'rxjs/operators';

// Example 1: Sequences are equal
const source1$ = from([1, 2, 3]);
const compareTo1$ = of(1, 2, 3);

source1$.pipe(
  sequenceEqual(compareTo1$)
).subscribe(
  result => console.log(`Sequences are equal: ${result}`)
);
// Output: "Sequences are equal: true"

// Example 2: Sequences have different values
const source2$ = from([1, 2, 3]);
const compareTo2$ = of(1, 2, 4);

source2$.pipe(
  sequenceEqual(compareTo2$)
).subscribe(
  result => console.log(`Sequences are equal: ${result}`)
);
// Output: "Sequences are equal: false"

// Example 3: Sequences have different lengths
const source3$ = from([1, 2, 3]);
const compareTo3$ = of(1, 2);

source3$.pipe(
  sequenceEqual(compareTo3$)
).subscribe(
  result => console.log(`Sequences are equal: ${result}`)
);
// Output: "Sequences are equal: false"
```

**When to use**:
- For testing if two sequences emit the same values in the same order
- In unit tests to verify Observable behavior
- For detecting changes between two sequences
- When implementing equality checks in reactive applications

## Comparison of Conditional Operators

| Operator | Purpose | Emission Behavior | Use Case |
|----------|---------|-------------------|----------|
| `defaultIfEmpty` | Provide fallback for empty Observables | Emits default value if source is empty | Handling "no data" scenarios |
| `every` | Check if all items satisfy a condition | Emits boolean result when source completes or condition fails | Validation of all items in a sequence |
| `iif` | Choose between Observables based on condition | Subscribes to one of two Observables | Conditional subscription logic |
| `sequenceEqual` | Compare two Observable sequences | Emits boolean result when both sequences complete | Testing sequence equality |

## Practical Examples

### Form Validation with `every`

```typescript
import { combineLatest, from } from 'rxjs';
import { map, every } from 'rxjs/operators';

// Simulate form fields as Observables
const username$ = from(['user123']); // Valid username
const email$ = from(['user@example.com']); // Valid email
const password$ = from(['pass']); // Invalid password (too short)

// Validation functions
const isUsernameValid = (username: string) => username.length >= 5;
const isEmailValid = (email: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
const isPasswordValid = (password: string) => password.length >= 8;

// Check individual field validity
const usernameValid$ = username$.pipe(map(isUsernameValid));
const emailValid$ = email$.pipe(map(isEmailValid));
const passwordValid$ = password$.pipe(map(isPasswordValid));

// Check if all fields are valid
combineLatest([usernameValid$, emailValid$, passwordValid$]).pipe(
  every(isValid => isValid === true)
).subscribe(
  formIsValid => console.log(`Form is valid: ${formIsValid}`)
);
// Output: "Form is valid: false" (because password is too short)
```

### Conditional Data Loading with `iif`

```typescript
import { iif, of, timer } from 'rxjs';
import { mergeMap, map } from 'rxjs/operators';

// Simulate user authentication state
const isAuthenticated$ = of(true); // Change to false to see the alternative flow

// Conditional data loading based on authentication
isAuthenticated$.pipe(
  mergeMap(isAuth => iif(
    () => isAuth,
    // If authenticated, load user data
    timer(1000).pipe(
      map(() => ({ name: 'John Doe', role: 'Admin', lastLogin: new Date() }))
    ),
    // If not authenticated, redirect to login
    of({ redirect: '/login', message: 'Please log in to continue' })
  ))
).subscribe(
  result => console.log('Result:', result)
);
// Output if authenticated:
// "Result: { name: 'John Doe', role: 'Admin', lastLogin: [Date] }"
// Output if not authenticated:
// "Result: { redirect: '/login', message: 'Please log in to continue' }"
```

### Empty State Handling with `defaultIfEmpty`

```typescript
import { from } from 'rxjs';
import { filter, defaultIfEmpty } from 'rxjs/operators';

// Simulate a search function that might return no results
function search(query: string) {
  const database = [
    { id: 1, name: 'JavaScript Book', category: 'programming' },
    { id: 2, name: 'TypeScript Guide', category: 'programming' },
    { id: 3, name: 'RxJS Cookbook', category: 'programming' },
    { id: 4, name: 'Novel', category: 'fiction' }
  ];
  
  return from(database).pipe(
    filter(item => item.name.toLowerCase().includes(query.toLowerCase())),
    defaultIfEmpty({ message: 'No results found for your search.' })
  );
}

// Search with results
search('script').subscribe(
  result => console.log('Search result:', result)
);
// Output:
// "Search result: { id: 2, name: 'TypeScript Guide', category: 'programming' }"

// Search with no results
search('python').subscribe(
  result => console.log('Search result:', result)
);
// Output:
// "Search result: { message: 'No results found for your search.' }"
```

## Conclusion

Conditional operators in RxJS provide powerful tools for controlling the flow of your reactive streams based on specific conditions. By understanding and using these operators effectively, you can create more robust and declarative code:

- Use `defaultIfEmpty` to handle empty Observables gracefully
- Use `every` to validate that all items in a sequence meet a condition
- Use `iif` to choose between different Observable sources based on conditions
- Use `sequenceEqual` to compare two Observable sequences for equality

These operators help you implement conditional logic in a reactive way, making your code more maintainable and easier to reason about.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
