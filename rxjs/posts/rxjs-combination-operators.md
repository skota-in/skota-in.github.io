# RxJS Combination Operators: Examples and Use Cases

*Published on August 10, 2023*

## Introduction

RxJS provides powerful operators for combining multiple Observables in different ways. These combination operators are essential tools for managing complex asynchronous data flows in modern web applications. This post explores the most commonly used combination operators with practical examples.

## Table of Contents

1. [combineLatest](#combinelatest)
2. [concat](#concat)
3. [merge](#merge)
4. [startWith](#startwith)
5. [withLatestFrom](#withlatestfrom)
6. [Comparison of Key Operators](#comparison-of-key-operators)

## combineLatest

**Purpose**: Combines multiple Observables to create an Observable that emits the latest values from each source whenever any of them emits.

```typescript
import { combineLatest, timer } from 'rxjs';
import { map } from 'rxjs/operators';

// Three Observables with different emission intervals
const positions$ = timer(0, 1000).pipe(map(x => `x-position: ${x * 10}px`));
const colors$ = timer(0, 2000).pipe(map(x => ['red', 'green', 'blue'][x % 3]));
const sizes$ = timer(0, 3000).pipe(map(x => `${10 + x * 5}px`));

// Combine latest values from each source
combineLatest([positions$, colors$, sizes$]).subscribe(
  ([position, color, size]) => {
    console.log(`Update element with ${position}, color: ${color}, size: ${size}`);
    // Would update UI element with these properties
  }
);

// Output example:
// "Update element with x-position: 0px, color: red, size: 10px"
// "Update element with x-position: 10px, color: red, size: 10px"
// "Update element with x-position: 10px, color: green, size: 10px"
// "Update element with x-position: 20px, color: green, size: 10px"
// "Update element with x-position: 20px, color: green, size: 15px"
// ...and so on
```

**When to use**: 
- When you need to react to changes in any of multiple related data streams
- For combining the latest state from multiple sources
- For UI components that depend on multiple changing inputs
- When all inputs need to have emitted at least once before processing

## concat

**Purpose**: Joins multiple Observables sequentially - subscribes to each one only after the previous one completes.

```typescript
import { concat, of } from 'rxjs';
import { delay, tap } from 'rxjs/operators';

// Create a sequence of operations
const saveData$ = of('Data saved successfully').pipe(
  delay(1000),
  tap(() => console.log('Saving data...'))
);

const updateUI$ = of('UI updated with saved data').pipe(
  delay(1000),
  tap(() => console.log('Updating UI...'))
);

const notifyUser$ = of('User notification sent').pipe(
  delay(1000),
  tap(() => console.log('Sending notification...'))
);

// Execute operations in sequence
concat(saveData$, updateUI$, notifyUser$).subscribe(
  message => console.log(`Completed: ${message}`)
);

// Output:
// "Saving data..."
// "Completed: Data saved successfully"
// "Updating UI..."
// "Completed: UI updated with saved data"
// "Sending notification..."
// "Completed: User notification sent"
```

**When to use**:
- For sequential operations where order matters
- When you need to ensure one Observable completes before starting another
- For step-by-step processes or workflows
- For chaining dependent async operations

## merge

**Purpose**: Combines multiple Observables into a single Observable that emits all values from all source Observables as they occur.

```typescript
import { merge, fromEvent, interval } from 'rxjs';
import { map, take } from 'rxjs/operators';

// Handle multiple event sources as a single stream
const clicks$ = fromEvent(document, 'click').pipe(
  map(() => 'Document clicked'),
  take(3)
);

const keyPresses$ = fromEvent(document, 'keydown').pipe(
  map((event: KeyboardEvent) => `Key pressed: ${event.key}`),
  take(3)
);

const timer$ = interval(2000).pipe(
  map(val => `Timer tick: ${val}`),
  take(3)
);

// Merge all events into a single stream
merge(clicks$, keyPresses$, timer$).subscribe(
  action => console.log(`Action received: ${action}`)
);

// Possible output (depends on user interaction):
// "Action received: Timer tick: 0"
// "Action received: Document clicked"
// "Action received: Key pressed: a"
// "Action received: Timer tick: 1"
// "Action received: Key pressed: b"
// "Action received: Document clicked"
// "Action received: Timer tick: 2"
// "Action received: Key pressed: Enter"
// "Action received: Document clicked"
```

**When to use**:
- When you need to handle multiple independent event sources as a single stream
- For combining unrelated user interactions
- When you care about all events from multiple sources but don't need to combine their values
- For implementing "or" logic between multiple streams

## startWith

**Purpose**: Emits specified values before beginning to emit values from the source Observable.

```typescript
import { interval } from 'rxjs';
import { startWith, scan, map, take } from 'rxjs/operators';

// Create a counter with initial state
const counter$ = interval(1000).pipe(
  map(() => 1),  // Each tick increments by 1
  scan((acc, curr) => acc + curr, 0),
  take(5)
);

// Add initial state with startWith
counter$.pipe(
  startWith('Counter starting...'),
  map(value => typeof value === 'number' ? `Count: ${value}` : value)
).subscribe(
  value => console.log(value)
);

// Output:
// "Counter starting..."
// "Count: 0"
// "Count: 1"
// "Count: 2"
// "Count: 3"
// "Count: 4"
```

**When to use**:
- For providing default or initial values
- When you need to ensure subscribers always receive an immediate value
- For initializing state in state management patterns
- To display loading states or default data before real data arrives

## withLatestFrom

**Purpose**: Combines the source Observable with other Observables, but only emits when the source Observable emits.

```typescript
import { fromEvent, interval } from 'rxjs';
import { withLatestFrom, map } from 'rxjs/operators';

// Track mouse position with current app state
const mouseClicks$ = fromEvent(document, 'click');

// These represent different parts of application state
const appState$ = interval(2000).pipe(
  map(x => ['initializing', 'loading', 'ready'][x % 3])
);

const userData$ = interval(3000).pipe(
  map(x => `user-${x + 1}`)
);

// Capture app state and user data only when mouse is clicked
mouseClicks$.pipe(
  withLatestFrom(appState$, userData$),
  map(([event, appState, user]) => {
    const mouseX = (event as MouseEvent).clientX;
    const mouseY = (event as MouseEvent).clientY;
    return {
      action: 'click',
      position: { x: mouseX, y: mouseY },
      appState,
      user
    };
  })
).subscribe(
  data => console.log('Captured event with context:', data)
);

// Example output when clicked:
// Captured event with context: {
//   action: 'click',
//   position: { x: 243, y: 124 },
//   appState: 'ready',
//   user: 'user-2'
// }
```

**When to use**:
- When you have a primary Observable (like user actions) that needs context from other streams
- When you want to sample secondary Observables only when a primary event occurs
- For capturing the state of the application when specific events happen
- For enriching primary events with additional data

## Comparison of Key Operators

| Operator | Emission Trigger | Value Combination | Use Case |
|----------|------------------|-------------------|----------|
| `combineLatest` | Any source emits | Latest from all sources | Coordinated updates from multiple changing inputs |
| `concat` | Previous source completes | Values in sequence | Step-by-step processes, sequential operations |
| `merge` | Any source emits | Individual values as they occur | Handling multiple independent event streams |
| `startWith` | Immediate, then source | Initial values, then source values | Providing default values, initializing state |
| `withLatestFrom` | Only primary source | Primary with latest from others | Enriching primary events with context |

## Conclusion

Understanding these combination operators is essential for effective reactive programming with RxJS. Each operator serves a specific purpose and choosing the right one depends on your specific use case:

- Use `combineLatest` when you need to react to changes in any of multiple related streams
- Use `concat` for sequential operations where order matters
- Use `merge` when handling multiple independent event sources
- Use `startWith` to provide initial values before a stream starts emitting
- Use `withLatestFrom` when you have a primary stream that needs context from other streams

By mastering these operators, you'll be able to handle complex asynchronous scenarios in your applications with clean, declarative code.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxMarbles: Interactive diagrams](https://rxmarbles.com/)
