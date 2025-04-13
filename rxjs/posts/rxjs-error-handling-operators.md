# RxJS Error Handling Operators: Examples and Use Cases

*Published on August 25, 2023*

## Introduction

Error handling is a critical aspect of reactive programming. RxJS provides specialized operators to handle errors in Observable streams gracefully, allowing you to recover from failures, retry operations, and provide fallback values. This post explores the error handling operators in RxJS with practical examples and real-world use cases.

## Table of Contents

1. [catch / catchError ⭐](#catch--catcherror)
2. [retry](#retry)
3. [retryWhen](#retrywhen)
4. [Comparison of Error Handling Operators](#comparison-of-error-handling-operators)
5. [Advanced Error Handling Patterns](#advanced-error-handling-patterns)

## catch / catchError ⭐

**Purpose**: Catches errors on the source Observable, providing an opportunity to recover by returning a new Observable or throwing an error.

```typescript
import { of, throwError } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

// Example 1: Basic error catching
const source$ = throwError(() => new Error('Something went wrong!'));

source$.pipe(
  catchError(error => {
    console.log(`Error caught: ${error.message}`);
    // Return a new Observable as a fallback
    return of('Fallback value');
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output:
// "Error caught: Something went wrong!"
// "Next: Fallback value"
// "Completed"

// Example 2: Error handling in a chain
of(1, 2, 3, 4, 5).pipe(
  map(n => {
    if (n === 4) {
      throw new Error('Four is not allowed!');
    }
    return n * 10;
  }),
  catchError(error => {
    console.log(`Error in map: ${error.message}`);
    // Return a safe Observable
    return of(-1);
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output:
// "Next: 10"
// "Next: 20"
// "Next: 30"
// "Error in map: Four is not allowed!"
// "Next: -1"
// "Completed"

// Example 3: Rethrowing errors
of(1, 2, 3).pipe(
  map(n => {
    if (n === 2) {
      throw new Error('Value is 2!');
    }
    return n;
  }),
  catchError(error => {
    console.log(`Caught error: ${error.message}`);
    // Decide whether to recover or rethrow
    if (error.message === 'Value is 2!') {
      // Rethrow the error
      return throwError(() => new Error('Rethrown: ' + error.message));
    }
    return of(0); // Fallback for other errors
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output:
// "Next: 1"
// "Caught error: Value is 2!"
// "Error: Rethrown: Value is 2!"
```

**When to use**:
- For providing fallback values when errors occur
- To prevent errors from propagating and breaking the Observable chain
- When you need to transform errors into different types
- For implementing graceful error recovery strategies

## retry

**Purpose**: Resubscribes to the source Observable a specified number of times when an error occurs, in an attempt to recover from the error.

```typescript
import { of, throwError, interval } from 'rxjs';
import { mergeMap, retry, map, take } from 'rxjs/operators';

// Example 1: Basic retry
console.log('Basic retry example:');
throwError(() => new Error('Initial error')).pipe(
  retry(2), // Retry 2 times (3 total attempts)
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of('Fallback after retries');
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output:
// "Basic retry example:"
// "All retries failed: Initial error"
// "Next: Fallback after retries"
// "Completed"

// Example 2: Retry with a flaky Observable
console.log('Flaky Observable example:');
let attemptCount = 0;

interval(1000).pipe(
  take(5),
  map(val => {
    attemptCount++;
    if (val > 0 && val % 2 === 0) {
      console.log(`Attempt ${attemptCount}: Generating error at value ${val}`);
      throw new Error(`Error at value ${val}`);
    }
    console.log(`Attempt ${attemptCount}: Emitting value ${val}`);
    return val;
  }),
  retry(2),
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of(-1);
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output (approximate):
// "Flaky Observable example:"
// "Attempt 1: Emitting value 0"
// "Next: 0"
// "Attempt 2: Emitting value 1"
// "Next: 1"
// "Attempt 3: Generating error at value 2"
// "Attempt 4: Emitting value 0"
// "Next: 0"
// "Attempt 5: Emitting value 1"
// "Next: 1"
// "Attempt 6: Generating error at value 2"
// "Attempt 7: Emitting value 0"
// "Next: 0"
// "Attempt 8: Emitting value 1"
// "Next: 1"
// "Attempt 9: Generating error at value 2"
// "All retries failed: Error at value 2"
// "Next: -1"
// "Completed"

// Example 3: Simulating HTTP requests with retry
console.log('HTTP request simulation:');

// Simulate an API call that sometimes fails
function simulateHttpRequest(shouldSucceed: boolean) {
  return of({ success: shouldSucceed }).pipe(
    mergeMap(result => {
      if (result.success) {
        return of({ data: 'Response data', status: 200 });
      } else {
        return throwError(() => new Error('Server error 500'));
      }
    })
  );
}

// First a failing request
simulateHttpRequest(false).pipe(
  retry(2),
  catchError(error => {
    console.log(`Request failed after retries: ${error.message}`);
    return of({ data: null, status: 500, error: error.message });
  })
).subscribe(response => {
  console.log('Response:', response);
});

// Then a successful request
simulateHttpRequest(true).pipe(
  retry(2),
  catchError(error => {
    console.log(`Request failed after retries: ${error.message}`);
    return of({ data: null, status: 500, error: error.message });
  })
).subscribe(response => {
  console.log('Response:', response);
});

// Output:
// "HTTP request simulation:"
// "Request failed after retries: Server error 500"
// "Response: {data: null, status: 500, error: 'Server error 500'}"
// "Response: {data: 'Response data', status: 200}"
```

**When to use**:
- For transient errors that might resolve on retry (network issues, race conditions)
- When implementing resilient HTTP requests
- For operations that might fail due to timing issues
- When you want simple retry logic without complex conditions

## retryWhen

**Purpose**: Provides fine-grained control over the retry logic, allowing you to implement advanced retry strategies like exponential backoff.

```typescript
import { of, throwError, timer, interval } from 'rxjs';
import { retryWhen, mergeMap, tap, delayWhen, scan, take } from 'rxjs/operators';

// Example 1: Basic retryWhen with delay
console.log('Basic retryWhen example:');
throwError(() => new Error('Transient error')).pipe(
  retryWhen(errors => 
    errors.pipe(
      tap(error => console.log(`Error occurred: ${error.message}`)),
      delayWhen(() => timer(1000)), // Wait 1 second before retrying
      take(3) // Limit to 3 retries
    )
  ),
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of('Fallback after retries');
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output:
// "Basic retryWhen example:"
// "Error occurred: Transient error"
// (wait 1 second)
// "Error occurred: Transient error"
// (wait 1 second)
// "Error occurred: Transient error"
// (wait 1 second)
// "All retries failed: Transient error"
// "Next: Fallback after retries"
// "Completed"

// Example 2: Exponential backoff retry strategy
console.log('Exponential backoff example:');
let attemptCount = 0;

interval(500).pipe(
  take(5),
  map(val => {
    attemptCount++;
    if (val > 0 && val % 2 === 0) {
      console.log(`Attempt ${attemptCount}: Generating error at value ${val}`);
      throw new Error(`Error at value ${val}`);
    }
    console.log(`Attempt ${attemptCount}: Emitting value ${val}`);
    return val;
  }),
  retryWhen(errors => 
    errors.pipe(
      scan((attempts, error) => {
        attempts += 1;
        if (attempts > 3) {
          throw error; // Give up after 3 retries
        }
        console.log(`Retry attempt ${attempts} after ${Math.pow(2, attempts)} seconds`);
        return attempts;
      }, 0),
      delayWhen(attempts => timer(Math.pow(2, attempts) * 1000))
    )
  ),
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of(-1);
  })
).subscribe({
  next: value => console.log(`Next: ${value}`),
  error: err => console.error(`Error: ${err.message}`),
  complete: () => console.log('Completed')
});

// Output (approximate):
// "Exponential backoff example:"
// "Attempt 1: Emitting value 0"
// "Next: 0"
// "Attempt 2: Emitting value 1"
// "Next: 1"
// "Attempt 3: Generating error at value 2"
// "Retry attempt 1 after 2 seconds"
// (wait 2 seconds)
// "Attempt 4: Emitting value 0"
// "Next: 0"
// "Attempt 5: Emitting value 1"
// "Next: 1"
// "Attempt 6: Generating error at value 2"
// "Retry attempt 2 after 4 seconds"
// (wait 4 seconds)
// "Attempt 7: Emitting value 0"
// "Next: 0"
// "Attempt 8: Emitting value 1"
// "Next: 1"
// "Attempt 9: Generating error at value 2"
// "Retry attempt 3 after 8 seconds"
// (wait 8 seconds)
// "Attempt 10: Emitting value 0"
// "Next: 0"
// "Attempt 11: Emitting value 1"
// "Next: 1"
// "Attempt 12: Generating error at value 2"
// "All retries failed: Error at value 2"
// "Next: -1"
// "Completed"

// Example 3: Conditional retry based on error type
console.log('Conditional retry example:');

function simulateHttpRequest(errorType: string | null) {
  return of(errorType).pipe(
    mergeMap(type => {
      if (!type) {
        return of({ data: 'Success response', status: 200 });
      } else if (type === 'timeout') {
        return throwError(() => new Error('Request timed out'));
      } else if (type === 'server') {
        return throwError(() => new Error('Internal server error'));
      } else {
        return throwError(() => new Error('Unknown error'));
      }
    })
  );
}

// Function to implement conditional retry logic
function conditionalRetry() {
  return retryWhen(errors => 
    errors.pipe(
      mergeMap((error, index) => {
        const retryAttempt = index + 1;
        
        // Only retry timeout errors, give up on server errors
        if (error.message.includes('timed out') && retryAttempt <= 3) {
          console.log(`Retrying timeout error, attempt ${retryAttempt} after ${retryAttempt} second(s)`);
          return timer(retryAttempt * 1000);
        }
        
        // For other errors, don't retry
        console.log(`Not retrying error: ${error.message}`);
        return throwError(() => error);
      })
    )
  );
}

// Test with different error types
simulateHttpRequest('timeout').pipe(
  conditionalRetry(),
  catchError(error => {
    console.log(`Final error handler: ${error.message}`);
    return of({ data: null, status: 408, error: error.message });
  })
).subscribe(response => {
  console.log('Timeout response:', response);
});

simulateHttpRequest('server').pipe(
  conditionalRetry(),
  catchError(error => {
    console.log(`Final error handler: ${error.message}`);
    return of({ data: null, status: 500, error: error.message });
  })
).subscribe(response => {
  console.log('Server error response:', response);
});

simulateHttpRequest(null).pipe(
  conditionalRetry(),
  catchError(error => {
    console.log(`Final error handler: ${error.message}`);
    return of({ data: null, status: 500, error: error.message });
  })
).subscribe(response => {
  console.log('Success response:', response);
});

// Output:
// "Conditional retry example:"
// "Not retrying error: Request timed out"
// "Final error handler: Request timed out"
// "Timeout response: {data: null, status: 408, error: 'Request timed out'}"
// "Not retrying error: Internal server error"
// "Final error handler: Internal server error"
// "Server error response: {data: null, status: 500, error: 'Internal server error'}"
// "Success response: {data: 'Success response', status: 200}"
```

**When to use**:
- When you need advanced retry strategies like exponential backoff
- For implementing conditional retry logic based on error types
- When you want to add delays between retry attempts
- For complex error recovery scenarios with custom logic

## Comparison of Error Handling Operators

| Operator | Purpose | Behavior | Use Case |
|----------|---------|----------|----------|
| `catchError` | Handle errors and provide fallbacks | Intercepts errors and returns a new Observable | General error handling, fallbacks |
| `retry` | Resubscribe to source on error | Retries a fixed number of times immediately | Simple retry for transient errors |
| `retryWhen` | Custom retry logic | Provides fine-grained control over retry behavior | Advanced retry strategies (backoff, conditional) |

## Advanced Error Handling Patterns

### Pattern 1: HTTP Request with Exponential Backoff

```typescript
import { of, throwError, timer } from 'rxjs';
import { mergeMap, retryWhen, scan, delayWhen, catchError } from 'rxjs/operators';

// Simulate an HTTP service
class HttpService {
  get(url: string, simulateError = false) {
    return of({ url }).pipe(
      mergeMap(req => {
        if (simulateError) {
          return throwError(() => new Error('Network error'));
        }
        return of({ data: 'Response data', status: 200 });
      })
    );
  }
}

// Create a reusable exponential backoff operator
function exponentialBackoff(maxRetries = 3, initialDelay = 1000) {
  return retryWhen(errors => 
    errors.pipe(
      scan((attempts, error) => {
        attempts += 1;
        if (attempts > maxRetries) {
          throw error;
        }
        console.log(`Attempt ${attempts}: Retrying after ${initialDelay * Math.pow(2, attempts - 1)}ms`);
        return attempts;
      }, 0),
      delayWhen(attempts => timer(initialDelay * Math.pow(2, attempts - 1)))
    )
  );
}

// Usage
const http = new HttpService();

console.log('HTTP request with exponential backoff:');
http.get('/api/data', true).pipe(
  exponentialBackoff(4, 500),
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of({ data: null, status: 0, error: error.message });
  })
).subscribe({
  next: response => console.log('Response:', response),
  complete: () => console.log('Request completed')
});

// Output:
// "HTTP request with exponential backoff:"
// "Attempt 1: Retrying after 500ms"
// (wait 500ms)
// "Attempt 2: Retrying after 1000ms"
// (wait 1000ms)
// "Attempt 3: Retrying after 2000ms"
// (wait 2000ms)
// "Attempt 4: Retrying after 4000ms"
// (wait 4000ms)
// "All retries failed: Network error"
// "Response: {data: null, status: 0, error: 'Network error'}"
// "Request completed"
```

### Pattern 2: Error Boundary with Isolation

```typescript
import { Observable, of, throwError } from 'rxjs';
import { catchError, finalize } from 'rxjs/operators';

// Create an error boundary operator
function errorBoundary<T>(fallback: T, errorCallback?: (error: any) => void) {
  return (source: Observable<T>) => 
    source.pipe(
      catchError(error => {
        if (errorCallback) {
          errorCallback(error);
        }
        console.error(`Error caught by boundary: ${error.message}`);
        return of(fallback);
      }),
      finalize(() => console.log('Error boundary finalized'))
    );
}

// Usage
console.log('Error boundary pattern:');

// Component 1 - will error
const component1$ = of(1, 2, 3).pipe(
  map(n => {
    if (n === 2) throw new Error('Component 1 failed');
    return `Component 1: ${n}`;
  })
);

// Component 2 - will succeed
const component2$ = of(4, 5, 6).pipe(
  map(n => `Component 2: ${n}`)
);

// Apply error boundaries to isolate failures
component1$.pipe(
  errorBoundary('Component 1 fallback', err => {
    // Log error or send to monitoring service
    console.log(`Logging error from Component 1: ${err.message}`);
  })
).subscribe(result => console.log(result));

component2$.pipe(
  errorBoundary('Component 2 fallback')
).subscribe(result => console.log(result));

// Output:
// "Error boundary pattern:"
// "Component 1: 1"
// "Error caught by boundary: Component 1 failed"
// "Logging error from Component 1: Component 1 failed"
// "Component 1 fallback"
// "Error boundary finalized"
// "Component 2: 4"
// "Component 2: 5"
// "Component 2: 6"
// "Error boundary finalized"
```

### Pattern 3: Retry with Progressive Delays

```typescript
import { timer, throwError, of } from 'rxjs';
import { mergeMap, retryWhen, scan, tap, delayWhen, catchError } from 'rxjs/operators';

// Create a progressive delay retry strategy
function progressiveRetry(
  maxRetries = 3,
  retryDelays = [1000, 2000, 5000]
) {
  return retryWhen(errors => 
    errors.pipe(
      scan((attempts, error) => {
        attempts += 1;
        if (attempts > maxRetries) {
          throw error;
        }
        return attempts;
      }, 0),
      tap(attempts => {
        const delay = retryDelays[attempts - 1] || retryDelays[retryDelays.length - 1];
        console.log(`Retry attempt ${attempts} after ${delay}ms`);
      }),
      delayWhen(attempts => {
        const delay = retryDelays[attempts - 1] || retryDelays[retryDelays.length - 1];
        return timer(delay);
      })
    )
  );
}

// Simulate a flaky API
function flakyApi(shouldSucceedOnAttempt = 0) {
  let attempts = 0;
  
  return of(null).pipe(
    mergeMap(() => {
      attempts += 1;
      console.log(`API attempt ${attempts}`);
      
      if (attempts === shouldSucceedOnAttempt || shouldSucceedOnAttempt === 0) {
        return of({ success: true, data: 'API response', attempt: attempts });
      }
      
      return throwError(() => new Error(`API failed on attempt ${attempts}`));
    })
  );
}

// Usage
console.log('Progressive retry pattern:');

// Will succeed on the 3rd attempt
flakyApi(3).pipe(
  progressiveRetry(3, [500, 1000, 3000]),
  catchError(error => {
    console.log(`All retries failed: ${error.message}`);
    return of({ success: false, error: error.message });
  })
).subscribe({
  next: result => console.log('Final result:', result),
  complete: () => console.log('Operation completed')
});

// Output:
// "Progressive retry pattern:"
// "API attempt 1"
// "Retry attempt 1 after 500ms"
// (wait 500ms)
// "API attempt 2"
// "Retry attempt 2 after 1000ms"
// (wait 1000ms)
// "API attempt 3"
// "Final result: {success: true, data: 'API response', attempt: 3}"
// "Operation completed"
```

## Conclusion

Error handling is a crucial aspect of building robust reactive applications. RxJS provides powerful operators to handle errors gracefully, allowing your applications to recover from failures and provide a smooth user experience.

Key takeaways:
- Use `catchError` for general error handling and providing fallbacks
- Use `retry` for simple retry scenarios with transient errors
- Use `retryWhen` for advanced retry strategies like exponential backoff
- Combine these operators to create sophisticated error handling patterns

By mastering these error handling operators, you can build more resilient applications that gracefully handle failures and provide a better user experience.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
