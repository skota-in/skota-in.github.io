# RxJS Filter Operators: Examples and Use Cases

*Published on April 13, 2025*

## Introduction

RxJS filter operators allow you to control which values pass through your Observable streams. These operators are essential for managing data flow and focusing on the values that matter to your application. This post explores the most commonly used filter operators with practical examples and real-world use cases.

## Table of Contents

1. [audit](#audit)
2. [auditTime](#audittime)
3. [debounce](#debounce)
4. [debounceTime ⭐](#debouncetime)
5. [distinct](#distinct)
6. [distinctUntilChanged ⭐](#distinctuntilchanged)
7. [distinctUntilKeyChanged](#distinctuntilkeychanged)
8. [filter ⭐](#filter)
9. [find](#find)
10. [first](#first)
11. [ignoreElements](#ignoreelements)
12. [last](#last)
13. [sample](#sample)
14. [single](#single)
15. [skip](#skip)
16. [skipUntil](#skipuntil)
17. [skipWhile](#skipwhile)
18. [take ⭐](#take)
19. [takeLast](#takelast)
20. [takeUntil ⭐](#takeuntil)
21. [takeWhile](#takewhile)
22. [throttle](#throttle)
23. [throttleTime](#throttletime)
24. [Comparison of Key Operators](#comparison-of-key-operators)

## audit

**Purpose**: Samples the source Observable when another Observable emits, emitting the most recent value from the source.

```typescript
import { interval } from 'rxjs';
import { audit } from 'rxjs/operators';

// Emit value every second
const source$ = interval(1000);

// Sample the most recent value when the audit Observable emits (every 2 seconds)
source$.pipe(
  audit(() => interval(2000))
).subscribe(
  value => console.log(`Audit: ${value}`)
);

// Output:
// "Audit: 1"  (after 2 seconds)
// "Audit: 3"  (after 4 seconds)
// "Audit: 5"  (after 6 seconds)
// ...and so on
```

**When to use**:
- When you need to sample a rapidly changing value at specific intervals
- For throttling high-frequency events while ensuring you get the most recent value
- When implementing "last value wins" logic for rapidly updating data
- For performance optimization when handling fast-changing values

## auditTime

**Purpose**: Samples the source Observable periodically based on a time span, emitting the most recent value.

```typescript
import { interval } from 'rxjs';
import { auditTime } from 'rxjs/operators';

// Emit value every 500ms
const source$ = interval(500);

// Sample the most recent value every 2 seconds
source$.pipe(
  auditTime(2000)
).subscribe(
  value => console.log(`AuditTime: ${value}`)
);

// Output:
// "AuditTime: 3"  (after 2 seconds)
// "AuditTime: 7"  (after 4 seconds)
// "AuditTime: 11" (after 6 seconds)
// ...and so on
```

**When to use**:
- When you need a simpler version of `audit` with a fixed time duration
- For throttling high-frequency events like mouse moves or scroll events
- When you want to limit the rate of emissions but always get the latest value
- For performance optimization in UI updates

## debounce

**Purpose**: Delays values from the source Observable until another Observable emits, discarding intermediate values.

```typescript
import { of, interval } from 'rxjs';
import { debounce, delay, concatMap } from 'rxjs/operators';

// Simulate typing in a search box with varying speeds
const typingStream$ = of('h', 'he', 'hel', 'hell', 'hello').pipe(
  concatMap((value, index) =>
    of(value).pipe(delay(index === 3 ? 1000 : 300)) // Pause longer after "hell"
  )
);

// Only emit when there's a pause in typing
typingStream$.pipe(
  debounce(() => interval(500))
).subscribe(
  value => console.log(`Debounce: ${value}`)
);

// Output:
// "Debounce: hell"  (after the 1000ms pause)
// "Debounce: hello" (after the final value)
```

**When to use**:
- When you need complex debounce logic with dynamic timing
- For implementing custom debounce behavior based on external factors
- When the debounce duration needs to vary based on the emitted value
- For advanced scenarios where `debounceTime` is insufficient

## debounceTime ⭐

**Purpose**: Delays values from the source Observable for a specified time period and emits only the most recent value, discarding intermediate values.

```typescript
import { fromEvent } from 'rxjs';
import { debounceTime, map } from 'rxjs/operators';

// In a browser environment
const searchInput = document.getElementById('search');
const searchTerms$ = fromEvent(searchInput, 'input').pipe(
  map((event: any) => event.target.value),
  debounceTime(500)
);

// Only makes API calls when user stops typing for 500ms
searchTerms$.subscribe(
  term => console.log(`Searching for: ${term}`)
);

// Output (depends on typing speed):
// User types "react" quickly, then pauses
// "Searching for: react"
// User continues typing to "reactive"
// "Searching for: reactive"
```

**When to use**:
- For handling user input in search boxes, forms, and autocomplete
- When you need to wait for a pause in activity before processing
- For window resize or scroll events where you only care about the final position
- When you want to reduce the number of API calls or expensive operations

## distinct

**Purpose**: Filters out duplicate values, emitting only values that haven't been emitted before.

```typescript
import { from } from 'rxjs';
import { distinct } from 'rxjs/operators';

// A stream with duplicate values
const numbers$ = from([1, 1, 2, 2, 3, 3, 4, 5, 3, 2, 1]);

// Filter out duplicates
numbers$.pipe(
  distinct()
).subscribe(
  value => console.log(`Distinct: ${value}`)
);

// Output:
// "Distinct: 1"
// "Distinct: 2"
// "Distinct: 3"
// "Distinct: 4"
// "Distinct: 5"
```

**With objects**:

```typescript
import { from } from 'rxjs';
import { distinct } from 'rxjs/operators';

const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 1, name: 'John Updated' }, // Same ID as first user
  { id: 3, name: 'Bob' }
];

from(users).pipe(
  distinct(user => user.id)
).subscribe(
  user => console.log(`Distinct User: ${user.name} (ID: ${user.id})`)
);

// Output:
// "Distinct User: John (ID: 1)"
// "Distinct User: Jane (ID: 2)"
// "Distinct User: Bob (ID: 3)"
```

**When to use**:
- When you need to ensure uniqueness across an entire stream
- For filtering out duplicate items in a dataset
- When implementing "show only once" functionality
- For deduplicating user actions or events

## distinctUntilChanged ⭐

**Purpose**: Emits values that are different from the previous value, filtering out consecutive duplicates.

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

// A stream with consecutive duplicates
const numbers$ = from([1, 1, 2, 2, 3, 1, 1, 3]);

// Only emit when the value changes
numbers$.pipe(
  distinctUntilChanged()
).subscribe(
  value => console.log(`DistinctUntilChanged: ${value}`)
);

// Output:
// "DistinctUntilChanged: 1"
// "DistinctUntilChanged: 2"
// "DistinctUntilChanged: 3"
// "DistinctUntilChanged: 1"
// "DistinctUntilChanged: 3"
```

**Real-world example**:

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

// Temperature sensor readings (°C)
const temperatures$ = from([22.5, 22.5, 22.7, 22.7, 22.6, 22.8, 22.8]);

// Only notify when temperature changes
temperatures$.pipe(
  distinctUntilChanged()
).subscribe(
  temp => console.log(`Temperature changed to: ${temp}°C`)
);

// Output:
// "Temperature changed to: 22.5°C"
// "Temperature changed to: 22.7°C"
// "Temperature changed to: 22.6°C"
// "Temperature changed to: 22.8°C"
```

**When to use**:
- When you only care about changes in values, not all unique values
- For monitoring state changes in an application
- When implementing change detection for data streams
- For reducing redundant updates in UI components

## distinctUntilKeyChanged

**Purpose**: Emits when the specified key value changes from the previous item, filtering out consecutive items with the same key value.

```typescript
import { from } from 'rxjs';
import { distinctUntilKeyChanged } from 'rxjs/operators';

const people = [
  { name: 'John', age: 30 },
  { name: 'John', age: 31 }, // Age changed
  { name: 'Jane', age: 25 }, // Name changed
  { name: 'Jane', age: 25 }, // No change
  { name: 'Jane', age: 26 }  // Age changed
];

// Only emit when the name changes
from(people).pipe(
  distinctUntilKeyChanged('name')
).subscribe(
  person => console.log(`Name changed to: ${person.name}, Age: ${person.age}`)
);

// Output:
// "Name changed to: John, Age: 30"
// "Name changed to: Jane, Age: 25"
```

**When to use**:
- When working with objects and you need to track changes to a specific property
- For monitoring changes to a particular aspect of your data
- When implementing property-specific change detection
- For filtering streams of complex objects based on key properties

## filter ⭐

**Purpose**: Emits values that pass the provided condition, filtering out values that don't satisfy the predicate.

```typescript
import { from } from 'rxjs';
import { filter } from 'rxjs/operators';

// A stream of numbers
const numbers$ = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

// Only emit even numbers
numbers$.pipe(
  filter(num => num % 2 === 0)
).subscribe(
  value => console.log(`Even number: ${value}`)
);

// Output:
// "Even number: 2"
// "Even number: 4"
// "Even number: 6"
// "Even number: 8"
// "Even number: 10"
```

**Real-world example**:

```typescript
import { from } from 'rxjs';
import { filter } from 'rxjs/operators';

// Stream of HTTP responses
const responses = [
  { status: 200, data: 'Success 1' },
  { status: 404, data: 'Not Found' },
  { status: 200, data: 'Success 2' },
  { status: 500, data: 'Server Error' },
  { status: 200, data: 'Success 3' }
];

// Only process successful responses
from(responses).pipe(
  filter(response => response.status === 200)
).subscribe(
  response => console.log(`Successful response: ${response.data}`)
);

// Output:
// "Successful response: Success 1"
// "Successful response: Success 2"
// "Successful response: Success 3"
```

**When to use**:
- When you need to include or exclude values based on a condition
- For implementing conditional logic in data streams
- When processing only certain types of events or data
- For validating data before further processing

## find

**Purpose**: Emits only the first value from the source Observable that meets a specified condition, then completes.

```typescript
import { from } from 'rxjs';
import { find } from 'rxjs/operators';

// A stream of values
const numbers$ = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

// Find the first number greater than 5
numbers$.pipe(
  find(num => num > 5)
).subscribe(
  value => console.log(`First number > 5: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "First number > 5: 6"
// "Completed"
```

**When to use**:
- When you need only the first item that matches a condition
- For finding a specific element in a stream
- When you want to complete the Observable after finding a match
- For implementing search functionality where only the first result matters

## first

**Purpose**: Emits only the first value (or the first value that meets a predicate) from the source Observable, then completes.

```typescript
import { from } from 'rxjs';
import { first } from 'rxjs/operators';

// Example 1: Basic usage - first value
const numbers$ = from([1, 2, 3, 4, 5]);

numbers$.pipe(
  first()
).subscribe(
  value => console.log(`First value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "First value: 1"
// "Completed"

// Example 2: With predicate
const numbers2$ = from([1, 2, 3, 4, 5]);

numbers2$.pipe(
  first(num => num % 2 === 0)
).subscribe(
  value => console.log(`First even number: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "First even number: 2"
// "Completed"
```

**When to use**:
- When you only need the first item from a stream
- For initializing values from a stream
- When you want to complete the Observable after getting the first value
- For implementing "take first match" functionality

## ignoreElements

**Purpose**: Ignores all elements from the source Observable, only passes through error or completion notifications.

```typescript
import { interval } from 'rxjs';
import { ignoreElements, take } from 'rxjs/operators';

// Emit values every 100ms, take 5 values
const source$ = interval(100).pipe(take(5));

// Ignore all values, only care about completion
source$.pipe(
  ignoreElements()
).subscribe(
  value => console.log(`Value: ${value}`), // This will never be called
  err => console.error(err),
  () => console.log('Operation completed!')
);

// Output (after ~500ms):
// "Operation completed!"
```

**When to use**:
- When you only care about when a stream completes, not its values
- For side-effect only operations where the values don't matter
- When implementing notification systems that only care about completion
- For converting value streams into completion signals

## last

**Purpose**: Emits only the last value (or the last value that meets a predicate) from the source Observable, then completes.

```typescript
import { from } from 'rxjs';
import { last } from 'rxjs/operators';

// Example 1: Basic usage - last value
const numbers$ = from([1, 2, 3, 4, 5]);

numbers$.pipe(
  last()
).subscribe(
  value => console.log(`Last value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Last value: 5"
// "Completed"

// Example 2: With predicate
const numbers2$ = from([1, 2, 3, 4, 5, 6, 7, 8]);

numbers2$.pipe(
  last(num => num % 2 === 0)
).subscribe(
  value => console.log(`Last even number: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Last even number: 8"
// "Completed"
```

**When to use**:
- When you only need the final value from a stream
- For getting the most recent or final state
- When you want to wait for a stream to complete before taking action
- For implementing "last value wins" scenarios

## sample

**Purpose**: Emits the most recently emitted value from the source Observable whenever another Observable, the notifier, emits.

```typescript
import { interval } from 'rxjs';
import { sample } from 'rxjs/operators';

// Emit value every 100ms
const source$ = interval(100);

// Sample the source every 500ms
source$.pipe(
  sample(interval(500)),
  take(5)
).subscribe(
  value => console.log(`Sampled value: ${value}`)
);

// Output (approximately):
// "Sampled value: 4"  (after 500ms)
// "Sampled value: 9"  (after 1000ms)
// "Sampled value: 14" (after 1500ms)
// "Sampled value: 19" (after 2000ms)
// "Sampled value: 24" (after 2500ms)
```

**When to use**:
- When you need to sample a stream at specific intervals
- For implementing polling behavior
- When you want to get snapshots of rapidly changing values
- For reducing the frequency of emissions based on another Observable

## single

**Purpose**: Emits the only value that matches a predicate, or the only value if no predicate is provided. Throws an error if the source emits multiple matching values.

```typescript
import { from } from 'rxjs';
import { single } from 'rxjs/operators';

// Example 1: Stream with a single value
const singleValue$ = from([42]);

singleValue$.pipe(
  single()
).subscribe(
  value => console.log(`Single value: ${value}`),
  err => console.error(`Error: ${err}`),
  () => console.log('Completed')
);

// Output:
// "Single value: 42"
// "Completed"

// Example 2: Stream with a single matching value
const numbers$ = from([1, 2, 3, 4, 5]);

numbers$.pipe(
  single(num => num === 3)
).subscribe(
  value => console.log(`Single matching value: ${value}`),
  err => console.error(`Error: ${err}`),
  () => console.log('Completed')
);

// Output:
// "Single matching value: 3"
// "Completed"

// Example 3: Stream with multiple matching values (error)
const multipleMatches$ = from([1, 2, 3, 4, 5]);

multipleMatches$.pipe(
  single(num => num > 2)
).subscribe(
  value => console.log(`Single matching value: ${value}`),
  err => console.error(`Error: ${err}`),
  () => console.log('Completed')
);

// Output:
// "Error: Sequence contains more than one element"
```

**When to use**:
- When you expect exactly one value to match a condition
- For validation scenarios where multiple matches would be an error
- When implementing uniqueness constraints
- For ensuring a stream contains exactly one specific value

## skip

**Purpose**: Skips the first `count` values emitted by the source Observable, then emits all subsequent values.

```typescript
import { from } from 'rxjs';
import { skip } from 'rxjs/operators';

// A stream of values
const numbers$ = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

// Skip the first 5 values
numbers$.pipe(
  skip(5)
).subscribe(
  value => console.log(`Value after skip: ${value}`)
);

// Output:
// "Value after skip: 6"
// "Value after skip: 7"
// "Value after skip: 8"
// "Value after skip: 9"
// "Value after skip: 10"
```

**When to use**:
- When you want to ignore the first N values from a stream
- For skipping initial states or events
- When implementing pagination (skipping already loaded items)
- For ignoring startup or initialization values

## skipUntil

**Purpose**: Skips values from the source Observable until a second Observable emits a value, then emits all subsequent values.

```typescript
import { interval, timer } from 'rxjs';
import { skipUntil, take } from 'rxjs/operators';

// Emit value every 100ms
const source$ = interval(100);

// Skip values until 500ms has passed
const result$ = source$.pipe(
  skipUntil(timer(500)),
  take(5)
);

result$.subscribe(
  value => console.log(`Value after skipUntil: ${value}`)
);

// Output (approximately):
// "Value after skipUntil: 5" (first value after 500ms)
// "Value after skipUntil: 6"
// "Value after skipUntil: 7"
// "Value after skipUntil: 8"
// "Value after skipUntil: 9"
```

**Real-world example**:

```typescript
import { fromEvent, interval } from 'rxjs';
import { skipUntil, takeUntil } from 'rxjs/operators';

// In a browser environment
const startButton = document.getElementById('start');
const stopButton = document.getElementById('stop');

// Create Observables from button clicks
const startClick$ = fromEvent(startButton, 'click');
const stopClick$ = fromEvent(stopButton, 'click');

// Emit values every 100ms, but only after start button is clicked
// and until stop button is clicked
interval(100).pipe(
  skipUntil(startClick$),
  takeUntil(stopClick$)
).subscribe(
  count => console.log(`Counter: ${count}`)
);

// Output (after clicking start):
// "Counter: 0"
// "Counter: 1"
// "Counter: 2"
// ... (continues until stop is clicked)
```

**When to use**:
- When you want to ignore values until a specific event occurs
- For implementing start/resume functionality
- When you need to wait for user interaction before processing
- For coordinating the start of multiple streams

## skipWhile

**Purpose**: Skips values from the source Observable as long as a specified condition is true, then emits all subsequent values once the condition becomes false.

```typescript
import { from } from 'rxjs';
import { skipWhile } from 'rxjs/operators';

// A stream of values
const numbers$ = from([1, 2, 3, 4, 5, 4, 3, 2, 1]);

// Skip values while they're less than 5
numbers$.pipe(
  skipWhile(value => value < 5)
).subscribe(
  value => console.log(`Value after skipWhile: ${value}`)
);

// Output:
// "Value after skipWhile: 5"
// "Value after skipWhile: 4"
// "Value after skipWhile: 3"
// "Value after skipWhile: 2"
// "Value after skipWhile: 1"
```

**When to use**:
- When you want to skip values based on a dynamic condition
- For ignoring values until a threshold is reached
- When implementing state-based filtering
- For skipping initial invalid or incomplete data

## take ⭐

**Purpose**: Emits only the first `count` values from the source Observable, then completes.

```typescript
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

// Emit values every 100ms
const source$ = interval(100);

// Take only the first 5 values
source$.pipe(
  take(5)
).subscribe(
  value => console.log(`Value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Value: 0"
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"
// "Completed"
```

**Real-world example**:

```typescript
import { fromEvent } from 'rxjs';
import { take, map } from 'rxjs/operators';

// In a browser environment
const clicksUntilDisabled$ = fromEvent(document, 'click').pipe(
  take(3),
  map((event, index) => {
    const remaining = 2 - index;
    return `Clicked! ${remaining} more click${remaining !== 1 ? 's' : ''} until disabled`;
  })
);

clicksUntilDisabled$.subscribe(
  message => console.log(message),
  null,
  () => console.log('Button disabled')
);

// Output (after 3 clicks):
// "Clicked! 2 more clicks until disabled"
// "Clicked! 1 more click until disabled"
// "Clicked! 0 more clicks until disabled"
// "Button disabled"
```

**When to use**:
- When you need to limit the number of values from a stream
- For implementing pagination or "load more" functionality
- When you want an Observable to complete after a specific number of emissions
- For creating finite streams from infinite sources

## takeLast

**Purpose**: Emits only the last `count` values from the source Observable, after it completes.

```typescript
import { range } from 'rxjs';
import { takeLast } from 'rxjs/operators';

// A finite stream of 10 values
const numbers$ = range(1, 10);

// Take only the last 3 values
numbers$.pipe(
  takeLast(3)
).subscribe(
  value => console.log(`Value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Value: 8"
// "Value: 9"
// "Value: 10"
// "Completed"
```

**When to use**:
- When you only need the most recent N values from a completed stream
- For getting the last few items in a list
- When implementing "most recent" functionality
- For buffering the last N values before processing

## takeUntil ⭐

**Purpose**: Emits values from the source Observable until a second Observable emits a value, then completes.

```typescript
import { interval, timer } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

// Emit value every 100ms
const source$ = interval(100);

// Take values until 1 second has passed
source$.pipe(
  takeUntil(timer(1000))
).subscribe(
  value => console.log(`Value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output (approximately):
// "Value: 0"
// "Value: 1"
// "Value: 2"
// ... (up to around Value: 9)
// "Completed"
```

**Real-world example**:

```typescript
import { fromEvent, interval } from 'rxjs';
import { takeUntil, map } from 'rxjs/operators';

// In a browser environment
const stopButton = document.getElementById('stop');
const stopClick$ = fromEvent(stopButton, 'click');

// Create a timer that stops when the button is clicked
const timer$ = interval(1000).pipe(
  map(value => value + 1),
  takeUntil(stopClick$)
);

timer$.subscribe(
  seconds => console.log(`Timer: ${seconds} seconds`),
  null,
  () => console.log('Timer stopped')
);

// Output (until stop button is clicked):
// "Timer: 1 seconds"
// "Timer: 2 seconds"
// "Timer: 3 seconds"
// ...
// "Timer stopped"
```

**When to use**:
- When you want to complete a stream based on an external event
- For implementing cancellation or stop functionality
- When creating time-limited operations
- For preventing memory leaks in long-lived streams

## takeWhile

**Purpose**: Emits values from the source Observable as long as a specified condition is true, then completes once the condition becomes false.

```typescript
import { from } from 'rxjs';
import { takeWhile } from 'rxjs/operators';

// A stream of values
const numbers$ = from([1, 2, 3, 4, 5, 4, 3, 2, 1]);

// Take values while they're less than 5
numbers$.pipe(
  takeWhile(value => value < 5)
).subscribe(
  value => console.log(`Value: ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Value: 1"
// "Value: 2"
// "Value: 3"
// "Value: 4"
// "Completed"
```

**With inclusive option**:

```typescript
import { from } from 'rxjs';
import { takeWhile } from 'rxjs/operators';

// A stream of values
const numbers$ = from([1, 2, 3, 4, 5, 4, 3, 2, 1]);

// Take values while they're less than or equal to 5 (inclusive)
numbers$.pipe(
  takeWhile(value => value <= 5, true)
).subscribe(
  value => console.log(`Value (inclusive): ${value}`),
  null,
  () => console.log('Completed')
);

// Output:
// "Value (inclusive): 1"
// "Value (inclusive): 2"
// "Value (inclusive): 3"
// "Value (inclusive): 4"
// "Value (inclusive): 5"
// "Completed"
```

**When to use**:
- When you want to take values until a condition is no longer met
- For implementing validation-based termination
- When processing data until a threshold is reached
- For creating conditional finite streams

## throttle

**Purpose**: Emits a value from the source Observable, then ignores subsequent values for a duration determined by another Observable, then repeats this process.

```typescript
import { interval } from 'rxjs';
import { throttle, take } from 'rxjs/operators';

// Emit value every 100ms
const source$ = interval(100);

// Emit a value, then ignore for 300ms, then repeat
source$.pipe(
  throttle(() => interval(300)),
  take(5)
).subscribe(
  value => console.log(`Throttled value: ${value}`)
);

// Output (approximately):
// "Throttled value: 0"  (at 0ms)
// "Throttled value: 3"  (at 400ms)
// "Throttled value: 7"  (at 800ms)
// "Throttled value: 11" (at 1200ms)
// "Throttled value: 15" (at 1600ms)
```

**When to use**:
- When you need to limit the rate of emissions
- For implementing rate limiting with dynamic durations
- When handling high-frequency events like scrolling or resizing
- For reducing the processing load of frequent events

## throttleTime

**Purpose**: Emits a value from the source Observable, then ignores subsequent values for a specified duration, then repeats this process.

```typescript
import { interval } from 'rxjs';
import { throttleTime, take } from 'rxjs/operators';

// Emit value every 100ms
const source$ = interval(100);

// Emit a value, then ignore for 300ms, then repeat
source$.pipe(
  throttleTime(300),
  take(5)
).subscribe(
  value => console.log(`Throttled value: ${value}`)
);

// Output (approximately):
// "Throttled value: 0"  (at 0ms)
// "Throttled value: 3"  (at 400ms)
// "Throttled value: 7"  (at 800ms)
// "Throttled value: 11" (at 1200ms)
// "Throttled value: 15" (at 1600ms)
```

**Real-world example**:

```typescript
import { fromEvent } from 'rxjs';
import { throttleTime, map } from 'rxjs/operators';

// In a browser environment
const scrollEvents$ = fromEvent(window, 'scroll').pipe(
  throttleTime(300),
  map(() => {
    return {
      scrollY: window.scrollY,
      timestamp: new Date().toISOString()
    };
  })
);

scrollEvents$.subscribe(
  info => console.log(`Scroll position: ${info.scrollY}px at ${info.timestamp}`)
);

// Output during rapid scrolling (only emits every 300ms):
// "Scroll position: 150px at 2023-08-25T12:34:56.789Z"
// "Scroll position: 450px at 2023-08-25T12:34:57.089Z"
// "Scroll position: 720px at 2023-08-25T12:34:57.389Z"
```

**When to use**:
- When you need to limit the rate of emissions with a fixed time interval
- For implementing rate limiting for UI events
- When processing high-frequency events like mouse moves
- For reducing server load from frequent user actions

## Comparison of Key Operators

| Operator | Emission Trigger | Memory Usage | Use Case |
|----------|------------------|--------------|----------|
| `audit`/`auditTime` | Time-based sampling | Low | Throttling with "latest value wins" |
| `debounce`/`debounceTime` | Pause in activity | Low | Wait for pause before processing |
| `distinct` | Never emitted before | High (keeps history) | Ensure absolute uniqueness |
| `distinctUntilChanged` | Different from previous | Low (only remembers last) | Process only when value changes |
| `distinctUntilKeyChanged` | Property different from previous | Low (only remembers last) | Track changes to specific properties |
| `filter` | Passes condition | None | Basic conditional filtering |
| `find` | First to pass condition | Low | Find first matching item |
| `first` | First item or match | Low | Get only the first item |
| `ignoreElements` | Never (values) | None | Process only completion/errors |
| `last` | Last item or match | Low | Get only the last item |
| `sample` | When notifier emits | Low | Sample at specific points |
| `single` | Only item or match | Low | Ensure exactly one match |
| `skip` | After N emissions | None | Ignore first N values |
| `skipUntil` | After notifier emits | Low | Ignore values until event |
| `skipWhile` | After condition fails | Low | Ignore values while condition is true |
| `take` | First N emissions | None | Limit to first N values |
| `takeLast` | Last N emissions | Medium (buffers values) | Get only the last N values |
| `takeUntil` | Until notifier emits | Low | Complete stream on event |
| `takeWhile` | While condition is true | Low | Complete when condition fails |
| `throttle`/`throttleTime` | First value, then after duration | Low | Rate limiting with "first value wins" |

## Practical Combinations

### Debounced Search with Minimum Length

```typescript
import { fromEvent } from 'rxjs';
import { map, debounceTime, filter, distinctUntilChanged } from 'rxjs/operators';

// In a browser environment
const searchInput = document.getElementById('search');
fromEvent(searchInput, 'input').pipe(
  map((event: any) => event.target.value.trim()),
  debounceTime(500),
  filter(term => term.length > 2),
  distinctUntilChanged()
).subscribe(
  term => console.log(`Searching for: ${term}`)
);
```

### Filtered and Throttled Mouse Movements

```typescript
import { fromEvent } from 'rxjs';
import { auditTime, map, filter } from 'rxjs/operators';

// In a browser environment
const mouseMoves$ = fromEvent(document, 'mousemove');
mouseMoves$.pipe(
  auditTime(1000), // Sample every second
  map((event: MouseEvent) => ({ x: event.clientX, y: event.clientY })),
  filter(pos => pos.x > window.innerWidth / 2) // Only right half of screen
).subscribe(
  pos => console.log(`Mouse in right half at: ${pos.x}, ${pos.y}`)
);
```

### Distinct User Actions

```typescript
import { from } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

const userActions = [
  { user: 'user1', action: 'click' },
  { user: 'user1', action: 'click' }, // Duplicate
  { user: 'user2', action: 'hover' },
  { user: 'user1', action: 'scroll' },
  { user: 'user2', action: 'click' }
];

from(userActions).pipe(
  distinctUntilChanged((prev, curr) =>
    prev.user === curr.user && prev.action === curr.action
  )
).subscribe(
  action => console.log(`User ${action.user} performed ${action.action}`)
);

// Output:
// "User user1 performed click"
// "User user2 performed hover"
// "User user1 performed scroll"
// "User user2 performed click"
```

## Conclusion

RxJS filter operators provide powerful tools for controlling the flow of data in your Observable streams. By understanding when and how to use each operator, you can build more responsive and efficient applications:

- Use `debounceTime` ⭐ for handling rapid user input and waiting for pauses in activity
- Use `distinctUntilChanged` ⭐ for processing only changed values and avoiding redundant updates
- Use `filter` ⭐ for basic conditional filtering and validating data
- Use `take` ⭐ for limiting the number of values from a stream
- Use `takeUntil` ⭐ for completing streams based on external events
- Use `throttleTime` for rate-limiting high-frequency events
- Use `skipUntil` and `skipWhile` for ignoring values until specific conditions are met

These operators become even more powerful when combined with other RxJS operators to create comprehensive data processing pipelines.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
