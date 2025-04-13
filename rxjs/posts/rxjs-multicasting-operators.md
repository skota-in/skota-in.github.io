# RxJS Multicasting Operators: Examples and Use Cases

*Published on May 15, 2023*

## Introduction

RxJS multicasting operators allow you to share a single subscription to an Observable among multiple subscribers. This is crucial for performance optimization and maintaining consistency across your application. Without multicasting, each subscriber would create a separate execution of the Observable, potentially causing duplicate side effects and wasting resources. This post explores the most commonly used multicasting operators with practical examples.

## Table of Contents

1. [multicast](#multicast)
2. [publish](#publish)
3. [share ⭐](#share)
4. [shareReplay ⭐](#sharereplay)
5. [Comparison of Key Operators](#comparison-of-key-operators)

## multicast

**Purpose**: Shares a source Observable by using a Subject as a proxy to which multiple Observers can subscribe.

```typescript
import { interval, Subject } from 'rxjs';
import { multicast, take, tap } from 'rxjs/operators';

// Create a source Observable with a side effect
const source$ = interval(1000).pipe(
  take(3),
  tap(value => console.log(`Source emitted: ${value}`))
);

// Create a Subject to use as the multicast destination
const subject = new Subject<number>();

// Create a multicasted Observable
const multicasted$ = source$.pipe(
  multicast(subject)
);

// First subscription
multicasted$.subscribe(value => console.log(`Observer 1: ${value}`));

// Second subscription (after a delay to demonstrate the shared execution)
setTimeout(() => {
  multicasted$.subscribe(value => console.log(`Observer 2: ${value}`));
}, 1500);

// Connect the multicasted Observable to the source
const subscription = multicasted$.connect();

// Later, unsubscribe to clean up
setTimeout(() => {
  subscription.unsubscribe();
  console.log('Unsubscribed from source');
}, 5000);

// Output:
// Source emitted: 0
// Observer 1: 0
// Source emitted: 1
// Observer 1: 1
// Observer 2: 1
// Source emitted: 2
// Observer 1: 2
// Observer 2: 2
// Unsubscribed from source
```

**When to use**:
- When you need fine-grained control over the Subject used for multicasting
- When you need to manually control when to connect to and disconnect from the source
- For advanced multicasting scenarios where you need to manage the lifecycle of the shared subscription
- When you want to use a specific Subject type (e.g., BehaviorSubject, ReplaySubject)

## publish

**Purpose**: A specialized version of `multicast` that uses a regular Subject under the hood.

```typescript
import { interval } from 'rxjs';
import { publish, tap, take } from 'rxjs/operators';

// Create a source Observable with a side effect
const source$ = interval(1000).pipe(
  take(3),
  tap(value => console.log(`Source emitted: ${value}`))
);

// Create a published (multicasted) Observable
const published$ = source$.pipe(
  publish()
);

// First subscription
published$.subscribe(value => console.log(`Observer 1: ${value}`));

// Second subscription
setTimeout(() => {
  published$.subscribe(value => console.log(`Observer 2: ${value}`));
}, 1500);

// Connect to the source
const subscription = published$.connect();

// Later, unsubscribe to clean up
setTimeout(() => {
  subscription.unsubscribe();
  console.log('Unsubscribed from source');
}, 5000);

// Output:
// Source emitted: 0
// Observer 1: 0
// Source emitted: 1
// Observer 1: 1
// Observer 2: 1
// Source emitted: 2
// Observer 1: 2
// Observer 2: 2
// Unsubscribed from source
```

**When to use**:
- When you need a simpler API than `multicast` but still want manual control over connection
- When you don't need to specify a custom Subject
- For scenarios where you need to prepare multiple subscribers before connecting to the source
- When you want to use a regular Subject (no replay or initial value behavior)

## share ⭐

**Purpose**: Shares a source Observable among multiple subscribers automatically, without requiring manual connection.

```typescript
import { interval } from 'rxjs';
import { share, tap, take } from 'rxjs/operators';

// Create a source Observable with a side effect
const source$ = interval(1000).pipe(
  take(5),
  tap(value => console.log(`Source emitted: ${value}`)),
  share() // Automatically share the subscription
);

// First subscription
console.log('Observer 1 subscribes');
const subscription1 = source$.subscribe(value => console.log(`Observer 1: ${value}`));

// Second subscription after a delay
setTimeout(() => {
  console.log('Observer 2 subscribes');
  const subscription2 = source$.subscribe(value => console.log(`Observer 2: ${value}`));
  
  // Unsubscribe the second observer after some time
  setTimeout(() => {
    console.log('Observer 2 unsubscribes');
    subscription2.unsubscribe();
  }, 2000);
}, 2000);

// Unsubscribe the first observer after some time
setTimeout(() => {
  console.log('Observer 1 unsubscribes');
  subscription1.unsubscribe();
}, 4000);

// Output:
// Observer 1 subscribes
// Source emitted: 0
// Observer 1: 0
// Source emitted: 1
// Observer 1: 1
// Observer 2 subscribes
// Observer 2: 1
// Source emitted: 2
// Observer 1: 2
// Observer 2: 2
// Observer 2 unsubscribes
// Source emitted: 3
// Observer 1: 3
// Observer 1 unsubscribes
// (Source stops emitting as all subscribers are gone)
```

**When to use**:
- For most common multicasting scenarios
- When you want automatic connection/disconnection based on subscriber count
- When you don't need to replay previous values to late subscribers
- For event streams where you only care about values emitted after subscription
- When you want to avoid manual connection management

## shareReplay ⭐

**Purpose**: Shares a source Observable and replays a specified number of emissions to new subscribers.

```typescript
import { interval } from 'rxjs';
import { shareReplay, take, tap } from 'rxjs/operators';

// Create a source Observable with a side effect
const source$ = interval(1000).pipe(
  take(5),
  tap(value => console.log(`Source emitted: ${value}`)),
  shareReplay(2) // Share and replay the last 2 values
);

// First subscription
console.log('Observer 1 subscribes');
source$.subscribe(value => console.log(`Observer 1: ${value}`));

// Second subscription after some emissions have occurred
setTimeout(() => {
  console.log('Observer 2 subscribes (late)');
  source$.subscribe(value => console.log(`Observer 2: ${value}`));
}, 3500);

// Output:
// Observer 1 subscribes
// Source emitted: 0
// Observer 1: 0
// Source emitted: 1
// Observer 1: 1
// Source emitted: 2
// Observer 1: 2
// Source emitted: 3
// Observer 1: 3
// Observer 2 subscribes (late)
// Observer 2: 2  (replayed)
// Observer 2: 3  (replayed)
// Source emitted: 4
// Observer 1: 4
// Observer 2: 4
```

**When to use**:
- When late subscribers need access to previously emitted values
- For caching API responses or expensive computations
- When implementing a "current value" pattern
- For UI components that need to display the most recent data immediately upon subscription
- When you want to combine the benefits of automatic connection management with replay functionality

## Comparison of Key Operators

| Operator | Auto-Connect | Replay Values | Reference Counting | Use Case |
|----------|--------------|--------------|-------------------|----------|
| `multicast` | No | Depends on Subject | No | Fine-grained control with custom Subjects |
| `publish` | No | No | No | Manual connection with standard Subject |
| `share` | Yes | No | Yes | Most common sharing scenarios |
| `shareReplay` | Yes | Yes | Yes (configurable) | Caching and late subscriber scenarios |

## Practical Examples

### API Caching with shareReplay

```typescript
import { Observable, of } from 'rxjs';
import { ajax } from 'rxjs/ajax';
import { shareReplay, catchError, map } from 'rxjs/operators';

// Service class for fetching user data
class UserService {
  private userCache$: Observable<any>;

  constructor() {
    // Initialize the cache
    this.userCache$ = null;
  }

  getUsers() {
    // If we already have a cached Observable, return it
    if (this.userCache$) {
      console.log('Returning cached users data');
      return this.userCache$;
    }

    // Otherwise, create a new request and cache it
    console.log('Fetching users from API');
    this.userCache$ = ajax.getJSON('https://jsonplaceholder.typicode.com/users').pipe(
      map(response => response),
      catchError(error => {
        console.error('Error fetching users:', error);
        return of([]);
      }),
      // Cache the result indefinitely (or until service is reset)
      shareReplay(1)
    );

    return this.userCache$;
  }

  // Method to clear the cache if needed
  clearCache() {
    this.userCache$ = null;
    console.log('Cache cleared');
  }
}

// Usage example
const userService = new UserService();

// First request - will fetch from API
userService.getUsers().subscribe(users => {
  console.log('First component received users:', users.length);
});

// Second request - should use cached data
setTimeout(() => {
  userService.getUsers().subscribe(users => {
    console.log('Second component received users:', users.length);
  });
}, 2000);

// Third request after cache clear - will fetch from API again
setTimeout(() => {
  userService.clearCache();
  userService.getUsers().subscribe(users => {
    console.log('Third component received users after cache clear:', users.length);
  });
}, 4000);
```

### Event Bus with share

```typescript
import { Subject } from 'rxjs';
import { share, filter, map } from 'rxjs/operators';

// Simple event bus implementation
class EventBus {
  private events$ = new Subject<{ type: string, payload: any }>();
  
  // Shared stream of events
  public sharedEvents$ = this.events$.pipe(
    share()
  );
  
  // Dispatch an event
  dispatch(type: string, payload: any) {
    this.events$.next({ type, payload });
  }
  
  // Listen for specific event types
  on(eventType: string) {
    return this.sharedEvents$.pipe(
      filter(event => event.type === eventType),
      map(event => event.payload)
    );
  }
}

// Usage example
const eventBus = new EventBus();

// Component 1 listens for 'USER_UPDATED' events
const userSubscription = eventBus.on('USER_UPDATED').subscribe(userData => {
  console.log('User component received update:', userData);
});

// Component 2 listens for 'NOTIFICATION' events
const notificationSubscription = eventBus.on('NOTIFICATION').subscribe(notification => {
  console.log('Notification received:', notification);
});

// Dispatch some events
eventBus.dispatch('USER_UPDATED', { id: 1, name: 'John Doe', status: 'active' });
eventBus.dispatch('NOTIFICATION', { message: 'New message received', type: 'info' });
eventBus.dispatch('USER_UPDATED', { id: 1, name: 'John Doe', status: 'away' });

// Clean up
setTimeout(() => {
  userSubscription.unsubscribe();
  notificationSubscription.unsubscribe();
  console.log('Subscriptions cleaned up');
}, 5000);
```

## Conclusion

Multicasting operators are essential for optimizing Observable streams in RxJS applications. They help you avoid duplicate side effects, reduce resource consumption, and maintain consistency across your application:

- Use `multicast` when you need fine-grained control with custom Subjects
- Use `publish` for manual connection with a standard Subject
- Use `share` ⭐ for most common sharing scenarios with automatic reference counting
- Use `shareReplay` ⭐ for caching and ensuring late subscribers receive important values

By choosing the right multicasting operator for your specific use case, you can significantly improve the performance and behavior of your reactive applications.

## References

- [RxJS Official Documentation](https://rxjs.dev/)
- [Learn RxJS](https://www.learnrxjs.io/)
- [RxJS Marbles: Interactive diagrams](https://rxmarbles.com/)
- [Understanding Multicasting in RxJS](https://blog.angular-university.io/rxjs-higher-order-mapping/)
