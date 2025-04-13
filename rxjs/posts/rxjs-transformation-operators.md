# RxJS Transformation Operators: Examples and Use Cases

*Published on June 20, 2023*

## Introduction

Transformation operators in RxJS allow you to modify, reshape, and process the data emitted by Observables. These operators are essential for converting raw data into the format your application needs. From simple value transformations to complex buffering strategies and mapping operations, transformation operators form the backbone of reactive data processing. This post explores the most commonly used transformation operators with practical examples.

## Table of Contents

1. [buffer](#buffer)
2. [bufferCount](#buffercount)
3. [bufferTime ⭐](#buffertime)
4. [bufferToggle](#buffertoggle)
5. [bufferWhen](#bufferwhen)
6. [concatMap ⭐](#concatmap)
7. [concatMapTo](#concatmapto)
8. [expand](#expand)
9. [exhaustMap](#exhaustmap)
10. [groupBy](#groupby)
11. [map ⭐](#map)
12. [mapTo](#mapto)
13. [mergeMap / flatMap ⭐](#mergemap--flatmap)
14. [mergeScan](#mergescan)
15. [partition](#partition)
16. [pluck](#pluck)
17. [reduce](#reduce)
18. [scan ⭐](#scan)
19. [switchMap ⭐](#switchmap)
20. [switchMapTo](#switchmapto)
21. [toArray](#toarray)
22. [window](#window)
23. [windowCount](#windowcount)
24. [windowTime](#windowtime)
25. [windowToggle](#windowtoggle)
26. [windowWhen](#windowwhen)
27. [Comparison of Key Operators](#comparison-of-key-operators)

## buffer

**Purpose**: Collects values from the source Observable until a notifier Observable emits, then emits the collected values as an array.

```typescript
import { interval, fromEvent } from 'rxjs';
import { buffer, take } from 'rxjs/operators';

// Create a source Observable that emits a value every 500ms
const source$ = interval(500);

// Use mouse clicks as the buffer boundary
const clicks$ = fromEvent(document, 'click');

// Buffer values from source until a click occurs
const buffered$ = source$.pipe(
  buffer(clicks$),
  take(3) // Take only 3 buffer emissions for the example
);

// Subscribe to see the buffered values
buffered$.subscribe(
  bufferedValues => console.log('Buffered values:', bufferedValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (after clicking three times, with varying intervals between clicks):
// Buffered values: [0, 1, 2, 3]
// Buffered values: [4, 5, 6]
// Buffered values: [7, 8, 9, 10, 11]
// Complete
```

**When to use**:
- When you need to collect values over time and process them as a batch
- For implementing custom debouncing or throttling logic
- When you want to process values only after a specific event occurs
- For grouping related events that occur within a time window

## bufferCount

**Purpose**: Collects values from the source Observable until the specified buffer size is reached, then emits the collected values as an array.

```typescript
import { interval } from 'rxjs';
import { bufferCount, take } from 'rxjs/operators';

// Example 1: Basic bufferCount
console.log('Basic bufferCount example:');
interval(500).pipe(
  bufferCount(3), // Collect 3 values before emitting
  take(4) // Take only 4 buffer emissions for the example
).subscribe(
  buffer => console.log('Buffer of 3:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Buffer of 3: [0, 1, 2]
// Buffer of 3: [3, 4, 5]
// Buffer of 3: [6, 7, 8]
// Buffer of 3: [9, 10, 11]
// Complete

// Example 2: Overlapping buffers
console.log('Overlapping bufferCount example:');
interval(500).pipe(
  bufferCount(3, 1), // Buffer size 3, start new buffer every 1 value
  take(4) // Take only 4 buffer emissions for the example
).subscribe(
  buffer => console.log('Overlapping buffer:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Overlapping buffer: [0, 1, 2]
// Overlapping buffer: [1, 2, 3]
// Overlapping buffer: [2, 3, 4]
// Overlapping buffer: [3, 4, 5]
// Complete
```

**When to use**:
- When you need to process data in fixed-size chunks
- For implementing sliding window algorithms
- When you want to group a specific number of events together
- For batch processing with fixed batch sizes

## bufferTime ⭐

**Purpose**: Collects values from the source Observable for a specified duration, then emits the collected values as an array.

```typescript
import { interval } from 'rxjs';
import { bufferTime, take } from 'rxjs/operators';

// Example 1: Basic bufferTime
console.log('Basic bufferTime example:');
interval(300).pipe(
  bufferTime(1000), // Collect values for 1 second
  take(5) // Take only 5 buffer emissions for the example
).subscribe(
  buffer => console.log('1-second buffer:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// 1-second buffer: [0, 1, 2]
// 1-second buffer: [3, 4, 5]
// 1-second buffer: [6, 7, 8]
// 1-second buffer: [9, 10, 11]
// 1-second buffer: [12, 13, 14]
// Complete

// Example 2: Overlapping time buffers
console.log('Overlapping bufferTime example:');
interval(300).pipe(
  bufferTime(1000, 500), // 1-second buffer, new buffer every 500ms
  take(5) // Take only 5 buffer emissions for the example
).subscribe(
  buffer => console.log('Overlapping time buffer:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Overlapping time buffer: [0, 1, 2]
// Overlapping time buffer: [1, 2, 3, 4]
// Overlapping time buffer: [3, 4, 5, 6]
// Overlapping time buffer: [5, 6, 7, 8]
// Overlapping time buffer: [7, 8, 9, 10]
// Complete
```

**When to use**:
- For time-based batching of events
- When implementing analytics or logging that groups events by time periods
- For reducing the frequency of updates in high-frequency data streams
- When you need to sample data at regular intervals
- For implementing time-window based operations

## bufferToggle

**Purpose**: Collects values from the source Observable starting when an opening Observable emits, and ending when a closing Observable emits.

```typescript
import { interval, timer } from 'rxjs';
import { bufferToggle, take } from 'rxjs/operators';

// Source emits every 100ms
const source$ = interval(100);

// Opening observable: emit every 500ms
const openings$ = interval(500);

// Closing observable factory: emit after 300ms
const closingSelector = () => timer(300);

// Buffer values when openings$ emits, and collect until closingSelector emits
source$.pipe(
  bufferToggle(openings$, closingSelector),
  take(3) // Take only 3 buffer emissions for the example
).subscribe(
  buffer => console.log('Buffered values:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Buffered values: [5, 6, 7]
// Buffered values: [10, 11, 12]
// Buffered values: [15, 16, 17]
// Complete
```

**When to use**:
- For complex buffering scenarios with dynamic start and end conditions
- When you need to capture values only during specific time windows
- For implementing recording or sampling functionality
- When you need fine-grained control over when buffering starts and stops

## bufferWhen

**Purpose**: Collects values from the source Observable and emits them as an array when a factory function produces an Observable that emits.

```typescript
import { interval } from 'rxjs';
import { bufferWhen, take } from 'rxjs/operators';

// Source emits every 100ms
const source$ = interval(100);

// Buffer closing factory: create random-duration buffers
const closingSelector = () => {
  // Random buffer duration between 500ms and 1500ms
  const randomDuration = Math.floor(Math.random() * 1000) + 500;
  return interval(randomDuration).pipe(take(1));
};

// Buffer values until closingSelector emits
source$.pipe(
  bufferWhen(closingSelector),
  take(3) // Take only 3 buffer emissions for the example
).subscribe(
  buffer => console.log('Buffered with random duration:', buffer),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (will vary due to random durations):
// Buffered with random duration: [0, 1, 2, 3, 4, 5]
// Buffered with random duration: [6, 7, 8, 9, 10, 11, 12]
// Buffered with random duration: [13, 14, 15, 16]
// Complete
```

**When to use**:
- When you need dynamic buffer durations that depend on runtime conditions
- For implementing adaptive buffering strategies
- When buffer closing conditions need to be determined by the application state
- For complex event collection scenarios with variable time windows

## concatMap ⭐

**Purpose**: Maps each value from the source Observable to an inner Observable, then flattens the result in a serialized manner, waiting for each inner Observable to complete before moving to the next.

```typescript
import { of, interval } from 'rxjs';
import { concatMap, take, delay, map } from 'rxjs/operators';

// Example 1: Basic concatMap with API calls
console.log('Basic concatMap example:');
const userIds = [1, 2, 3, 4];

// Simulate API calls that take different times to complete
function getUserDetails(id: number) {
  // Simulate different response times
  const delayTime = id * 1000;
  return of({ id, name: `User ${id}`, email: `user${id}@example.com` }).pipe(
    delay(delayTime),
    tap(() => console.log(`Fetched details for user ${id} after ${delayTime}ms`))
  );
}

// Process users sequentially, waiting for each API call to complete
of(...userIds).pipe(
  concatMap(id => getUserDetails(id))
).subscribe(
  user => console.log('Processed user:', user),
  err => console.error(err),
  () => console.log('All users processed')
);

// Output:
// Fetched details for user 1 after 1000ms
// Processed user: {id: 1, name: "User 1", email: "user1@example.com"}
// Fetched details for user 2 after 2000ms
// Processed user: {id: 2, name: "User 2", email: "user2@example.com"}
// Fetched details for user 3 after 3000ms
// Processed user: {id: 3, name: "User 3", email: "user3@example.com"}
// Fetched details for user 4 after 4000ms
// Processed user: {id: 4, name: "User 4", email: "user4@example.com"}
// All users processed

// Example 2: concatMap with events
console.log('concatMap with events example:');
const clicks$ = fromEvent(document, 'click');

clicks$.pipe(
  concatMap(() => interval(200).pipe(
    take(3),
    map(i => `Click result: ${i}`)
  ))
).subscribe(
  result => console.log(result)
);

// Output (after clicking twice in succession):
// Click result: 0
// Click result: 1
// Click result: 2
// (only after the first click sequence completes)
// Click result: 0
// Click result: 1
// Click result: 2
```

**When to use**:
- When order matters and operations need to be processed sequentially
- For API calls that must be executed in a specific order
- When you need to ensure that one operation completes before starting the next
- For implementing step-by-step processes or workflows
- When you want to avoid race conditions in asynchronous operations

## concatMapTo

**Purpose**: Maps each value from the source Observable to the same inner Observable, then flattens the result in a serialized manner.

```typescript
import { fromEvent, interval } from 'rxjs';
import { concatMapTo, take } from 'rxjs/operators';

// Source: click events
const clicks$ = fromEvent(document, 'click');

// Target: always map to the same sequence
const resultSequence$ = interval(500).pipe(
  take(3),
  map(i => `Result ${i}`)
);

// Map each click to the same result sequence
clicks$.pipe(
  concatMapTo(resultSequence$),
  take(6) // Limit the example to 6 emissions
).subscribe(
  result => console.log(result),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (after clicking twice in succession):
// Result 0
// Result 1
// Result 2
// (only after the first sequence completes)
// Result 0
// Result 1
// Result 2
// Complete
```

**When to use**:
- When you always want to map to the same inner Observable
- For implementing a sequence that should run after each trigger event
- When the mapping operation doesn't depend on the source value
- For simplifying code when the inner Observable is constant

## expand

**Purpose**: Recursively projects each source value to an Observable, and merges the results into the output Observable.

```typescript
import { of } from 'rxjs';
import { expand, take } from 'rxjs/operators';

// Start with 1
of(1).pipe(
  // For each value x, return an Observable of x * 2
  expand(x => of(x * 2)),
  take(5) // Limit to 5 values to avoid infinite sequence
).subscribe(
  x => console.log(`Expanded value: ${x}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Expanded value: 1
// Expanded value: 2
// Expanded value: 4
// Expanded value: 8
// Expanded value: 16
// Complete
```

**When to use**:
- For implementing recursive algorithms with Observables
- When you need to generate a sequence where each value depends on the previous one
- For tree traversal or graph exploration
- When implementing breadth-first search patterns

## exhaustMap

**Purpose**: Maps each value from the source Observable to an inner Observable, but ignores source values while the previous inner Observable is still active.

```typescript
import { fromEvent, interval } from 'rxjs';
import { exhaustMap, take, tap } from 'rxjs/operators';

// Source: click events
const clicks$ = fromEvent(document, 'click');

// Map each click to a 3-second countdown, but ignore clicks during the countdown
clicks$.pipe(
  tap(() => console.log('Click detected')),
  exhaustMap(() => interval(1000).pipe(
    take(3),
    tap(i => console.log(`Processing click: ${i + 1} second`))
  ))
).subscribe(
  count => console.log(`Countdown: ${count + 1}`),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (clicking multiple times during the countdown):
// Click detected
// Processing click: 1 second
// Countdown: 1
// Processing click: 2 second
// Countdown: 2
// Processing click: 3 second
// Countdown: 3
// (additional clicks during the countdown are ignored)
// Click detected (only after the previous countdown completes)
// Processing click: 1 second
// Countdown: 1
// ...
```

**When to use**:
- When you want to ignore rapid-fire events while processing is in progress
- For preventing duplicate form submissions
- When implementing "do not disturb" patterns
- For rate-limiting user interactions
- When you want to ensure that an operation completes before starting a new one

## groupBy

**Purpose**: Groups emissions from the source Observable based on a specified key selector function, emitting an Observable for each group.

```typescript
import { from } from 'rxjs';
import { groupBy, mergeMap, toArray } from 'rxjs/operators';

// Sample data: people with different roles
const people = [
  { name: 'Alice', role: 'Developer' },
  { name: 'Bob', role: 'Designer' },
  { name: 'Charlie', role: 'Developer' },
  { name: 'Dave', role: 'Manager' },
  { name: 'Eve', role: 'Designer' },
  { name: 'Frank', role: 'Developer' }
];

// Group people by their role
from(people).pipe(
  // Group by role
  groupBy(person => person.role),
  // For each group, collect all items into an array
  mergeMap(group =>
    group.pipe(
      toArray(),
      map(groupArray => ({
        role: group.key,
        members: groupArray
      }))
    )
  )
).subscribe(
  group => console.log(`${group.role}: ${group.members.map(p => p.name).join(', ')}`),
  err => console.error(err),
  () => console.log('Grouping complete')
);

// Output:
// Developer: Alice, Charlie, Frank
// Designer: Bob, Eve
// Manager: Dave
// Grouping complete
```

**When to use**:
- When you need to categorize or classify emissions based on a property
- For implementing data grouping functionality
- When processing data that needs to be organized by category
- For creating histograms or frequency distributions
- When implementing analytics or reporting features

## map ⭐

**Purpose**: Transforms each value emitted by the source Observable using a projection function.

```typescript
import { from } from 'rxjs';
import { map } from 'rxjs/operators';

// Example 1: Basic value transformation
console.log('Basic map example:');
from([1, 2, 3, 4, 5]).pipe(
  map(value => value * 10)
).subscribe(
  result => console.log(`Transformed value: ${result}`),
  err => console.error(err),
  () => console.log('Transformation complete')
);

// Output:
// Transformed value: 10
// Transformed value: 20
// Transformed value: 30
// Transformed value: 40
// Transformed value: 50
// Transformation complete

// Example 2: Object transformation
console.log('Object transformation example:');
from([
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 35 }
]).pipe(
  map(person => ({
    displayName: person.name.toUpperCase(),
    birthYear: new Date().getFullYear() - person.age,
    canRetireIn: 65 - person.age
  }))
).subscribe(
  result => console.log(result),
  err => console.error(err),
  () => console.log('Object transformation complete')
);

// Output:
// { displayName: 'ALICE', birthYear: 1998, canRetireIn: 40 }
// { displayName: 'BOB', birthYear: 1993, canRetireIn: 35 }
// { displayName: 'CHARLIE', birthYear: 1988, canRetireIn: 30 }
// Object transformation complete
```

**When to use**:
- For basic value transformations (the most common transformation operator)
- When you need to convert data from one format to another
- For extracting specific properties from complex objects
- When preparing data for display or further processing
- As a building block in more complex transformation pipelines

## mapTo

**Purpose**: Maps each emission from the source Observable to a constant value.

```typescript
import { fromEvent, interval } from 'rxjs';
import { mapTo, take } from 'rxjs/operators';

// Example 1: Basic mapTo
console.log('Basic mapTo example:');
interval(1000).pipe(
  mapTo('PING'),
  take(3)
).subscribe(
  result => console.log(result),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// PING
// PING
// PING
// Complete

// Example 2: Event normalization
console.log('Event normalization example:');
const buttonClicks$ = fromEvent(document.getElementById('myButton'), 'click');

buttonClicks$.pipe(
  mapTo({ action: 'BUTTON_CLICKED', timestamp: new Date().toISOString() })
).subscribe(
  event => console.log('Normalized event:', event),
  err => console.error(err)
);

// Output (when button is clicked):
// Normalized event: { action: 'BUTTON_CLICKED', timestamp: '2023-06-20T16:45:23.456Z' }
// Normalized event: { action: 'BUTTON_CLICKED', timestamp: '2023-06-20T16:45:25.789Z' }
// ...
```

**When to use**:
- When you want to replace all emissions with a constant value
- For normalizing events to a standard format
- When implementing simple toggles or flags
- For creating action creators in state management patterns
- When the source value is irrelevant, but the timing of emissions matters

## mergeMap / flatMap ⭐

**Purpose**: Maps each value from the source Observable to an inner Observable, then flattens the result by merging all inner Observables into a single Observable.

```typescript
import { fromEvent, interval, of } from 'rxjs';
import { mergeMap, take, delay } from 'rxjs/operators';

// Example 1: Basic mergeMap with concurrent API calls
console.log('Basic mergeMap example:');
from([1, 2, 3, 4]).pipe(
  mergeMap(id => {
    // Simulate API call with different response times
    console.log(`Making API call for ID: ${id}`);
    return of({ id, name: `Item ${id}` }).pipe(
      delay(1000 / id) // Faster response for higher IDs
    );
  })
).subscribe(
  result => console.log(`Got result for ID ${result.id}: ${result.name}`),
  err => console.error(err),
  () => console.log('All API calls completed')
);

// Output (order may vary due to concurrent execution):
// Making API call for ID: 1
// Making API call for ID: 2
// Making API call for ID: 3
// Making API call for ID: 4
// Got result for ID 4: Item 4
// Got result for ID 3: Item 3
// Got result for ID 2: Item 2
// Got result for ID 1: Item 1
// All API calls completed

// Example 2: Controlling concurrency
console.log('Controlled concurrency example:');
from([1, 2, 3, 4, 5, 6]).pipe(
  // Second parameter limits concurrent inner Observables
  mergeMap(
    id => {
      console.log(`Starting task ${id}`);
      return interval(500).pipe(
        take(3),
        map(x => `Task ${id}: step ${x + 1}`)
      );
    },
    2 // Maximum 2 concurrent inner Observables
  )
).subscribe(
  result => console.log(result),
  err => console.error(err),
  () => console.log('All tasks completed')
);

// Output:
// Starting task 1
// Starting task 2
// Task 1: step 1
// Task 2: step 1
// Task 1: step 2
// Task 2: step 2
// Task 1: step 3
// Task 2: step 3
// Starting task 3
// Starting task 4
// Task 3: step 1
// ...
```

**When to use**:
- When you need to perform concurrent operations
- For making multiple API calls in parallel
- When order of operations doesn't matter
- For high-throughput scenarios where you want to process items as quickly as possible
- When you need to control the level of concurrency in your application

## mergeScan

**Purpose**: Applies an accumulator function to each value from the source Observable where the accumulator returns an Observable, then merges the results into the output Observable.

```typescript
import { fromEvent, of } from 'rxjs';
import { mergeScan, map } from 'rxjs/operators';

// Track mouse drag distance
const mouseDown$ = fromEvent(document, 'mousedown');
const mouseMove$ = fromEvent(document, 'mousemove');
const mouseUp$ = fromEvent(document, 'mouseup');

mouseDown$.pipe(
  // When mouse down, start tracking movement until mouse up
  mergeMap(start => {
    // Extract starting coordinates
    const startX = (start as MouseEvent).clientX;
    const startY = (start as MouseEvent).clientY;

    // Track movement until mouse up
    return mouseMove$.pipe(
      // Use mergeScan to accumulate total distance
      mergeScan(
        (acc, move) => {
          const currentX = (move as MouseEvent).clientX;
          const currentY = (move as MouseEvent).clientY;

          // Calculate distance from last position
          const deltaX = currentX - acc.lastX;
          const deltaY = currentY - acc.lastY;
          const stepDistance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);

          // Return new accumulated state
          return of({
            totalDistance: acc.totalDistance + stepDistance,
            lastX: currentX,
            lastY: currentY
          });
        },
        { totalDistance: 0, lastX: startX, lastY: startY }
      ),
      // Stop tracking when mouse up
      takeUntil(mouseUp$)
    );
  })
).subscribe(
  state => console.log(`Total drag distance: ${state.totalDistance.toFixed(2)}px`),
  err => console.error(err)
);

// Output (while dragging mouse):
// Total drag distance: 10.54px
// Total drag distance: 25.32px
// Total drag distance: 42.18px
// ...
```

**When to use**:
- For accumulating values over time with asynchronous operations
- When implementing complex state machines
- For tracking cumulative changes that involve async operations
- When you need both the accumulation behavior of `scan` and the flattening behavior of `mergeMap`

## partition

**Purpose**: Splits the source Observable into two Observables: one with values that satisfy a predicate function and one with values that don't.

```typescript
import { from, partition } from 'rxjs';

// Sample data
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Partition numbers into evens and odds
const [evens$, odds$] = partition(
  from(numbers),
  value => value % 2 === 0
);

// Subscribe to even numbers
evens$.subscribe(
  value => console.log(`Even number: ${value}`),
  err => console.error(err),
  () => console.log('Even numbers complete')
);

// Subscribe to odd numbers
odds$.subscribe(
  value => console.log(`Odd number: ${value}`),
  err => console.error(err),
  () => console.log('Odd numbers complete')
);

// Output:
// Odd number: 1
// Even number: 2
// Odd number: 3
// Even number: 4
// Odd number: 5
// Even number: 6
// Odd number: 7
// Even number: 8
// Odd number: 9
// Even number: 10
// Even numbers complete
// Odd numbers complete
```

**When to use**:
- When you need to split a stream into two based on a condition
- For implementing filtering with different handling for matched and unmatched items
- When you need to process different categories of data in separate streams
- For error handling strategies where valid and invalid data need different processing

## pluck

**Purpose**: Extracts a specified nested property from each emitted object.

```typescript
import { from } from 'rxjs';
import { pluck } from 'rxjs/operators';

// Sample data with nested properties
const users = [
  { id: 1, name: 'Alice', profile: { age: 25, role: 'Developer' } },
  { id: 2, name: 'Bob', profile: { age: 30, role: 'Designer' } },
  { id: 3, name: 'Charlie', profile: { age: 35, role: 'Manager' } }
];

// Example 1: Extract simple property
console.log('Simple property extraction:');
from(users).pipe(
  pluck('name')
).subscribe(
  name => console.log(`Name: ${name}`),
  err => console.error(err),
  () => console.log('Simple extraction complete')
);

// Output:
// Name: Alice
// Name: Bob
// Name: Charlie
// Simple extraction complete

// Example 2: Extract nested property
console.log('Nested property extraction:');
from(users).pipe(
  pluck('profile', 'role')
).subscribe(
  role => console.log(`Role: ${role}`),
  err => console.error(err),
  () => console.log('Nested extraction complete')
);

// Output:
// Role: Developer
// Role: Designer
// Role: Manager
// Nested extraction complete
```

**When to use**:
- For extracting specific properties from objects in a stream
- When you need a simpler alternative to `map` for property extraction
- For accessing nested properties in a concise way
- When working with event objects where you only need specific properties

## reduce

**Purpose**: Applies an accumulator function over the source Observable, and emits the final accumulated value when the source completes.

```typescript
import { from } from 'rxjs';
import { reduce } from 'rxjs/operators';

// Example 1: Sum of numbers
console.log('Sum example:');
from([1, 2, 3, 4, 5]).pipe(
  reduce((acc, value) => acc + value, 0)
).subscribe(
  sum => console.log(`Sum: ${sum}`),
  err => console.error(err),
  () => console.log('Reduction complete')
);

// Output:
// Sum: 15
// Reduction complete

// Example 2: Building a complex result
console.log('Complex reduction example:');
from([
  { name: 'Alice', sales: 120, department: 'Electronics' },
  { name: 'Bob', sales: 90, department: 'Home' },
  { name: 'Charlie', sales: 150, department: 'Electronics' },
  { name: 'Dave', sales: 80, department: 'Home' },
  { name: 'Eve', sales: 200, department: 'Electronics' }
]).pipe(
  reduce((result, employee) => {
    // Group by department
    if (!result[employee.department]) {
      result[employee.department] = {
        totalSales: 0,
        employees: []
      };
    }

    // Update department data
    result[employee.department].totalSales += employee.sales;
    result[employee.department].employees.push(employee.name);

    return result;
  }, {} as Record<string, { totalSales: number, employees: string[] }>)
).subscribe(
  result => {
    console.log('Sales report:');
    Object.entries(result).forEach(([dept, data]) => {
      console.log(`${dept}: ${data.totalSales} units sold by ${data.employees.join(', ')}`);
    });
  },
  err => console.error(err),
  () => console.log('Report complete')
);

// Output:
// Sales report:
// Electronics: 470 units sold by Alice, Charlie, Eve
// Home: 170 units sold by Bob, Dave
// Report complete
```

**When to use**:
- When you need a single result that depends on all source values
- For calculating totals, averages, or other aggregate values
- When building complex data structures from a stream of inputs
- For final processing of a finite Observable
- When you only care about the final result, not intermediate values

## scan ⭐

**Purpose**: Similar to `reduce`, but emits the accumulated value after each source emission instead of just the final value.

```typescript
import { from, fromEvent } from 'rxjs';
import { scan, map } from 'rxjs/operators';

// Example 1: Running total
console.log('Running total example:');
from([1, 2, 3, 4, 5]).pipe(
  scan((acc, value) => acc + value, 0)
).subscribe(
  total => console.log(`Running total: ${total}`),
  err => console.error(err),
  () => console.log('Scan complete')
);

// Output:
// Running total: 1
// Running total: 3
// Running total: 6
// Running total: 10
// Running total: 15
// Scan complete

// Example 2: State management
console.log('State management example:');

// Define actions
const increment = { type: 'INCREMENT' };
const decrement = { type: 'DECREMENT' };
const reset = { type: 'RESET' };

// Create action stream
const actions$ = fromEvent(document, 'click').pipe(
  map((event: MouseEvent) => {
    if (event.altKey) return reset;
    return event.shiftKey ? decrement : increment;
  })
);

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'RESET':
      return { ...state, count: 0 };
    default:
      return state;
  }
}

// Initial state
const initialState = { count: 0 };

// Create state stream
const state$ = actions$.pipe(
  scan(counterReducer, initialState)
);

// Subscribe to state changes
state$.subscribe(
  state => console.log(`Counter: ${state.count}`),
  err => console.error(err)
);

// Output (after clicking):
// Counter: 1
// Counter: 2
// Counter: 3
// (shift+click) Counter: 2
// (alt+click) Counter: 0
```

**When to use**:
- For maintaining state that updates with each emission
- When implementing state management patterns like Redux
- For calculating running totals or moving averages
- When you need to track the history or progression of values
- For creating accumulators that emit intermediate results

## switchMap ⭐

**Purpose**: Maps each value from the source Observable to an inner Observable, then flattens the result by replacing the previous inner Observable with the new one when a new source value arrives.

```typescript
import { fromEvent, interval, of } from 'rxjs';
import { switchMap, take, delay, tap } from 'rxjs/operators';

// Example 1: Basic switchMap with search
console.log('Basic switchMap example:');

// Simulate a search input field
const searchInput$ = fromEvent(document.getElementById('searchInput'), 'input');

// Use switchMap to handle search requests
searchInput$.pipe(
  tap(() => console.log('Search input changed')),
  // Extract the search term
  map((event: InputEvent) => (event.target as HTMLInputElement).value),
  // Only proceed if search term is at least 3 characters
  filter(term => term.length >= 3),
  // Debounce to avoid too many requests
  debounceTime(300),
  // Use switchMap to cancel previous search and start a new one
  switchMap(term => {
    console.log(`Searching for: ${term}`);
    // Simulate API call
    return of(`Search results for: ${term}`).pipe(
      delay(1000), // Simulate network delay
      tap(() => console.log(`Received results for: ${term}`))
    );
  })
).subscribe(
  results => console.log(results),
  err => console.error(err)
);

// Output (when typing "react" then quickly changing to "reactive"):
// Search input changed
// Search input changed
// Search input changed
// Searching for: rea
// Search input changed
// Search input changed
// Search input changed
// Searching for: reactive
// Received results for: reactive
// Search results for: reactive
// (Note: results for "rea" were cancelled)

// Example 2: HTTP requests with switchMap
console.log('HTTP requests with switchMap example:');

// Simulate user ID selection
const userIds$ = from([1, 2, 3]);

// Function to simulate HTTP request
function getUserDetails(id: number) {
  console.log(`Fetching details for user ${id}...`);
  return of({ id, name: `User ${id}`, email: `user${id}@example.com` }).pipe(
    delay(id * 1000), // Different delay times
    tap(() => console.log(`Completed fetch for user ${id}`))
  );
}

// Use switchMap to always get the latest user
userIds$.pipe(
  // Delay between emissions to see the effect
  concatMap(id => of(id).pipe(delay(500))),
  // Use switchMap to switch to new user and cancel previous request
  switchMap(id => getUserDetails(id))
).subscribe(
  user => console.log('Received user:', user),
  err => console.error(err),
  () => console.log('User sequence complete')
);

// Output:
// Fetching details for user 1...
// Fetching details for user 2... (previous request for user 1 is cancelled)
// Fetching details for user 3... (previous request for user 2 is cancelled)
// Completed fetch for user 3
// Received user: { id: 3, name: 'User 3', email: 'user3@example.com' }
// User sequence complete
```

**When to use**:
- When you need to cancel previous operations when a new one starts
- For search functionality where only the latest search matters
- When handling user input that supersedes previous inputs
- For HTTP requests where only the response to the most recent request is relevant
- When implementing "latest value wins" scenarios

## switchMapTo

**Purpose**: Maps each value from the source Observable to the same inner Observable, then flattens the result by replacing the previous inner Observable with the new one when a new source value arrives.

```typescript
import { fromEvent, interval } from 'rxjs';
import { switchMapTo, take, tap } from 'rxjs/operators';

// Example: Reset a timer on each click
console.log('switchMapTo example:');

// Source: click events
const clicks$ = fromEvent(document.getElementById('resetButton'), 'click');

// Target: always the same 5-second countdown
const countdown$ = interval(1000).pipe(
  take(5),
  map(i => 4 - i),
  tap(value => console.log(`Countdown: ${value}`))
);

// Reset the countdown on each click
clicks$.pipe(
  tap(() => console.log('Button clicked, resetting countdown')),
  switchMapTo(countdown$)
).subscribe(
  value => console.log(`Displaying: ${value}`),
  err => console.error(err)
);

// Output (when clicking during countdown):
// Button clicked, resetting countdown
// Countdown: 4
// Displaying: 4
// Countdown: 3
// Displaying: 3
// Button clicked, resetting countdown (clicked again)
// Countdown: 4
// Displaying: 4
// Countdown: 3
// Displaying: 3
// ...
```

**When to use**:
- When you need to switch to the same inner Observable for each source emission
- For resetting or restarting processes when an event occurs
- When the inner Observable doesn't depend on the source value
- For simplifying code when the inner Observable is constant

## toArray

**Purpose**: Collects all values from the source Observable and emits them as an array when the source completes.

```typescript
import { interval, of } from 'rxjs';
import { toArray, take, map } from 'rxjs/operators';

// Example 1: Basic toArray
console.log('Basic toArray example:');
of(1, 2, 3, 4, 5).pipe(
  toArray()
).subscribe(
  array => console.log('Collected array:', array),
  err => console.error(err),
  () => console.log('Collection complete')
);

// Output:
// Collected array: [1, 2, 3, 4, 5]
// Collection complete

// Example 2: Collecting async values
console.log('Async collection example:');
interval(500).pipe(
  take(4),
  map(i => ({ id: i, value: `Item ${i}` })),
  toArray()
).subscribe(
  array => console.log('Collected objects:', array),
  err => console.error(err),
  () => console.log('Async collection complete')
);

// Output (after 2 seconds):
// Collected objects: [
//   { id: 0, value: 'Item 0' },
//   { id: 1, value: 'Item 1' },
//   { id: 2, value: 'Item 2' },
//   { id: 3, value: 'Item 3' }
// ]
// Async collection complete
```

**When to use**:
- When you need to collect all emissions before processing them
- For converting a stream of values into a single array
- When implementing batch processing
- For gathering all results from a finite Observable
- When you need to perform operations on the complete set of values

## window

**Purpose**: Similar to `buffer`, but emits nested Observables instead of arrays.

```typescript
import { interval, fromEvent } from 'rxjs';
import { window, mergeMap, take, toArray } from 'rxjs/operators';

// Create a source Observable that emits a value every 500ms
const source$ = interval(500);

// Use mouse clicks as the window boundary
const clicks$ = fromEvent(document, 'click');

// Window values from source until a click occurs
source$.pipe(
  window(clicks$),
  take(3), // Take only 3 window emissions for the example
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('Window values:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (after clicking three times, with varying intervals between clicks):
// Window values: [0, 1, 2, 3]
// Window values: [4, 5, 6]
// Window values: [7, 8, 9, 10, 11]
// Complete
```

**When to use**:
- When you need more control over the collected values than `buffer` provides
- For implementing complex windowing strategies
- When you need to apply operators to each window of values
- For advanced stream processing scenarios

## windowCount

**Purpose**: Similar to `bufferCount`, but emits nested Observables instead of arrays.

```typescript
import { interval } from 'rxjs';
import { windowCount, mergeMap, take, toArray } from 'rxjs/operators';

// Example 1: Basic windowCount
console.log('Basic windowCount example:');
interval(500).pipe(
  take(10),
  windowCount(3), // Collect 3 values per window
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('Window of 3:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Window of 3: [0, 1, 2]
// Window of 3: [3, 4, 5]
// Window of 3: [6, 7, 8]
// Window of 3: [9]
// Complete

// Example 2: Overlapping windows
console.log('Overlapping windowCount example:');
interval(500).pipe(
  take(10),
  windowCount(3, 1), // Window size 3, start new window every 1 value
  mergeMap(window$ =>
    window$.pipe(
      toArray(),
      map(arr => arr.join(', '))
    )
  ),
  take(5) // Take only 5 window emissions for the example
).subscribe(
  windowValues => console.log('Overlapping window:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Overlapping window: 0, 1, 2
// Overlapping window: 1, 2, 3
// Overlapping window: 2, 3, 4
// Overlapping window: 3, 4, 5
// Overlapping window: 4, 5, 6
// Complete
```

**When to use**:
- When you need more control over count-based windows than `bufferCount` provides
- For implementing sliding window algorithms with fine-grained control
- When you need to apply operators to each window of values
- For complex data processing with fixed-size windows

## windowTime

**Purpose**: Similar to `bufferTime`, but emits nested Observables instead of arrays.

```typescript
import { interval } from 'rxjs';
import { windowTime, mergeMap, take, toArray } from 'rxjs/operators';

// Example 1: Basic windowTime
console.log('Basic windowTime example:');
interval(300).pipe(
  windowTime(1000), // Collect values for 1 second
  take(3), // Take only 3 window emissions for the example
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('1-second window:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// 1-second window: [0, 1, 2]
// 1-second window: [3, 4, 5]
// 1-second window: [6, 7, 8]
// Complete

// Example 2: Overlapping time windows
console.log('Overlapping windowTime example:');
interval(300).pipe(
  windowTime(1000, 500), // 1-second window, new window every 500ms
  take(3), // Take only 3 window emissions for the example
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('Overlapping time window:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Overlapping time window: [0, 1, 2]
// Overlapping time window: [1, 2, 3, 4]
// Overlapping time window: [3, 4, 5, 6]
// Complete
```

**When to use**:
- When you need more control over time-based windows than `bufferTime` provides
- For implementing complex time-window operations
- When you need to apply operators to each window of values
- For advanced time-based stream processing

## windowToggle

**Purpose**: Similar to `bufferToggle`, but emits nested Observables instead of arrays.

```typescript
import { interval, timer } from 'rxjs';
import { windowToggle, mergeMap, take, toArray } from 'rxjs/operators';

// Source emits every 100ms
const source$ = interval(100);

// Opening observable: emit every 500ms
const openings$ = interval(500);

// Closing observable factory: emit after 300ms
const closingSelector = () => timer(300);

// Window values when openings$ emits, and collect until closingSelector emits
source$.pipe(
  windowToggle(openings$, closingSelector),
  take(3), // Take only 3 window emissions for the example
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('Windowed values:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output:
// Windowed values: [5, 6, 7]
// Windowed values: [10, 11, 12]
// Windowed values: [15, 16, 17]
// Complete
```

**When to use**:
- When you need more control over start/stop windows than `bufferToggle` provides
- For implementing complex recording or sampling functionality
- When you need to apply operators to each window of values
- For advanced scenarios with dynamic start and end conditions

## windowWhen

**Purpose**: Similar to `bufferWhen`, but emits nested Observables instead of arrays.

```typescript
import { interval } from 'rxjs';
import { windowWhen, mergeMap, take, toArray } from 'rxjs/operators';

// Source emits every 100ms
const source$ = interval(100);

// Window closing factory: create random-duration windows
const closingSelector = () => {
  // Random window duration between 500ms and 1500ms
  const randomDuration = Math.floor(Math.random() * 1000) + 500;
  return interval(randomDuration).pipe(take(1));
};

// Window values until closingSelector emits
source$.pipe(
  windowWhen(closingSelector),
  take(3), // Take only 3 window emissions for the example
  mergeMap(window$ => window$.pipe(toArray())), // Convert each window to an array
).subscribe(
  windowValues => console.log('Windowed with random duration:', windowValues),
  err => console.error(err),
  () => console.log('Complete')
);

// Output (will vary due to random durations):
// Windowed with random duration: [0, 1, 2, 3, 4, 5]
// Windowed with random duration: [6, 7, 8, 9, 10, 11, 12]
// Windowed with random duration: [13, 14, 15, 16]
// Complete
```

**When to use**:
- When you need more control over dynamic windows than `bufferWhen` provides
- For implementing adaptive windowing strategies
- When you need to apply operators to each window of values
- For complex event collection scenarios with variable time windows

## Comparison of Key Operators

| Operator | Buffering Strategy | Order Preservation | Concurrency | Use Case |
|----------|-------------------|-------------------|-------------|----------|
| `buffer` | Event-based | Yes | N/A | Collect values between events |
| `bufferCount` | Count-based | Yes | N/A | Fixed-size batches |
| `bufferTime` ⭐ | Time-based | Yes | N/A | Time-window batching |
| `bufferToggle` | Start/stop events | Yes | N/A | Complex time windows |
| `bufferWhen` | Dynamic closing | Yes | N/A | Adaptive buffering |
| `concatMap` ⭐ | N/A | Yes | Sequential | Ordered processing |
| `concatMapTo` | N/A | Yes | Sequential | Same sequence for each value |
| `expand` | N/A | Breadth-first | Concurrent | Recursive algorithms |
| `exhaustMap` | N/A | First only | Blocking | Ignore during processing |
| `groupBy` | N/A | Depends on source | N/A | Categorize by key |
| `map` ⭐ | N/A | Yes | N/A | Basic value transformation |
| `mapTo` | N/A | Yes | N/A | Replace with constant value |
| `mergeMap` ⭐ | N/A | No | Concurrent | Parallel processing |
| `mergeScan` | N/A | No | Concurrent | Stateful async accumulation |
| `partition` | N/A | Yes | N/A | Split stream by condition |
| `pluck` | N/A | Yes | N/A | Extract object properties |
| `reduce` | N/A | N/A | N/A | Final accumulation |
| `scan` ⭐ | N/A | Yes | N/A | Running accumulation |
| `switchMap` ⭐ | N/A | No | Switching | Latest value wins |
| `switchMapTo` | N/A | No | Switching | Reset to same Observable |
| `toArray` | N/A | Yes | N/A | Collect all as array |
| `window` | Event-based | Yes | N/A | Observable windows between events |
| `windowCount` | Count-based | Yes | N/A | Observable windows of fixed size |
| `windowTime` | Time-based | Yes | N/A | Observable windows by time |
| `windowToggle` | Start/stop events | Yes | N/A | Observable windows with complex conditions |
| `windowWhen` | Dynamic closing | Yes | N/A | Observable windows with dynamic duration |

## Practical Examples

### Real-time Data Aggregation with bufferTime

```typescript
import { interval } from 'rxjs';
import { bufferTime, map } from 'rxjs/operators';

// Simulate sensor readings coming in at high frequency (every 100ms)
const sensorReadings$ = interval(100).pipe(
  map(() => ({
    temperature: 20 + Math.random() * 5,
    humidity: 50 + Math.random() * 10,
    timestamp: new Date().toISOString()
  }))
);

// Aggregate readings over 1-second windows
sensorReadings$.pipe(
  bufferTime(1000),
  map(readings => {
    // Skip empty buffers
    if (readings.length === 0) return null;

    // Calculate averages
    const avgTemperature = readings.reduce((sum, reading) => sum + reading.temperature, 0) / readings.length;
    const avgHumidity = readings.reduce((sum, reading) => sum + reading.humidity, 0) / readings.length;

    return {
      avgTemperature: avgTemperature.toFixed(2),
      avgHumidity: avgHumidity.toFixed(2),
      sampleCount: readings.length,
      timeWindow: {
        start: readings[0].timestamp,
        end: readings[readings.length - 1].timestamp
      }
    };
  }),
  // Filter out null results from empty buffers
  filter(result => result !== null)
).subscribe(
  aggregatedData => console.log('Aggregated sensor data:', aggregatedData)
);

// Output:
// Aggregated sensor data: {
//   avgTemperature: "22.34",
//   avgHumidity: "54.21",
//   sampleCount: 10,
//   timeWindow: {
//     start: "2023-06-20T15:30:00.123Z",
//     end: "2023-06-20T15:30:00.923Z"
//   }
// }
// ... (continues every second)
```

### Sequential API Calls with concatMap

```typescript
import { from, of } from 'rxjs';
import { concatMap, catchError, tap } from 'rxjs/operators';

// Simulate a database with user IDs and their order IDs
const userDatabase = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
  { id: 3, name: 'Charlie' }
];

const orderDatabase = {
  1: [{ id: 101, product: 'Laptop', price: 1200 }, { id: 102, product: 'Phone', price: 800 }],
  2: [{ id: 201, product: 'Tablet', price: 500 }],
  3: [] // No orders
};

// Simulate API calls
function getUser(id) {
  return of(userDatabase.find(user => user.id === id)).pipe(
    delay(500), // Simulate network delay
    tap(user => console.log(`Fetched user: ${user?.name || 'Unknown'}`)),
    catchError(err => {
      console.error(`Error fetching user ${id}:`, err);
      return of(null);
    })
  );
}

function getUserOrders(userId) {
  return of(orderDatabase[userId] || []).pipe(
    delay(500), // Simulate network delay
    tap(orders => console.log(`Fetched ${orders.length} orders for user ${userId}`)),
    catchError(err => {
      console.error(`Error fetching orders for user ${userId}:`, err);
      return of([]);
    })
  );
}

// Process users sequentially, getting their orders
from([1, 2, 3, 4]).pipe(
  concatMap(userId =>
    getUser(userId).pipe(
      concatMap(user => {
        if (!user) return of({ userId, error: 'User not found' });

        return getUserOrders(user.id).pipe(
          map(orders => ({
            user,
            orders,
            totalSpent: orders.reduce((sum, order) => sum + order.price, 0)
          }))
        );
      })
    )
  )
).subscribe(
  result => console.log('Processed result:', result),
  err => console.error('Error in processing:', err),
  () => console.log('All users processed')
);

// Output:
// Fetched user: Alice
// Fetched 2 orders for user 1
// Processed result: {
//   user: { id: 1, name: 'Alice' },
//   orders: [
//     { id: 101, product: 'Laptop', price: 1200 },
//     { id: 102, product: 'Phone', price: 800 }
//   ],
//   totalSpent: 2000
// }
// Fetched user: Bob
// Fetched 1 orders for user 2
// Processed result: {
//   user: { id: 2, name: 'Bob' },
//   orders: [{ id: 201, product: 'Tablet', price: 500 }],
//   totalSpent: 500
// }
// Fetched user: Charlie
// Fetched 0 orders for user 3
// Processed result: {
//   user: { id: 3, name: 'Charlie' },
//   orders: [],
//   totalSpent: 0
// }
// Fetched user: Unknown
// Processed result: { userId: 4, error: 'User not found' }
// All users processed
```

## Conclusion

Transformation operators are the workhorses of RxJS, allowing you to reshape, combine, and process data in powerful ways. By understanding these operators, you can build sophisticated data processing pipelines that handle complex asynchronous scenarios with clean, declarative code:

- Use `buffer` and its variants (`bufferCount`, `bufferTime` ⭐, `bufferToggle`, `bufferWhen`) for collecting and processing values in batches
- Use `concatMap` ⭐ for sequential processing where order matters
- Use `concatMapTo` when you need to map each source value to the same inner Observable
- Use `expand` for recursive algorithms and tree-like data structures
- Use `exhaustMap` when you want to ignore new events while processing is in progress
- Use `groupBy` for categorizing emissions into separate Observables based on a key
- Use `map` ⭐ for basic value transformations (most common transformation operator)
- Use `mapTo` when you need to replace emissions with a constant value
- Use `mergeMap` ⭐ for concurrent processing where order doesn't matter
- Use `mergeScan` for stateful accumulations that involve asynchronous operations
- Use `partition` to split a stream into two based on a condition
- Use `pluck` for extracting specific properties from objects in a stream
- Use `reduce` for calculating a single result from all emissions
- Use `scan` ⭐ for maintaining state that updates with each emission
- Use `switchMap` ⭐ for cancelling previous operations when a new one starts
- Use `switchMapTo` for resetting to the same inner Observable when a new value arrives
- Use `toArray` for collecting all emissions into a single array
- Use `window` and its variants (`windowCount`, `windowTime`, `windowToggle`, `windowWhen`) for creating Observable windows instead of arrays

These transformation operators become even more powerful when combined with other RxJS operators to create comprehensive data processing pipelines tailored to your application's needs.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
- [RxJS Transformation Operators](https://www.learnrxjs.io/learn-rxjs/operators/transformation)
