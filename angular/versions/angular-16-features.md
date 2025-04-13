# Angular 16 Features: A Comprehensive Guide with Examples

*Published on May 15, 2023*

## Introduction

Angular 16 was officially released on May 3, 2023, bringing significant improvements to the framework's reactivity model, server-side rendering capabilities, and developer tooling. This version represents one of the most substantial updates to Angular in recent years, with a focus on performance optimization and enhanced developer experience.

In this comprehensive guide, we'll explore the key features of Angular 16 with practical examples to help you leverage these new capabilities in your projects.

## Table of Contents

1. [Signals (Developer Preview)](#signals-developer-preview)
2. [RxJS Interoperability](#rxjs-interoperability)
3. [Non-destructive Hydration](#non-destructive-hydration)
4. [Required Inputs](#required-inputs)
5. [Router Improvements](#router-improvements)
6. [DestroyRef for Cleanup](#destroyref-for-cleanup)
7. [Standalone Components Improvements](#standalone-components-improvements)
8. [esbuild-based Builder](#esbuild-based-builder)
9. [Self-closing Tags](#self-closing-tags)
10. [Injection Context Improvements](#injection-context-improvements)
11. [Jest Support](#jest-support)
12. [Content Security Policy Improvements](#content-security-policy-improvements)
13. [TypeScript 5.0 Support](#typescript-50-support)
14. [Conclusion](#conclusion)

## Signals (Developer Preview)

Angular 16 introduces a new reactivity primitive called Signals, which provides a simpler and more efficient way to handle reactive state in your applications. Signals are objects that hold values and notify consumers when those values change.

### Basic Signal Usage

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <div>
      <h2>Counter: {{ count() }}</h2>
      <p>Doubled: {{ doubled() }}</p>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
    </div>
  `
})
export class CounterComponent {
  // Create a signal with an initial value
  count = signal(0);
  
  // Create a computed signal that depends on count
  doubled = computed(() => this.count() * 2);
  
  constructor() {
    // Create an effect that runs whenever count changes
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
    });
  }
  
  increment() {
    // Update the signal value
    this.count.update(value => value + 1);
  }
  
  decrement() {
    // Set the signal value directly
    this.count.set(this.count() - 1);
  }
}
```

### Signal Types

Angular 16 introduces three main types of signals:

1. **Writable Signals**: Created with the `signal()` function, these can be read and written to.
2. **Computed Signals**: Created with the `computed()` function, these derive their value from other signals.
3. **Effects**: Created with the `effect()` function, these run side effects when signals they depend on change.

### Benefits of Signals

- More explicit reactivity model compared to Zone.js
- Fine-grained updates that only affect what changed
- Better performance through reduced change detection cycles
- Simpler mental model for state management
- Future foundation for making Zone.js optional

## RxJS Interoperability

Angular 16 introduces a new package `@angular/core/rxjs-interop` that provides utilities for converting between Signals and RxJS Observables.

### Converting Signals to Observables

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-signal-to-observable',
  template: `<div>Check console for logs</div>`
})
export class SignalToObservableComponent {
  counter = signal(0);
  counter$ = toObservable(this.counter);
  
  constructor() {
    this.counter$.subscribe(value => {
      console.log(`Counter value from observable: ${value}`);
    });
    
    // Update the signal
    setInterval(() => {
      this.counter.update(value => value + 1);
    }, 1000);
  }
}
```

### Converting Observables to Signals

```typescript
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';
import { map } from 'rxjs/operators';

@Component({
  selector: 'app-observable-to-signal',
  template: `
    <div>
      <h2>Timer: {{ timer() }}</h2>
    </div>
  `
})
export class ObservableToSignalComponent {
  // Create an observable
  timer$ = interval(1000).pipe(
    map(value => `${Math.floor(value / 60)}:${(value % 60).toString().padStart(2, '0')}`)
  );
  
  // Convert the observable to a signal with an initial value
  timer = toSignal(this.timer$, { initialValue: '0:00' });
}
```

### takeUntilDestroyed Operator

Angular 16 also introduces a new RxJS operator called `takeUntilDestroyed` that automatically unsubscribes from observables when the component is destroyed:

```typescript
import { Component } from '@angular/core';
import { interval } from 'rxjs';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-auto-unsubscribe',
  template: `<div>Check console for logs</div>`
})
export class AutoUnsubscribeComponent {
  constructor() {
    // This subscription will be automatically cleaned up when the component is destroyed
    interval(1000)
      .pipe(takeUntilDestroyed())
      .subscribe(value => {
        console.log(`Interval: ${value}`);
      });
  }
}
```

## Non-destructive Hydration

Angular 16 introduces non-destructive hydration for server-side rendered (SSR) applications. This feature significantly improves the performance of SSR applications by reusing the server-rendered DOM instead of re-rendering everything on the client.

### Enabling Hydration

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration()
  ]
});
```

### Setting Up SSR

To set up server-side rendering with hydration:

```bash
# Add SSR support to your project
ng add @nguniversal/express-engine
```

### Skipping Hydration for Specific Components

For components that manipulate the DOM directly or use third-party libraries that do so, you can skip hydration:

```html
<app-third-party-component ngSkipHydration></app-third-party-component>
```

Or using host binding:

```typescript
@Component({
  selector: 'app-third-party-component',
  template: `...`,
  host: { 'ngSkipHydration': 'true' }
})
export class ThirdPartyComponent {
  // ...
}
```

### Benefits of Non-destructive Hydration

- Eliminates content flickering during hydration
- Improves Core Web Vitals metrics (up to 45% improvement in Largest Contentful Paint)
- Reduces Time to Interactive (TTI)
- Provides a smoother user experience

## Required Inputs

Angular 16 introduces the ability to mark component inputs as required, which helps catch missing inputs at compile time rather than runtime.

### Marking Inputs as Required

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  template: `
    <div class="user-profile">
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
    </div>
  `
})
export class UserProfileComponent {
  @Input({ required: true }) user: User;
}
```

With this change, the Angular compiler will generate an error if you try to use the component without providing the required input:

```html
<!-- This will cause a compile-time error -->
<app-user-profile></app-user-profile>

<!-- This is correct -->
<app-user-profile [user]="currentUser"></app-user-profile>
```

## Router Improvements

Angular 16 enhances the router with several new features that improve developer experience and functionality.

### Component Input Binding

The router can now automatically bind route parameters to component inputs:

```typescript
// app.routes.ts
const routes: Routes = [
  {
    path: 'product/:id',
    component: ProductDetailComponent
  }
];

// product-detail.component.ts
@Component({
  selector: 'app-product-detail',
  template: `
    <div>
      <h2>Product Details</h2>
      <p>Product ID: {{ id }}</p>
    </div>
  `
})
export class ProductDetailComponent {
  @Input() id: string;
}
```

To enable this feature, use the `withComponentInputBinding` function:

```typescript
// app.config.ts
import { provideRouter, withComponentInputBinding } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding())
  ]
};
```

### Last Successful Navigation

The router now provides access to the last successful navigation:

```typescript
import { Component, inject } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-navigation-info',
  template: `
    <div>
      <h2>Navigation Info</h2>
      <p>Current URL: {{ currentUrl }}</p>
    </div>
  `
})
export class NavigationInfoComponent {
  router = inject(Router);
  
  get currentUrl(): string {
    return this.router.lastSuccessfulNavigation?.finalUrl?.toString() || '';
  }
}
```

## DestroyRef for Cleanup

Angular 16 introduces a new `DestroyRef` service that provides a way to register cleanup functions that will be called when a component, directive, or service is destroyed.

### Basic Usage

```typescript
import { Component, inject, DestroyRef } from '@angular/core';
import { interval, Subscription } from 'rxjs';

@Component({
  selector: 'app-cleanup-demo',
  template: `<div>Check console for logs</div>`
})
export class CleanupDemoComponent {
  private destroyRef = inject(DestroyRef);
  private subscription: Subscription;
  
  constructor() {
    this.subscription = interval(1000).subscribe(value => {
      console.log(`Interval: ${value}`);
    });
    
    // Register cleanup function
    this.destroyRef.onDestroy(() => {
      console.log('Cleaning up subscription');
      this.subscription.unsubscribe();
    });
  }
}
```

### Canceling Cleanup

You can also cancel a registered cleanup function if needed:

```typescript
import { Component, inject, DestroyRef } from '@angular/core';

@Component({
  selector: 'app-cancel-cleanup-demo',
  template: `<div>Check console for logs</div>`
})
export class CancelCleanupDemoComponent {
  private destroyRef = inject(DestroyRef);
  
  constructor() {
    // Register cleanup function and store the cleanup handle
    const cleanup = this.destroyRef.onDestroy(() => {
      console.log('This will not be called if canceled');
    });
    
    // Cancel the cleanup function
    cleanup();
    
    // Register another cleanup function that will be called
    this.destroyRef.onDestroy(() => {
      console.log('This will be called when component is destroyed');
    });
  }
}
```

## Standalone Components Improvements

Angular 16 continues to improve support for standalone components, making them easier to use and more powerful.

### Standalone by Default in CLI

You can now create new projects with standalone components by default:

```bash
ng new my-app --standalone
```

This will create a new project without NgModules, using standalone components throughout.

### Standalone Migration Schematic

Angular 16 provides a schematic to help migrate existing applications to standalone components:

```bash
ng generate @angular/core:standalone
```

This schematic will:
- Convert components, directives, and pipes to standalone
- Remove unnecessary NgModule declarations
- Update the bootstrap process to use standalone APIs

### Zone.js Configuration with Standalone

You can now configure Zone.js with the standalone bootstrap API:

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideZoneChangeDetection } from '@angular/core';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
});
```

## esbuild-based Builder

Angular 16 introduces a developer preview of an esbuild-based build system that significantly improves build performance.

### Enabling esbuild

To use the new esbuild-based builder, update your `angular.json` file:

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular-devkit/build-angular:browser-esbuild",
          "options": {
            // ...
          }
        }
      }
    }
  }
}
```

### Benefits of esbuild

- Up to 72% faster build times compared to webpack
- Improved development experience with faster rebuilds
- Better memory usage
- Simplified configuration

## Self-closing Tags

Angular 16 now supports self-closing tags for components in templates, which can make your templates cleaner and more concise.

### Before Angular 16

```html
<app-user-avatar [user]="currentUser"></app-user-avatar>
```

### With Angular 16

```html
<app-user-avatar [user]="currentUser" />
```

This is especially useful for components with many attributes or long names.

## Injection Context Improvements

Angular 16 improves the injection context system with new functions that make it easier to work with dependency injection.

### assertInInjectionContext

The new `assertInInjectionContext` function helps ensure that code is running within an injection context:

```typescript
import { assertInInjectionContext, inject } from '@angular/core';

function getService(): UserService {
  assertInInjectionContext(getService);
  return inject(UserService);
}
```

### runInInjectionContext

The `runInInjectionContext` function allows you to run code within an injection context:

```typescript
import { runInInjectionContext, inject, Injector } from '@angular/core';

function getUserData(injector: Injector): User {
  let userService: UserService;
  
  runInInjectionContext(injector, () => {
    userService = inject(UserService);
  });
  
  return userService.getCurrentUser();
}
```

## Jest Support

Angular 16 introduces experimental support for Jest as a test runner, providing an alternative to Karma.

### Enabling Jest

To use Jest for testing, update your `angular.json` file:

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "test": {
          "builder": "@angular-devkit/build-angular:jest",
          "options": {
            "tsConfig": "tsconfig.spec.json",
            "polyfills": ["zone.js", "zone.js/testing"]
          }
        }
      }
    }
  }
}
```

### Benefits of Jest

- Faster test execution
- Simpler configuration
- Better snapshot testing
- Improved mocking capabilities
- No need for a browser for most tests

## Content Security Policy Improvements

Angular 16 enhances support for Content Security Policy (CSP) by allowing you to specify a nonce for inline styles.

### Using ngCspNonce Attribute

```html
<html>
<body>
  <app-root ngCspNonce="random-nonce-value"></app-root>
</body>
</html>
```

### Using CSP_NONCE Injection Token

```typescript
import { bootstrapApplication, CSP_NONCE } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    { provide: CSP_NONCE, useValue: 'random-nonce-value' }
  ]
});
```

## TypeScript 5.0 Support

Angular 16 adds support for TypeScript 5.0, which brings several improvements and new features:

- Decorators are now a first-class language feature (Stage 3 ECMAScript proposal)
- Improved type inference
- Better performance
- Smaller npm package size
- Removal of the need for ngcc (Angular Compatibility Compiler)

## Conclusion

Angular 16 represents a significant step forward for the framework, introducing powerful new features like Signals and non-destructive hydration that lay the groundwork for future improvements. The focus on developer experience, performance, and modern web capabilities makes this one of the most substantial Angular releases in recent years.

By adopting these new features, you can create faster, more responsive applications while improving your development workflow. The introduction of Signals marks the beginning of a new era for Angular's reactivity model, promising even more improvements in future versions.

## References

- [Angular Official Documentation](https://angular.io/)
- [Angular Blog: Angular v16 is here!](https://blog.angular.dev/angular-v16-is-here-4d7a28ec680d)
- [Angular GitHub Repository](https://github.com/angular/angular)
- [What's new in Angular 16?](https://www.angulararchitects.io/en/blog/whats-new-in-angular-16-signals-hydration-esbuild-and-more/)
- [Angular CLI Documentation](https://angular.io/cli)
