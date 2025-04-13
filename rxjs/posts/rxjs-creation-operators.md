# RxJS Creation Operators: Examples and Use Cases

*Published on August 20, 2023*

## Introduction

Creation operators in RxJS are the starting point of reactive programming. They create Observables from various sources like events, promises, arrays, or even from scratch. These operators provide the foundation for building reactive applications by converting different types of inputs into Observable streams. This post explores the most commonly used creation operators with practical examples.

## Table of Contents

1. [ajax ⭐](#ajax)
2. [create](#create)
3. [defer](#defer)
4. [empty](#empty)
5. [from ⭐](#from)
6. [fromEvent](#fromevent)
7. [generate](#generate)
8. [interval](#interval)
9. [of ⭐](#of)
10. [range](#range)
11. [throw](#throw)
12. [timer](#timer)
13. [Comparison of Creation Operators](#comparison-of-creation-operators)

## ajax ⭐

**Purpose**: Creates an Observable for an Ajax request with either a request object or a string URL.

```typescript
import { ajax } from 'rxjs/ajax';
import { map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

// Example 1: Basic GET request
ajax('https://api.github.com/users').pipe(
  map(response => response.response),
  catchError(error => {
    console.error('Error:', error);
    return of([]);
  })
).subscribe(
  users => console.log('Users:', users)
);
// Output: "Users: [...]" (array of GitHub users)

// Example 2: POST request with custom headers
ajax({
  url: 'https://api.example.com/data',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  },
  body: {
    name: 'John Doe',
    email: 'john@example.com'
  }
}).pipe(
  map(response => response.response),
  catchError(error => {
    console.error('Error:', error);
    return of({ error: true, message: 'Failed to submit data' });
  })
).subscribe(
  result => console.log('Result:', result)
);
// Output depends on the API response
```

**When to use**:
- For making HTTP requests in a reactive way
- When you need to handle responses as Observable streams
- For integrating with REST APIs
- When you need advanced features like request cancellation

## create

**Purpose**: Creates an Observable from scratch by explicitly calling `next()`, `error()`, and `complete()` methods.

```typescript
import { Observable } from 'rxjs';

// Example: Creating a custom Observable
const customObservable$ = new Observable(subscriber => {
  console.log('Observable execution started');
  
  // Emit values
  subscriber.next('First value');
  
  // Simulate async operation
  setTimeout(() => {
    subscriber.next('Async value after 1 second');
    
    // Complete the Observable
    subscriber.complete();
  }, 1000);
  
  // Cleanup function (called on unsubscribe or complete)
  return () => {
    console.log('Cleanup: Observable execution ended');
  };
});

// Subscribe to the Observable
const subscription = customObservable$.subscribe({
  next: value => console.log(`Received: ${value}`),
  error: err => console.error(`Error: ${err}`),
  complete: () => console.log('Completed')
});

// Later: Unsubscribe (this won't be reached in this example because the Observable completes first)
// setTimeout(() => subscription.unsubscribe(), 2000);

// Output:
// "Observable execution started"
// "Received: First value"
// "Received: Async value after 1 second"
// "Completed"
// "Cleanup: Observable execution ended"
```

**When to use**:
- When you need complete control over the Observable lifecycle
- For wrapping non-RxJS async APIs
- When implementing custom Observable behavior
- For educational purposes to understand how Observables work

## defer

**Purpose**: Creates an Observable factory that creates a new Observable for each subscriber, allowing for dynamic Observable creation.

```typescript
import { defer, of } from 'rxjs';

// Example 1: Basic defer usage
const deferredObservable$ = defer(() => {
  // This code runs when someone subscribes
  console.log('Creating new Observable');
  const random = Math.random();
  
  // Return a different Observable based on the random value
  return of(`Random value: ${random}`);
});

// First subscription
console.log('First subscription:');
deferredObservable$.subscribe(
  value => console.log(`Subscriber 1: ${value}`)
);

// Second subscription (gets a different random value)
console.log('Second subscription:');
deferredObservable$.subscribe(
  value => console.log(`Subscriber 2: ${value}`)
);

// Output:
// "First subscription:"
// "Creating new Observable"
// "Subscriber 1: Random value: 0.123456789" (random number)
// "Second subscription:"
// "Creating new Observable"
// "Subscriber 2: Random value: 0.987654321" (different random number)

// Example 2: Using defer with conditional logic
function getDataObservable(forceRefresh: boolean) {
  return defer(() => {
    if (forceRefresh) {
      console.log('Fetching fresh data from server...');
      return of('Fresh data from server');
    } else {
      console.log('Getting data from cache...');
      return of('Cached data');
    }
  });
}

// With cache
getDataObservable(false).subscribe(
  data => console.log(`Received: ${data}`)
);
// Output:
// "Getting data from cache..."
// "Received: Cached data"

// With force refresh
getDataObservable(true).subscribe(
  data => console.log(`Received: ${data}`)
);
// Output:
// "Fetching fresh data from server..."
// "Received: Fresh data from server"
```

**When to use**:
- When you need to create a new Observable for each subscriber
- For implementing lazy evaluation of Observables
- When the Observable creation depends on subscription-time conditions
- For implementing retry logic with different parameters

## empty

**Purpose**: Creates an Observable that emits no items and immediately completes.

```typescript
import { empty, EMPTY } from 'rxjs';

// Example: Using EMPTY (preferred way)
console.log('Subscribing to EMPTY:');
EMPTY.subscribe({
  next: value => console.log(`Value: ${value}`),
  complete: () => console.log('Completed')
});

// Output:
// "Subscribing to EMPTY:"
// "Completed"
```

**When to use**:
- As a default or fallback Observable
- When you need an Observable that just completes
- In conditional operators like `defaultIfEmpty`
- For representing "no operation" in Observable chains

## from ⭐

**Purpose**: Creates an Observable from various data sources like arrays, iterables, Promises, or other Observable-like objects.

```typescript
import { from } from 'rxjs';

// Example 1: From an array
console.log('From array:');
from([1, 2, 3, 4, 5]).subscribe(
  value => console.log(`Value: ${value}`)
);
// Output:
// "From array:"
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"
// "Value: 5"

// Example 2: From a Promise
console.log('From Promise:');
from(Promise.resolve('Resolved value')).subscribe(
  value => console.log(`Value: ${value}`)
);
// Output:
// "From Promise:"
// "Value: Resolved value"

// Example 3: From a string (iterable)
console.log('From string:');
from('Hello').subscribe(
  value => console.log(`Value: ${value}`)
);
// Output:
// "From string:"
// "Value: H"
// "Value: e"
// "Value: l"
// "Value: l"
// "Value: o"

// Example 4: From a Map
console.log('From Map:');
const map = new Map([
  ['name', 'John'],
  ['age', 30],
  ['city', 'New York']
]);
from(map).subscribe(
  entry => console.log(`Key: ${entry[0]}, Value: ${entry[1]}`)
);
// Output:
// "From Map:"
// "Key: name, Value: John"
// "Key: age, Value: 30"
// "Key: city, Value: New York"
```

**When to use**:
- To convert arrays, iterables, or Promises to Observables
- When working with async/await and you need to convert Promises to Observables
- For iterating over collections reactively
- When integrating RxJS with non-RxJS code that returns Promises or arrays

## fromEvent

**Purpose**: Creates an Observable from DOM events or Node.js EventEmitter events.

```typescript
import { fromEvent } from 'rxjs';
import { map, throttleTime } from 'rxjs/operators';

// Example 1: Mouse clicks
const clicks$ = fromEvent(document, 'click');

clicks$.pipe(
  map((event: MouseEvent) => ({
    x: event.clientX,
    y: event.clientY
  })),
  throttleTime(1000) // Limit to one event per second
).subscribe(
  position => console.log('Click position:', position)
);
// Output when clicking:
// "Click position: {x: 123, y: 456}"

// Example 2: Input changes
const input = document.getElementById('search-input');
if (input) {
  const input$ = fromEvent(input, 'input');
  
  input$.pipe(
    map((event: any) => event.target.value)
  ).subscribe(
    value => console.log('Search term:', value)
  );
  // Output when typing:
  // "Search term: t"
  // "Search term: ty"
  // "Search term: typ"
  // "Search term: type"
}

// Example 3: Multiple events
const mouseEvents$ = fromEvent(document, 'mousemove').pipe(
  throttleTime(300)
);

const subscription = mouseEvents$.subscribe(
  event => console.log('Mouse moved')
);

// Later: Unsubscribe to stop listening
// setTimeout(() => subscription.unsubscribe(), 5000);
```

**When to use**:
- For handling DOM events reactively
- When building interactive UI components
- For implementing event-driven features
- When you need to apply RxJS operators to event streams

## generate

**Purpose**: Creates an Observable that generates a sequence of values based on a loop-like state machine.

```typescript
import { generate } from 'rxjs';

// Example 1: Generate a sequence of numbers (like a for loop)
generate(
  0,                    // Initial state
  value => value < 5,   // Condition
  value => value + 1,   // Iterator
  value => `Value: ${value}`  // Result selector
).subscribe(
  result => console.log(result)
);
// Output:
// "Value: 0"
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"

// Example 2: Generate Fibonacci sequence
generate(
  { first: 0, second: 1, index: 0 },  // Initial state
  state => state.index < 10,          // Condition
  state => ({                         // Iterator
    first: state.second,
    second: state.first + state.second,
    index: state.index + 1
  }),
  state => state.first                // Result selector
).subscribe(
  fibonacci => console.log(`Fibonacci ${fibonacci}`)
);
// Output:
// "Fibonacci 0"
// "Fibonacci 1"
// "Fibonacci 1"
// "Fibonacci 2"
// "Fibonacci 3"
// "Fibonacci 5"
// "Fibonacci 8"
// "Fibonacci 13"
// "Fibonacci 21"
// "Fibonacci 34"
```

**When to use**:
- When you need to generate values based on a loop-like state machine
- For creating sequences with complex generation rules
- When you need more control than simple range or interval operators
- For implementing custom iteration logic

## interval

**Purpose**: Creates an Observable that emits sequential numbers at specified time intervals.

```typescript
import { interval } from 'rxjs';
import { take, map } from 'rxjs/operators';

// Example 1: Basic interval
console.log('Starting basic interval:');
const counter$ = interval(1000); // Emit every 1 second

const subscription = counter$.pipe(
  take(5) // Take only 5 values then complete
).subscribe(
  value => console.log(`Counter: ${value}`)
);
// Output (one per second):
// "Starting basic interval:"
// "Counter: 0"
// "Counter: 1"
// "Counter: 2"
// "Counter: 3"
// "Counter: 4"

// Example 2: Creating a countdown timer
console.log('Starting countdown:');
interval(1000).pipe(
  map(value => 10 - value),
  take(11)
).subscribe(
  value => {
    if (value > 0) {
      console.log(`Countdown: ${value}`);
    } else {
      console.log('Blast off! 🚀');
    }
  }
);
// Output (one per second):
// "Starting countdown:"
// "Countdown: 10"
// "Countdown: 9"
// ...
// "Countdown: 1"
// "Blast off! 🚀"
```

**When to use**:
- For creating timers or counters
- When implementing polling mechanisms
- For animations or transitions with regular intervals
- When you need to perform an action repeatedly at fixed intervals

## of ⭐

**Purpose**: Creates an Observable that emits the specified arguments and then completes.

```typescript
import { of } from 'rxjs';

// Example 1: Basic usage with multiple values
console.log('Basic of() example:');
of(1, 2, 3, 4, 5).subscribe(
  value => console.log(`Value: ${value}`),
  null,
  () => console.log('Completed')
);
// Output:
// "Basic of() example:"
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"
// "Value: 5"
// "Completed"

// Example 2: With different types of values
console.log('Mixed types example:');
of(
  'Hello',
  42,
  true,
  { name: 'John', age: 30 },
  [1, 2, 3]
).subscribe(
  value => console.log(`Value:`, value),
  null,
  () => console.log('Completed')
);
// Output:
// "Mixed types example:"
// "Value: Hello"
// "Value: 42"
// "Value: true"
// "Value: {name: 'John', age: 30}"
// "Value: [1, 2, 3]"
// "Completed"

// Example 3: Creating an Observable from a single object
console.log('Single object example:');
const user = {
  id: 1,
  name: 'Alice',
  email: 'alice@example.com'
};

of(user).subscribe(
  userData => console.log('User data:', userData),
  null,
  () => console.log('Completed')
);
// Output:
// "Single object example:"
// "User data: {id: 1, name: 'Alice', email: 'alice@example.com'}"
// "Completed"
```

**When to use**:
- For creating an Observable from known values
- When you need a simple Observable that emits a fixed set of values
- For testing and prototyping
- As a fallback or default Observable in error handling

## range

**Purpose**: Creates an Observable that emits a sequence of numbers within a specified range.

```typescript
import { range } from 'rxjs';
import { map } from 'rxjs/operators';

// Example 1: Basic range
console.log('Basic range:');
range(1, 5).subscribe(
  value => console.log(`Value: ${value}`)
);
// Output:
// "Basic range:"
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"
// "Value: 5"

// Example 2: Range with transformation
console.log('Range with transformation:');
range(0, 6).pipe(
  map(x => x * x)
).subscribe(
  value => console.log(`Square: ${value}`)
);
// Output:
// "Range with transformation:"
// "Square: 0"
// "Square: 1"
// "Square: 4"
// "Square: 9"
// "Square: 16"
// "Square: 25"

// Example 3: Creating a range of dates
console.log('Date range:');
const today = new Date();
range(0, 7).pipe(
  map(dayOffset => {
    const date = new Date(today);
    date.setDate(today.getDate() + dayOffset);
    return date.toISOString().split('T')[0]; // YYYY-MM-DD format
  })
).subscribe(
  dateStr => console.log(`Date: ${dateStr}`)
);
// Output:
// "Date range:"
// "Date: 2023-08-20" (today)
// "Date: 2023-08-21"
// "Date: 2023-08-22"
// "Date: 2023-08-23"
// "Date: 2023-08-24"
// "Date: 2023-08-25"
// "Date: 2023-08-26"
```

**When to use**:
- When you need to generate a sequence of consecutive numbers
- For pagination implementations
- When creating test data with sequential values
- For generating date ranges or other sequential data

## throw

**Purpose**: Creates an Observable that immediately emits an error.

```typescript
import { throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

// Example 1: Basic error throwing
console.log('Basic throwError:');
throwError(() => new Error('This is an error!')).subscribe({
  next: value => console.log(`Value: ${value}`),
  error: err => console.error(`Error caught: ${err.message}`),
  complete: () => console.log('Completed')
});
// Output:
// "Basic throwError:"
// "Error caught: This is an error!"

// Example 2: Using throwError in error handling
console.log('throwError with catchError:');
throwError(() => new Error('Original error')).pipe(
  catchError(err => {
    console.log(`Handling error: ${err.message}`);
    // Return a new Observable or rethrow
    return throwError(() => new Error('Transformed error'));
  })
).subscribe({
  next: value => console.log(`Value: ${value}`),
  error: err => console.error(`Final error: ${err.message}`),
  complete: () => console.log('Completed')
});
// Output:
// "throwError with catchError:"
// "Handling error: Original error"
// "Final error: Transformed error"
```

**When to use**:
- For testing error handling logic
- When you need to create an Observable that represents a failure
- In conditional operators to signal errors
- For transforming errors in error handling chains

## timer

**Purpose**: Creates an Observable that emits a value after a specified delay, and optionally repeats at specified intervals.

```typescript
import { timer } from 'rxjs';
import { take } from 'rxjs/operators';

// Example 1: Single delayed emission
console.log('Single delayed emission:');
console.log('Starting timer...');
timer(2000).subscribe(
  value => console.log(`Emitted after 2 seconds: ${value}`),
  null,
  () => console.log('Completed')
);
// Output:
// "Single delayed emission:"
// "Starting timer..."
// (after 2 seconds)
// "Emitted after 2 seconds: 0"
// "Completed"

// Example 2: Repeated emissions with interval
console.log('Repeated emissions:');
console.log('Starting timer with interval...');
timer(1000, 500).pipe( // Start after 1s, then every 500ms
  take(5) // Take only 5 values
).subscribe(
  value => console.log(`Emission ${value}`),
  null,
  () => console.log('Completed')
);
// Output:
// "Repeated emissions:"
// "Starting timer with interval..."
// (after 1 second)
// "Emission 0"
// (after 500ms)
// "Emission 1"
// (after 500ms)
// "Emission 2"
// (after 500ms)
// "Emission 3"
// (after 500ms)
// "Emission 4"
// "Completed"
```

**When to use**:
- For delayed execution of operations
- When implementing timeouts
- For creating polling mechanisms with initial delay
- When you need more control than interval (initial delay + periodic emissions)

## Comparison of Creation Operators

| Operator | Purpose | Emission Behavior | Use Case |
|----------|---------|-------------------|----------|
| `ajax` | Create Observable from HTTP request | Emits response object | API calls, data fetching |
| `create` | Create custom Observable | Custom emission logic | Wrapping non-RxJS async APIs |
| `defer` | Create Observable factory | New Observable per subscriber | Dynamic Observable creation |
| `empty` | Create empty Observable | No emissions, just completes | Default/fallback Observable |
| `from` | Convert various sources to Observable | Emits items from source | Converting arrays/promises |
| `fromEvent` | Create Observable from events | Emits event objects | DOM event handling |
| `generate` | Create sequence with state machine | Emits based on iteration logic | Complex sequences |
| `interval` | Emit sequential numbers periodically | Emits incrementing numbers | Timers, polling |
| `of` | Create Observable from arguments | Emits each argument | Simple value streams |
| `range` | Create sequence of numbers | Emits range of numbers | Sequential number generation |
| `throw` | Create error Observable | Emits error immediately | Error testing, fallbacks |
| `timer` | Delayed and/or periodic emissions | Emits after delay, then periodically | Timeouts, delayed operations |

## Practical Examples

### Data Fetching with `ajax`

```typescript
import { ajax } from 'rxjs/ajax';
import { map, catchError, retry, switchMap } from 'rxjs/operators';
import { of, timer } from 'rxjs';

// Function to fetch user data with retry logic
function fetchUserData(userId: number) {
  return ajax(`https://jsonplaceholder.typicode.com/users/${userId}`).pipe(
    map(response => response.response),
    retry(3), // Retry up to 3 times on error
    catchError(error => {
      console.error('Error fetching user:', error);
      return of({ id: userId, name: 'Unknown User', error: true });
    })
  );
}

// Function to fetch user with exponential backoff retry
function fetchUserWithBackoff(userId: number) {
  return timer(0).pipe(
    switchMap(() => 
      ajax(`https://jsonplaceholder.typicode.com/users/${userId}`).pipe(
        map(response => response.response),
        catchError((error, attempt) => {
          // Retry with exponential backoff
          const retryAttempt = attempt as any + 1;
          if (retryAttempt > 3) {
            return of({ id: userId, name: 'Unknown User', error: true });
          }
          console.log(`Retry attempt ${retryAttempt} after ${Math.pow(2, retryAttempt)} seconds`);
          return timer(Math.pow(2, retryAttempt) * 1000).pipe(
            switchMap(() => fetchUserWithBackoff(userId))
          );
        })
      )
    )
  );
}

// Usage
fetchUserData(1).subscribe(
  user => console.log('User data:', user)
);
```

### Event Handling with `fromEvent`

```typescript
import { fromEvent, merge } from 'rxjs';
import { map, debounceTime, distinctUntilChanged, filter } from 'rxjs/operators';

// Implement a search box with typeahead
function setupSearchBox(inputId: string, resultsId: string) {
  const input = document.getElementById(inputId);
  const results = document.getElementById(resultsId);
  
  if (!input || !results) return;
  
  // Create Observables for different events
  const keyup$ = fromEvent(input, 'keyup');
  const focus$ = fromEvent(input, 'focus');
  const blur$ = fromEvent(input, 'blur');
  
  // Handle search input
  keyup$.pipe(
    map((event: any) => event.target.value.trim()),
    filter(text => text.length > 2),
    debounceTime(300),
    distinctUntilChanged()
  ).subscribe(searchTerm => {
    console.log(`Searching for: ${searchTerm}`);
    // In a real app, you would call an API here
    results.innerHTML = `<p>Search results for "${searchTerm}"...</p>`;
  });
  
  // Show/hide results based on focus
  merge(
    focus$.pipe(map(() => true)),
    blur$.pipe(map(() => false))
  ).subscribe(isFocused => {
    results.style.display = isFocused ? 'block' : 'none';
  });
}

// Usage
// setupSearchBox('search-input', 'search-results');
```

### Timer-based Animation with `interval` and `timer`

```typescript
import { interval, timer } from 'rxjs';
import { takeUntil, tap, scan } from 'rxjs/operators';

// Create a progress bar that fills up over time
function createProgressBar(durationMs: number, onComplete: () => void) {
  const tick$ = interval(50); // Update every 50ms
  const timeout$ = timer(durationMs); // End after duration
  
  return tick$.pipe(
    scan((progress) => {
      // Calculate progress percentage
      const newProgress = progress + (50 / durationMs) * 100;
      return Math.min(newProgress, 100); // Cap at 100%
    }, 0),
    tap(progress => {
      console.log(`Progress: ${Math.round(progress)}%`);
      // In a real app, you would update the DOM here
    }),
    takeUntil(timeout$),
    tap({
      complete: () => {
        console.log('Progress complete!');
        onComplete();
      }
    })
  );
}

// Usage
createProgressBar(3000, () => console.log('Action completed!')).subscribe();
```

## Conclusion

Creation operators in RxJS provide a variety of ways to create Observable streams from different sources. By understanding these operators, you can effectively convert any data source or event into an Observable stream that can be processed using RxJS operators.

Key takeaways:
- Use `ajax` for HTTP requests
- Use `from` to convert arrays, promises, or iterables to Observables
- Use `fromEvent` for DOM or EventEmitter events
- Use `of` for simple value emissions
- Use `interval` and `timer` for time-based operations
- Use `create` when you need custom Observable behavior
- Use `defer` for dynamic Observable creation

These creation operators form the foundation of reactive programming with RxJS, allowing you to start with any data source and transform it into a reactive stream.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
