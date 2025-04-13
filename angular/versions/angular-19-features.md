# Angular 19 Features: A Comprehensive Guide with Examples

*Published on December 15, 2024*

## Introduction

Angular 19 was officially released on November 19, 2024, bringing a wealth of new features, improvements, and optimizations to enhance both developer experience and application performance. This version focuses on speed, flexibility, and modern development practices, with significant improvements to server-side rendering, state management, and overall application architecture.

In this comprehensive guide, we'll explore the key features of Angular 19 with practical examples to help you leverage these new capabilities in your projects.

## Table of Contents

1. [Incremental Hydration](#incremental-hydration)
2. [Route-level Render Mode](#route-level-render-mode)
3. [Linked Signals](#linked-signals)
4. [Event Replay](#event-replay)
5. [Standalone by Default](#standalone-by-default)
6. [Strict Standalone Enforcement](#strict-standalone-enforcement)
7. [Resource and rxResource APIs](#resource-and-rxresource-apis)
8. [Hot Module Replacement](#hot-module-replacement)
9. [Warnings for Unused Standalone Imports](#warnings-for-unused-standalone-imports)
10. [Time Picker Component](#time-picker-component)
11. [Two-dimensional Drag and Drop](#two-dimensional-drag-and-drop)
12. [Zoneless Change Detection](#zoneless-change-detection)
13. [Modernizing Code with Language Service](#modernizing-code-with-language-service)
14. [Security Improvements](#security-improvements)
15. [Conclusion](#conclusion)

## Incremental Hydration

Incremental hydration is one of the most significant features in Angular 19, aimed at optimizing the performance of server-side rendered applications. This feature allows you to selectively hydrate parts of your application based on specific triggers, rather than hydrating the entire application at once.

### How It Works

Incremental hydration uses the familiar `@defer` syntax (introduced in Angular 17) to mark sections of your template that should be lazy-loaded and hydrated only when certain triggers occur.

### Example

```typescript
// app.component.html
<header>This content is immediately hydrated</header>

@defer (hydrate on viewport) {
  <user-profile></user-profile>
}

@defer (hydrate on idle) {
  <recommendations></recommendations>
}

@defer (hydrate on interaction) {
  <comments-section></comments-section>
}

<footer>This content is also immediately hydrated</footer>
```

### Enabling Incremental Hydration

To enable incremental hydration in your Angular 19 application:

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withIncrementalHydration } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(withIncrementalHydration())
  ]
});
```

### Benefits

- Faster initial page load times by reducing the amount of JavaScript needed for interactivity
- Prioritized hydration of critical components first
- Better user experience with more responsive applications
- Reduced Time to Interactive (TTI) metrics

## Route-level Render Mode

Angular 19 introduces server route configuration, allowing developers to specify different rendering modes for individual routes in their application. This feature provides granular control over how each route is rendered, choosing between Server-Side Rendering (SSR), Static Site Generation (SSG), or Client-Side Rendering (CSR).

### Example

```typescript
// app.routes.server.ts
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: '', // Root path
    renderMode: RenderMode.Client, // Rendered on the client (CSR)
  },
  {
    path: 'about',
    renderMode: RenderMode.Prerender, // Prerendered at build time (SSG)
  },
  {
    path: 'profile',
    renderMode: RenderMode.Server, // Rendered on the server per request (SSR)
  },
  {
    path: '**', // Wildcard route for all other paths
    renderMode: RenderMode.Server, // Default to SSR
  },
];
```

### Prerendering with Route Parameters

You can also prerender routes with dynamic parameters:

```typescript
// app.routes.server.ts
import { RenderMode, ServerRoute } from '@angular/ssr';
import { inject } from '@angular/core';
import { ProductService } from './services/product.service';

export const serverRoutes: ServerRoute[] = [
  {
    path: 'product/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderPaths() {
      const productService = inject(ProductService);
      const productIds = await productService.getProductIds(); // Returns ["1", "2", "3"]
      return productIds.map(id => ({ id })); // Generates ["/product/1", "/product/2", "/product/3"]
    },
  },
];
```

### Benefits

- Optimized rendering strategy for each route based on its specific requirements
- Improved performance by using the most appropriate rendering mode
- Better SEO for content-heavy pages with server-side rendering
- Enhanced user experience with faster initial loads

## Linked Signals

Angular 19 introduces a new reactive primitive called `linkedSignal`, which creates a writable signal that automatically updates its value based on changes in a source signal. This feature is particularly useful for managing state that depends on other reactive states.

### Basic Example

```typescript
import { signal, linkedSignal } from '@angular/core';

// Create a source signal
const animals = signal(['guinea pig', 'cat', 'dog']);

// Create a linked signal that depends on the animals signal
const selectedAnimal = linkedSignal(() => animals()[0]);

console.log(selectedAnimal()); // 'guinea pig'

// When the source signal changes, the linked signal updates automatically
animals.set(['elephant', 'rabbit']);
console.log(selectedAnimal()); // 'elephant'

// You can also manually update the linked signal
selectedAnimal.set('rabbit');
console.log(selectedAnimal()); // 'rabbit'
```

### Advanced Example with Computation

For more complex scenarios, you can use the advanced API with source and computation:

```typescript
import { signal, linkedSignal } from '@angular/core';

const animals = signal<string[]>(['guinea pig', 'cat', 'dog']);

const selectedAnimal = linkedSignal({
  source: animals,
  computation: (newAnimals: string[], previous?: { value: string }) => {
    // If the previous selection exists in the new list, keep it. Otherwise, use the first animal.
    return previous && newAnimals.includes(previous.value)
      ? previous.value
      : newAnimals[0];
  },
});

// Initial selection is the first animal
console.log(selectedAnimal()); // 'guinea pig'

// Update the list but keep 'guinea pig' in it
animals.set(['horse', 'guinea pig']);
console.log(selectedAnimal()); // 'guinea pig' (preserved because it's still in the list)

// Update the list without the current selection
animals.set(['horse', 'fish']);
console.log(selectedAnimal()); // 'horse' (reset to first item because 'guinea pig' is no longer in the list)

// Manually select an animal
selectedAnimal.set('fish');
console.log(selectedAnimal()); // 'fish'
```

### Benefits

- Simplified state management for dependent values
- Reduced boilerplate code compared to using effects
- Clear expression of relationships between signals
- Flexibility for both simple and complex state dependencies

## Event Replay

Event replay is a feature that ensures seamless execution of user interactions in server-side rendered or partially hydrated applications. It captures and queues user interactions that occur before the application is fully hydrated, then replays them once hydration is complete.

### Enabling Event Replay

In Angular 19, event replay is stable and enabled by default for all new applications that use server-side rendering:

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideClientHydration, withEventReplay } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideClientHydration(withEventReplay())
  ]
});
```

### Benefits

- Improved user experience by handling interactions that occur during hydration
- Reduced perceived latency for users interacting with the application
- Seamless transition from server-rendered HTML to interactive application
- No lost user interactions during the hydration process

## Standalone by Default

In Angular 19, standalone components are now the default way to define components, directives, and pipes. This change simplifies the metadata of all your Angular abstractions.

### Example

Before Angular 19, you had to explicitly set `standalone: true`:

```typescript
// Before Angular 19
@Component({
  selector: 'app-user-profile',
  templateUrl: './user-profile.component.html',
  standalone: true,
  imports: [CommonModule]
})
export class UserProfileComponent { }
```

With Angular 19, the `standalone` property is no longer needed for standalone components:

```typescript
// Angular 19
@Component({
  selector: 'app-user-profile',
  templateUrl: './user-profile.component.html',
  imports: [CommonModule]
})
export class UserProfileComponent { }
```

For non-standalone components, you need to explicitly set `standalone: false`:

```typescript
// Angular 19 - non-standalone component
@Component({
  selector: 'app-legacy-component',
  templateUrl: './legacy-component.component.html',
  standalone: false
})
export class LegacyComponent { }
```

### Benefits

- Reduced boilerplate code
- Simplified component definition
- Encourages the use of modern Angular patterns
- Better alignment with Angular's future direction

## Strict Standalone Enforcement

Angular 19 introduces a compiler flag to enforce the use of standalone components, directives, and pipes. This helps teams ensure that all new code follows modern Angular practices.

### Enabling Strict Standalone

To enable strict standalone enforcement, update your `tsconfig.json`:

```json
{
  "angularCompilerOptions": {
    "strictStandalone": true
  }
}
```

With this flag enabled, the compiler will generate an error if it detects a component, directive, or pipe that isn't standalone:

```
[ERROR] TS-992023: Only standalone components/directives are allowed when 'strictStandalone' is enabled. [plugin angular-compiler]
```

### Benefits

- Enforces consistent use of modern Angular patterns
- Helps teams migrate to standalone components
- Prevents accidental creation of non-standalone components
- Simplifies application architecture

## Resource and rxResource APIs

Angular 19 introduces experimental `resource` and `rxResource` APIs to streamline asynchronous data retrieval and management by leveraging signals and observables.

### Resource API Example

The `resource` function creates a reactive data source that fetches asynchronous data and provides signals to monitor its state:

```typescript
import { Component, inject } from '@angular/core';
import { resource, signal } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user.isLoading()">Loading...</div>
    <div *ngIf="user.error() as error">Error: {{ error.message }}</div>
    <div *ngIf="user.value() as userData">
      <h2>{{ userData.name }}</h2>
      <p>{{ userData.email }}</p>
    </div>
    <button (click)="updateUserId(userId() + 1)">Next User</button>
  `
})
export class UserProfileComponent {
  private userService = inject(UserService);
  userId = signal(1);

  updateUserId(id: number) {
    this.userId.set(id);
  }

  user = resource({
    request: () => ({ id: this.userId() }),
    loader: ({ request, abortSignal }) =>
      this.userService.getUser(request.id, abortSignal)
  });
}
```

### rxResource API Example

The `rxResource` function offers similar functionality but integrates with RxJS observables:

```typescript
import { Component, inject } from '@angular/core';
import { rxResource, signal } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user.isLoading()">Loading...</div>
    <div *ngIf="user.error() as error">Error: {{ error.message }}</div>
    <div *ngIf="user.value() as userData">
      <h2>{{ userData.name }}</h2>
      <p>{{ userData.email }}</p>
    </div>
    <button (click)="updateUserId(userId() + 1)">Next User</button>
  `
})
export class UserProfileComponent {
  private userService = inject(UserService);
  userId = signal(1);

  updateUserId(id: number) {
    this.userId.set(id);
  }

  user = rxResource({
    request: () => ({ id: this.userId() }),
    loader: ({ request }) => this.userService.getUserAsObservable(request.id)
  });
}
```

### Benefits

- Simplified data fetching and state management
- Automatic handling of loading states and errors
- Reactive updates when dependencies change
- Cancellation of in-flight requests when parameters change
- Integration with both Promise-based and Observable-based APIs

## Hot Module Replacement

Angular 19 introduces hot module replacement (HMR) for both styles and templates, streamlining the development workflow by eliminating the need for full-page refreshes after style or template modifications.

### Enabling HMR

To enable HMR in your Angular 19 project, update your `angular.json` file:

```json
{
  "projects": {
    "your-project": {
      "architect": {
        "serve": {
          "options": {
            "hmr": true
          }
        }
      }
    }
  }
}
```

Then run your development server with the HMR flag:

```bash
ng serve --hmr
```

### Benefits

- Faster development cycles
- Preserved application state during updates
- Immediate feedback for style and template changes
- Improved developer experience

## Warnings for Unused Standalone Imports

Angular 19 includes a feature that reports warnings for unused imports in your application, helping you identify unnecessary or obsolete imports in your project files.

### Example

If you have a component with unused imports:

```typescript
@Component({
  selector: 'app-test',
  imports: [IconComponent], // IconComponent is not used in the template
  templateUrl: './test.component.html',
})
export class TestComponent { }
```

Angular will generate a warning:

```
[WARNING] TS-998113: All imports are unused [plugin angular-compiler]
imports: [IconComponent]
```

### Benefits

- Cleaner code with fewer unnecessary imports
- Smaller bundle sizes
- Improved code maintainability
- Early detection of unused dependencies

## Time Picker Component

Angular Material in Angular 19 introduces a new TimePicker component, a highly requested feature that allows users to select a specific time.

### Example

```typescript
import { Component } from '@angular/core';
import { FormControl } from '@angular/forms';
import { MatTimepickerModule } from '@angular/material/timepicker';
import { ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-time-picker-example',
  standalone: true,
  imports: [MatTimepickerModule, ReactiveFormsModule],
  template: `
    <mat-form-field>
      <mat-label>Select time</mat-label>
      <input matTimepicker [formControl]="timeControl">
      <mat-timepicker-toggle matSuffix [for]="timepicker"></mat-timepicker-toggle>
      <mat-timepicker #timepicker></mat-timepicker>
    </mat-form-field>
    <p>Selected time: {{ timeControl.value | date:'shortTime' }}</p>
  `
})
export class TimePickerExampleComponent {
  timeControl = new FormControl(new Date());
}
```

### Benefits

- Native time selection in Angular Material
- Consistent design with other Material components
- Accessibility features built-in
- Customizable time formats and validation

## Two-dimensional Drag and Drop

Angular 19 enhances the CDK (Component Development Kit) with support for two-dimensional drag-and-drop functionality, allowing for more flexible and interactive interfaces.

### Example

```typescript
import { Component } from '@angular/core';
import { CdkDrag, CdkDragHandle, CdkDropList } from '@angular/cdk/drag-drop';

@Component({
  selector: 'app-grid-drag-drop',
  standalone: true,
  imports: [CdkDrag, CdkDragHandle, CdkDropList],
  template: `
    <div
      cdkDropList
      cdkDropListOrientation="mixed"
      class="grid-container"
      (cdkDropListDropped)="drop($event)">
      @for (item of items; track item.id) {
        <div cdkDrag class="grid-item">
          {{ item.content }}
          <mat-icon cdkDragHandle svgIcon="dnd-move"></mat-icon>
        </div>
      }
    </div>
  `,
  styles: [`
    .grid-container {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      padding: 10px;
    }
    .grid-item {
      padding: 20px;
      background-color: #f0f0f0;
      border-radius: 4px;
      cursor: move;
      position: relative;
    }
    .cdk-drag-placeholder {
      opacity: 0.3;
    }
    .cdk-drag-animating {
      transition: transform 250ms cubic-bezier(0, 0, 0.2, 1);
    }
  `]
})
export class GridDragDropComponent {
  items = Array.from({ length: 9 }, (_, i) => ({
    id: i.toString(),
    content: `Item ${i + 1}`
  }));

  drop(event: CdkDragDrop<string[]>) {
    moveItemInArray(this.items, event.previousIndex, event.currentIndex);
  }
}
```

### Benefits

- Support for grid-based layouts
- Flexible item positioning
- Improved user experience for drag-and-drop interactions
- Enhanced control over drag-and-drop behavior

## Zoneless Change Detection

Angular 19 continues to develop zoneless change detection, which aims to improve performance by eliminating the need for Zone.js. While still experimental, this feature shows promise in enhancing application performance.

### Enabling Zoneless Change Detection

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideExperimentalZonelessChangeDetection } from '@angular/core';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

### Example Component with Manual Change Detection

```typescript
import { Component, ChangeDetectorRef } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Counter: {{ count }}</h2>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = 0;

  constructor(private cdr: ChangeDetectorRef) {}

  increment() {
    this.count++;
    // With zoneless change detection, we need to manually trigger change detection
    this.cdr.detectChanges();
  }
}
```

### Benefits

- Improved performance with reduced overhead
- Smaller bundle sizes by eliminating Zone.js
- More predictable change detection
- Better control over when change detection occurs

## Modernizing Code with Language Service

Angular 19 introduces a seamless integration between schematics and the language service to streamline the update process for your code. This makes it easier to migrate to the latest Angular APIs directly within your code editor.

### Benefits

- In-editor code modernization suggestions
- Simplified migration to latest Angular features
- Reduced manual refactoring effort
- Improved developer productivity

## Security Improvements

Angular 19 continues to benefit from Google's strong security practices and commitment to open-source development. The framework undergoes regular security audits and updates to address vulnerabilities promptly.

### Benefits

- Regular security updates
- Protection against common web vulnerabilities
- Secure-by-default architecture
- Ongoing security maintenance

## Conclusion

Angular 19 brings a wealth of new features and improvements that enhance both developer experience and application performance. From incremental hydration and route-level rendering to linked signals and resource APIs, this release provides powerful tools for building modern, efficient web applications.

By adopting these new features, you can create faster, more responsive applications while improving your development workflow. As Angular continues to evolve, it remains committed to providing a robust, scalable framework for building enterprise-grade web applications.

## References

- [Angular Official Documentation](https://angular.dev/)
- [Angular Blog: Meet Angular v19](https://blog.angular.dev/meet-angular-v19-7b29dfd05b84)
- [Angular GitHub Repository](https://github.com/angular/angular)
- [Angular Material Documentation](https://material.angular.io/)
- [Angular CDK Documentation](https://material.angular.io/cdk/categories)