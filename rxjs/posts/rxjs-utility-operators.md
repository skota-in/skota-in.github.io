# RxJS Utility Operators: Examples and Use Cases

*Published on July 5, 2023*

## Introduction

Utility operators in RxJS provide essential tools for debugging, side effects, timing control, and other common tasks that don't fit neatly into transformation or filtering categories. These operators help you manage the flow of your Observable streams without changing the emitted values. This post explores the most commonly used utility operators with practical examples and real-world use cases.

## Table of Contents

1. [tap / do ⭐](#tap--do)
2. [delay](#delay)
3. [delayWhen](#delaywhen)
4. [dematerialize](#dematerialize)
5. [finalize / finally](#finalize--finally)
6. [let](#let)
7. [repeat](#repeat)
8. [Comparison of Key Operators](#comparison-of-key-operators)

## tap / do ⭐

**Purpose**: Performs side effects for each emission without modifying the values in the stream.

```typescript
import { of } from 'rxjs';
import { tap, map } from 'rxjs/operators';

// Example 1: Basic tap for debugging
console.log('Basic tap example:');
of(1, 2, 3, 4, 5).pipe(
  tap(value => console.log(`Before map: ${value}`)),
  map(value => value * 10),
  tap(value => console.log(`After map: ${value}`))
).subscribe(
  value => console.log(`Emitted: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Before map: 1
// After map: 10
// Emitted: 10
// Before map: 2
// After map: 20
// Emitted: 20
// Before map: 3
// After map: 30
// Emitted: 30
// Before map: 4
// After map: 40
// Emitted: 40
// Before map: 5
// After map: 50
// Emitted: 50
// Complete

// Example 2: Using tap for side effects
console.log('Side effects example:');
let sum = 0;
of(1, 2, 3, 4, 5).pipe(
  tap({
    next: value => sum += value,
    complete: () => console.log(`Sum: ${sum}`)
  })
).subscribe();

// Output:
// Sum: 15
```

**When to use**:
- For debugging Observable streams
- When you need to perform side effects without modifying the stream
- For logging or analytics
- To inspect values at different points in an operator chain
- When you need to update external state based on stream values

## delay

**Purpose**: Delays the emissions from the source Observable by a specified amount of time.

```typescript
import { of, timer } from 'rxjs';
import { delay, concatMap, tap } from 'rxjs/operators';

// Example 1: Basic delay
console.log('Basic delay example:');
console.log(`Current time: ${new Date().toLocaleTimeString()}`);
of('Delayed message').pipe(
  delay(2000), // Delay by 2 seconds
  tap(() => console.log(`Emission time: ${new Date().toLocaleTimeString()}`))
).subscribe(
  message => console.log(message),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Current time: 10:15:30 AM
// Emission time: 10:15:32 AM
// Delayed message
// Complete

// Example 2: Delay with Date object
console.log('Delay with Date example:');
const futureDate = new Date(Date.now() + 3000); // 3 seconds in the future
console.log(`Current time: ${new Date().toLocaleTimeString()}`);
console.log(`Target time: ${futureDate.toLocaleTimeString()}`);
of('Future message').pipe(
  delay(futureDate),
  tap(() => console.log(`Emission time: ${new Date().toLocaleTimeString()}`))
).subscribe(
  message => console.log(message),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Current time: 10:15:30 AM
// Target time: 10:15:33 AM
// Emission time: 10:15:33 AM
// Future message
// Complete

// Example 3: Sequential delays
console.log('Sequential delays example:');
of(1, 2, 3).pipe(
  concatMap(value => of(value).pipe(
    delay(1000 * value), // Delay each value by its own value in seconds
    tap(() => console.log(`Emitting ${value} at ${new Date().toLocaleTimeString()}`))
  ))
).subscribe(
  value => console.log(`Processed: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Emitting 1 at 10:15:31 AM
// Processed: 1
// Emitting 2 at 10:15:33 AM
// Processed: 2
// Emitting 3 at 10:15:36 AM
// Processed: 3
// Complete
```

**When to use**:
- When you need to introduce a time delay in your Observable stream
- For implementing timeouts or delays in user interfaces
- When simulating network latency for testing
- For rate limiting or throttling
- When you need to schedule emissions for a future time

## delayWhen

**Purpose**: Delays the emissions from the source Observable by a duration determined by a function that returns an Observable.

```typescript
import { of, timer, interval } from 'rxjs';
import { delayWhen, take, tap } from 'rxjs/operators';

// Example 1: Basic delayWhen
console.log('Basic delayWhen example:');
of(1, 2, 3, 4, 5).pipe(
  tap(value => console.log(`Processing value: ${value} at ${new Date().toLocaleTimeString()}`)),
  // Delay each value by value * 1000 milliseconds
  delayWhen(value => timer(value * 1000)),
  tap(value => console.log(`Emitting value: ${value} at ${new Date().toLocaleTimeString()}`))
).subscribe(
  value => console.log(`Received: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Processing value: 1 at 10:15:30 AM
// Processing value: 2 at 10:15:30 AM
// Processing value: 3 at 10:15:30 AM
// Processing value: 4 at 10:15:30 AM
// Processing value: 5 at 10:15:30 AM
// Emitting value: 1 at 10:15:31 AM
// Received: 1
// Emitting value: 2 at 10:15:32 AM
// Received: 2
// Emitting value: 3 at 10:15:33 AM
// Received: 3
// Emitting value: 4 at 10:15:34 AM
// Received: 4
// Emitting value: 5 at 10:15:35 AM
// Received: 5
// Complete

// Example 2: Dynamic delay based on external factors
console.log('Dynamic delay example:');

// Simulate a service with varying load
const serviceLoad$ = interval(500).pipe(
  take(10),
  map(() => Math.floor(Math.random() * 5) + 1) // Random load between 1-5
);

// Process requests with dynamic delay based on load
of('Request 1', 'Request 2', 'Request 3').pipe(
  concatMap(request => 
    of(request).pipe(
      tap(req => console.log(`Sending ${req} at ${new Date().toLocaleTimeString()}`)),
      // Delay based on current service load
      delayWhen(() => serviceLoad$.pipe(
        take(1),
        tap(load => console.log(`Current service load: ${load}`)),
        // Higher load means longer delay
        switchMap(load => timer(load * 500))
      )),
      tap(req => console.log(`${req} processed at ${new Date().toLocaleTimeString()}`))
    )
  )
).subscribe(
  request => console.log(`Completed: ${request}`),
  err => console.error(err),
  () => console.log('All requests complete')
);

// Output (will vary due to random load):
// Sending Request 1 at 10:15:30 AM
// Current service load: 3
// Request 1 processed at 10:15:31.5 AM
// Completed: Request 1
// Sending Request 2 at 10:15:31.5 AM
// Current service load: 2
// Request 2 processed at 10:15:32.5 AM
// Completed: Request 2
// Sending Request 3 at 10:15:32.5 AM
// Current service load: 4
// Request 3 processed at 10:15:34.5 AM
// Completed: Request 3
// All requests complete
```

**When to use**:
- When you need dynamic delays based on the emitted values
- For implementing adaptive throttling or backoff strategies
- When the delay duration depends on external factors
- For complex scheduling scenarios
- When implementing custom timing logic

## dematerialize

**Purpose**: Converts a stream of Notification objects into the actual values they represent.

```typescript
import { of, Notification } from 'rxjs';
import { dematerialize, materialize } from 'rxjs/operators';

// Example: Using materialize and dematerialize
console.log('Dematerialize example:');

// Create a stream with next, error, and complete notifications
const notifications$ = of(
  Notification.createNext('Value 1'),
  Notification.createNext('Value 2'),
  Notification.createError(new Error('Test error')),
  Notification.createNext('Value 3'), // This won't be emitted due to the error
  Notification.createComplete()
);

// Convert notifications back to actual emissions
notifications$.pipe(
  dematerialize()
).subscribe(
  value => console.log(`Next: ${value}`),
  err => console.error(`Error: ${err.message}`),
  () => console.log('Complete')
);

// Output:
// Next: Value 1
// Next: Value 2
// Error: Test error

// Example 2: Using materialize and dematerialize for error handling
console.log('Error handling with materialize/dematerialize:');

of('Value 1', 'Value 2', 'Value 3').pipe(
  // Map even-length strings to errors
  map(value => {
    if (value.length % 2 === 0) {
      throw new Error(`Even length string: ${value}`);
    }
    return value;
  }),
  // Convert emissions to notifications
  materialize(),
  // Filter out error notifications
  filter(notification => notification.kind !== 'E'),
  // Convert back to normal emissions
  dematerialize()
).subscribe(
  value => console.log(`Next: ${value}`),
  err => console.error(`Error: ${err.message}`),
  () => console.log('Complete')
);

// Output:
// Next: Value 1
// Next: Value 3
// Complete
```

**When to use**:
- When working with Notification objects
- For implementing custom error handling strategies
- When you need to manipulate the notification objects before processing them
- For advanced flow control where you need to handle next, error, and complete notifications uniformly
- When implementing retry or recovery logic

## finalize / finally

**Purpose**: Executes a function when the source Observable completes, errors, or is unsubscribed.

```typescript
import { interval, of, throwError } from 'rxjs';
import { finalize, take, catchError, tap } from 'rxjs/operators';

// Example 1: Basic finalize
console.log('Basic finalize example:');
interval(500).pipe(
  take(3),
  tap(value => console.log(`Emitting: ${value}`)),
  finalize(() => console.log('Finalize called - Resource cleanup'))
).subscribe(
  value => console.log(`Next: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Emitting: 0
// Next: 0
// Emitting: 1
// Next: 1
// Emitting: 2
// Next: 2
// Complete
// Finalize called - Resource cleanup

// Example 2: Finalize with error
console.log('Finalize with error example:');
of(1, 2, 3, 4).pipe(
  tap(value => {
    console.log(`Processing: ${value}`);
    if (value === 3) throw new Error('Error at value 3');
  }),
  catchError(err => {
    console.log(`Caught error: ${err.message}`);
    return throwError(err);
  }),
  finalize(() => console.log('Finalize called even after error'))
).subscribe(
  value => console.log(`Next: ${value}`),
  err => console.error(`Error: ${err.message}`),
  () => console.log('Complete')
);

// Output:
// Processing: 1
// Next: 1
// Processing: 2
// Next: 2
// Processing: 3
// Caught error: Error at value 3
// Error: Error at value 3
// Finalize called even after error

// Example 3: Finalize with unsubscribe
console.log('Finalize with unsubscribe example:');
const subscription = interval(500).pipe(
  tap(value => console.log(`Emitting: ${value}`)),
  finalize(() => console.log('Finalize called due to unsubscribe'))
).subscribe(
  value => console.log(`Next: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Unsubscribe after 2 seconds
setTimeout(() => {
  console.log('Unsubscribing...');
  subscription.unsubscribe();
}, 2000);

// Output:
// Emitting: 0
// Next: 0
// Emitting: 1
// Next: 1
// Emitting: 2
// Next: 2
// Unsubscribing...
// Finalize called due to unsubscribe
```

**When to use**:
- For cleanup operations when an Observable completes or errors
- When you need to release resources regardless of how the Observable terminates
- For logging or analytics at the end of a stream
- When implementing try/finally-like behavior
- For closing connections or stopping services when they're no longer needed

## let

**Purpose**: Allows you to apply a custom operator to the source Observable.

```typescript
import { of, Observable } from 'rxjs';
import { map, filter, tap } from 'rxjs/operators';

// Example: Using let to create a custom operator
console.log('Custom operator with let example:');

// Define a custom operator that filters even numbers and doubles them
function filterEvenAndDouble<T>() {
  return (source: Observable<number>) => {
    return source.pipe(
      filter(value => value % 2 === 0),
      map(value => value * 2),
      tap(value => console.log(`Processed value: ${value}`))
    );
  };
}

// Apply the custom operator using let
of(1, 2, 3, 4, 5, 6).pipe(
  // Note: In RxJS 6+, 'let' is no longer needed as we can directly use pipe
  // This is how it would have been used in earlier versions:
  // let(filterEvenAndDouble())
  
  // In RxJS 6+, we can directly use our custom operator:
  filterEvenAndDouble()
).subscribe(
  value => console.log(`Result: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Processed value: 4
// Result: 4
// Processed value: 8
// Result: 8
// Processed value: 12
// Result: 12
// Complete
```

**When to use**:
- For creating reusable custom operators
- When you need to apply a complex sequence of operators as a single unit
- For improving code organization and reusability
- When implementing domain-specific operators
- For creating higher-order operators

## repeat

**Purpose**: Repeats the source Observable a specified number of times.

```typescript
import { of, interval } from 'rxjs';
import { repeat, take, tap, delay } from 'rxjs/operators';

// Example 1: Basic repeat
console.log('Basic repeat example:');
of('Repeat this').pipe(
  tap(value => console.log(`Processing: ${value}`)),
  repeat(3) // Repeat 3 times
).subscribe(
  value => console.log(`Emitted: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Processing: Repeat this
// Emitted: Repeat this
// Processing: Repeat this
// Emitted: Repeat this
// Processing: Repeat this
// Emitted: Repeat this
// Complete

// Example 2: Repeat with interval
console.log('Repeat with interval example:');
interval(500).pipe(
  take(2),
  tap(value => console.log(`Original value: ${value}`)),
  repeat(3) // Repeat the sequence of [0, 1] three times
).subscribe(
  value => console.log(`Received: ${value}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Original value: 0
// Received: 0
// Original value: 1
// Received: 1
// Original value: 0
// Received: 0
// Original value: 1
// Received: 1
// Original value: 0
// Received: 0
// Original value: 1
// Received: 1
// Complete

// Example 3: Polling with repeat
console.log('Polling example:');

// Simulate an API call
function fetchData() {
  console.log('Fetching data...');
  return of({ data: 'API Response', timestamp: new Date().toLocaleTimeString() }).pipe(
    delay(1000) // Simulate network delay
  );
}

// Poll the API every 3 seconds
fetchData().pipe(
  tap(response => console.log(`Received response: ${JSON.stringify(response)}`)),
  delay(2000), // Wait 2 seconds after each response
  repeat(3) // Poll 3 times
).subscribe(
  response => console.log(`Processed: ${response.data} at ${response.timestamp}`),
  err => console.error(err),
  () => console.log('Polling complete')
);

// Output:
// Fetching data...
// Received response: {"data":"API Response","timestamp":"10:15:31 AM"}
// Processed: API Response at 10:15:31 AM
// Fetching data...
// Received response: {"data":"API Response","timestamp":"10:15:34 AM"}
// Processed: API Response at 10:15:34 AM
// Fetching data...
// Received response: {"data":"API Response","timestamp":"10:15:37 AM"}
// Processed: API Response at 10:15:37 AM
// Polling complete
```

**When to use**:
- When you need to repeat an operation multiple times
- For implementing polling or retry logic
- When you want to resubscribe to a source after it completes
- For creating recurring tasks or jobs
- When implementing animations or sequences that need to repeat

## Comparison of Key Operators

| Operator | Purpose | Affects Values | Timing Control | Use Case |
|----------|---------|---------------|---------------|----------|
| `tap` / `do` ⭐ | Side effects | No | No | Debugging, logging, external state updates |
| `delay` | Fixed time delay | No | Yes | Simple timing delays, throttling |
| `delayWhen` | Dynamic time delay | No | Yes | Adaptive delays, backoff strategies |
| `dematerialize` | Convert notifications to values | Yes | No | Custom error handling, notification processing |
| `finalize` / `finally` | Cleanup | No | No | Resource cleanup, logging completion |
| `let` | Custom operators | Depends | Depends | Creating reusable operator chains |
| `repeat` | Resubscription | No | No | Polling, retries, recurring operations |

## Practical Examples

### Logging Service with tap

```typescript
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

// Simple logging service
class LoggingService {
  log(message: string, level: 'info' | 'warn' | 'error' = 'info') {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] [${level.toUpperCase()}] ${message}`);
  }
}

// Create a logging operator
function withLogging<T>(
  loggingService: LoggingService,
  message: string | ((value: T) => string),
  level: 'info' | 'warn' | 'error' = 'info'
) {
  return (source: Observable<T>) => {
    return source.pipe(
      tap({
        next: (value) => {
          const logMessage = typeof message === 'function' 
            ? message(value) 
            : message;
          loggingService.log(logMessage, level);
        },
        error: (err) => {
          loggingService.log(`Error: ${err.message}`, 'error');
        },
        complete: () => {
          loggingService.log('Observable completed', 'info');
        }
      })
    );
  };
}

// Usage
const logger = new LoggingService();

fetchUserData(userId).pipe(
  withLogging(logger, (user) => `Fetched user: ${user.name}`, 'info'),
  map(user => transformUserData(user)),
  withLogging(logger, 'User data transformed', 'info'),
  catchError(err => {
    // Error already logged by the withLogging operator
    return of(defaultUser);
  })
).subscribe();
```

### Retry with Exponential Backoff using delayWhen and repeat

```typescript
import { Observable, throwError, timer, of } from 'rxjs';
import { mergeMap, tap, delayWhen, repeat, scan, take } from 'rxjs/operators';

// Create a retry with exponential backoff operator
function retryWithBackoff<T>(
  maxRetries = 3,
  initialDelay = 1000,
  backoffFactor = 2
) {
  return (source: Observable<T>) => {
    return source.pipe(
      // Track retry attempts
      scan((attempts, value) => {
        return { attempts: attempts.attempts + 1, value };
      }, { attempts: 0, value: null as any }),
      
      // Handle retry logic
      mergeMap(({ attempts, value }) => {
        // If we've reached max retries, throw error
        if (attempts > maxRetries) {
          return throwError(new Error(`Retry limit exceeded (${maxRetries})`));
        }
        
        // Calculate delay based on attempt number
        const delay = attempts === 1 ? 0 : initialDelay * Math.pow(backoffFactor, attempts - 2);
        
        // Log retry information
        if (attempts > 1) {
          console.log(`Retry attempt ${attempts - 1} after ${delay}ms`);
        }
        
        // Delay based on retry count
        return of(value).pipe(
          delayWhen(() => timer(delay))
        );
      })
    );
  };
}

// Usage example
function fetchDataWithRetry() {
  let attempts = 0;
  
  return new Observable(subscriber => {
    attempts++;
    console.log(`Making API request (attempt ${attempts})...`);
    
    // Simulate API that fails the first 2 times
    if (attempts <= 2) {
      subscriber.error(new Error(`API error on attempt ${attempts}`));
    } else {
      subscriber.next({ data: 'Success!', timestamp: new Date() });
      subscriber.complete();
    }
  }).pipe(
    retryWithBackoff(3, 1000, 2),
    tap({
      next: response => console.log(`Request succeeded: ${JSON.stringify(response)}`),
      error: err => console.error(`Request failed: ${err.message}`)
    })
  );
}

// Execute the request
fetchDataWithRetry().subscribe(
  result => console.log(`Received result: ${result.data}`),
  err => console.error(`Error: ${err.message}`),
  () => console.log('Operation complete')
);

// Output:
// Making API request (attempt 1)...
// Making API request (attempt 2)...
// Retry attempt 1 after 1000ms
// Making API request (attempt 3)...
// Request succeeded: {"data":"Success!","timestamp":"2023-07-05T10:15:32.456Z"}
// Received result: Success!
// Operation complete
```

## Conclusion

Utility operators in RxJS provide essential tools for managing Observable streams without necessarily changing the emitted values. They help with debugging, timing control, and other common tasks that are crucial for building robust reactive applications:

- Use `tap` / `do` ⭐ for debugging and side effects without affecting the stream
- Use `delay` for introducing fixed time delays
- Use `delayWhen` for dynamic delays based on values or external factors
- Use `dematerialize` when working with Notification objects
- Use `finalize` / `finally` for cleanup operations regardless of how the stream terminates
- Use `let` for creating custom, reusable operators
- Use `repeat` for resubscribing to a source multiple times

By mastering these utility operators, you can build more robust, maintainable, and efficient reactive applications.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
- [RxJS Utility Operators](https://www.learnrxjs.io/learn-rxjs/operators/utility)
