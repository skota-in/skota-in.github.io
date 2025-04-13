# Angular 18 Features: A Comprehensive Guide with Examples

*Published on June 15, 2024*

## Introduction

Angular 18 was officially released on May 22, 2024, bringing significant improvements and new features to enhance both developer experience and application performance. This version focuses on modernizing the framework with reactive programming patterns, improved server-side rendering, and better developer tooling.

In this comprehensive guide, we'll explore the key features of Angular 18 with practical examples to help you leverage these new capabilities in your projects.

## Table of Contents

1. [Zoneless Change Detection with Signals](#zoneless-change-detection-with-signals)
2. [Function-based Route Redirects](#function-based-route-redirects)
3. [New Signal APIs](#new-signal-apis)
4. [Server-Side Rendering Improvements](#server-side-rendering-improvements)
5. [Event Replay](#event-replay)
6. [Hydration Support for i18n](#hydration-support-for-i18n)
7. [Angular Material Enhancements](#angular-material-enhancements)
8. [Improved Developer Tooling](#improved-developer-tooling)
9. [Standalone API Improvements](#standalone-api-improvements)
10. [Performance Optimizations](#performance-optimizations)
11. [Conclusion](#conclusion)

## Zoneless Change Detection with Signals

Angular 18 introduces experimental support for zoneless change detection using signals. This approach eliminates the need for Zone.js, resulting in improved performance and a more predictable change detection mechanism.

### How It Works

Zoneless change detection relies on signals to explicitly track state changes, rather than the implicit tracking provided by Zone.js. This gives developers more control over when change detection occurs and reduces unnecessary change detection cycles.

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

### Example Component with Signals

```typescript
// counter.component.ts
import { Component, signal, computed, effect } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-counter',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div>
      <h2>Counter: {{ count() }}</h2>
      <p>Doubled value: {{ doubledCount() }}</p>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
    </div>
  `
})
export class CounterComponent {
  // Create a signal with an initial value of 0
  count = signal(0);
  
  // Create a computed signal that depends on count
  doubledCount = computed(() => this.count() * 2);
  
  // Create an effect that runs whenever count changes
  constructor() {
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

### Benefits

- Improved performance by eliminating Zone.js overhead
- More predictable change detection
- Better control over when change detection occurs
- Reduced bundle size
- Explicit reactivity model

## Function-based Route Redirects

Angular 18 introduces function-based route redirects, allowing for more dynamic and flexible routing configurations. This feature enables you to make routing decisions based on runtime conditions.

### Example

```typescript
// app-routing.module.ts
import { Routes } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component').then(m => m.DashboardComponent)
  },
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component').then(m => m.AdminComponent)
  },
  {
    // Function-based redirect that dynamically determines the target route
    path: '',
    redirectTo: () => {
      const authService = inject(AuthService);
      
      if (authService.isLoggedIn()) {
        if (authService.isAdmin()) {
          return 'admin';
        }
        return 'dashboard';
      }
      
      return 'login';
    },
    pathMatch: 'full'
  }
];
```

### Advanced Example with Query Parameters

```typescript
// app-routing.module.ts
import { Routes } from '@angular/router';
import { inject } from '@angular/core';
import { UserPreferencesService } from './user-preferences.service';

export const routes: Routes = [
  {
    path: 'products',
    redirectTo: (_, segments) => {
      const userPrefs = inject(UserPreferencesService);
      const defaultCategory = userPrefs.getDefaultCategory();
      const defaultSort = userPrefs.getDefaultSorting();
      
      // Redirect to products/list with user's preferred category and sorting
      return `products/list?category=${defaultCategory}&sort=${defaultSort}`;
    }
  },
  {
    path: 'products/list',
    component: ProductListComponent
  }
];
```

### Benefits

- Dynamic routing decisions based on runtime conditions
- Ability to inject services into route configurations
- More flexible and powerful routing patterns
- Cleaner routing code for complex scenarios

## New Signal APIs

Angular 18 introduces several new signal APIs in developer preview, expanding the reactive programming capabilities of the framework.

### Effect API

The `effect` function creates a side effect that automatically runs when any of the signals it depends on change.

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-logger',
  template: `<div>Check console for logs</div>`
})
export class LoggerComponent {
  firstName = signal('John');
  lastName = signal('Doe');
  
  constructor() {
    // This effect will run whenever firstName or lastName changes
    effect(() => {
      console.log(`Name changed to: ${this.firstName()} ${this.lastName()}`);
      
      // You can also perform other side effects like API calls
      this.logNameChange(this.firstName(), this.lastName());
    });
  }
  
  updateName(first: string, last: string) {
    this.firstName.set(first);
    this.lastName.set(last);
  }
  
  private logNameChange(first: string, last: string) {
    // Send to analytics, update localStorage, etc.
  }
}
```

### Untracked API

The `untracked` function allows you to access signal values without creating a dependency.

```typescript
import { Component, signal, effect, untracked } from '@angular/core';

@Component({
  selector: 'app-untracked-demo',
  template: `<div>Check console for logs</div>`
})
export class UntrackedDemoComponent {
  counter = signal(0);
  threshold = signal(5);
  
  constructor() {
    effect(() => {
      // This effect depends on counter but not on threshold
      const count = this.counter();
      const limit = untracked(() => this.threshold());
      
      console.log(`Counter: ${count}`);
      
      if (count > limit) {
        console.log('Counter exceeded threshold!');
      }
    });
  }
  
  increment() {
    this.counter.update(c => c + 1);
  }
  
  updateThreshold(newThreshold: number) {
    // Changing threshold won't trigger the effect
    this.threshold.set(newThreshold);
  }
}
```

### Benefits

- More powerful reactive programming patterns
- Better control over side effects
- Improved performance by avoiding unnecessary re-computations
- Cleaner separation of concerns

## Server-Side Rendering Improvements

Angular 18 brings significant improvements to server-side rendering (SSR), making it more robust and easier to use.

### Better Debugging

Angular 18 improves the debugging experience for SSR applications by providing more detailed error messages and better stack traces.

### Hydration Support in Angular Material

Angular Material components now have better support for hydration, ensuring a smooth transition from server-rendered content to interactive components.

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule, provideClientHydration } from '@angular/platform-browser';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    MatButtonModule,
    MatCardModule
  ],
  providers: [
    provideClientHydration() // Enable hydration
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Example SSR Component

```typescript
// product-card.component.ts
import { Component, Input } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatButtonModule } from '@angular/material/button';
import { MatCardModule } from '@angular/material/card';

@Component({
  selector: 'app-product-card',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatCardModule],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>{{ product.name }}</mat-card-title>
        <mat-card-subtitle>${{ product.price }}</mat-card-subtitle>
      </mat-card-header>
      <mat-card-content>
        <p>{{ product.description }}</p>
      </mat-card-content>
      <mat-card-actions>
        <button mat-button (click)="addToCart()">ADD TO CART</button>
        <button mat-button (click)="viewDetails()">DETAILS</button>
      </mat-card-actions>
    </mat-card>
  `
})
export class ProductCardComponent {
  @Input() product: any;
  
  addToCart() {
    console.log(`Added ${this.product.name} to cart`);
  }
  
  viewDetails() {
    console.log(`Viewing details for ${this.product.name}`);
  }
}
```

### Benefits

- Improved performance with faster initial page loads
- Better SEO with fully rendered content
- Smoother transition from server-rendered content to interactive components
- Enhanced debugging experience

## Event Replay

Angular 18 introduces event replay functionality, which captures user events during the hydration process and replays them once the application is fully hydrated. This ensures that no user interactions are lost during the transition from server-rendered content to the fully interactive application.

### Enabling Event Replay

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

- No lost user interactions during hydration
- Improved user experience with more responsive applications
- Seamless transition from server-rendered content to interactive components

## Hydration Support for i18n

Angular 18 adds hydration support for internationalization (i18n), ensuring that translated content is properly hydrated without flickering or content shifts.

### Example

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <h1 i18n="@@welcomeHeader">Welcome to our application!</h1>
    <p i18n="@@welcomeMessage">Thank you for using our platform.</p>
    
    <button i18n="@@loginButton">Login</button>
    <button i18n="@@registerButton">Register</button>
  `
})
export class AppComponent { }
```

### Configuration

```typescript
// angular.json
{
  "projects": {
    "my-app": {
      "i18n": {
        "sourceLocale": "en-US",
        "locales": {
          "fr": "src/locale/messages.fr.xlf",
          "es": "src/locale/messages.es.xlf"
        }
      },
      "architect": {
        "build": {
          "options": {
            "localize": true
          }
        },
        "serve-ssr": {
          "options": {
            "localize": true
          }
        }
      }
    }
  }
}
```

### Benefits

- Consistent internationalized content during hydration
- No content flickering when switching from server-rendered to client-rendered content
- Better user experience for international audiences

## Angular Material Enhancements

Angular 18 brings several improvements to Angular Material, including better hydration support, new components, and enhanced accessibility.

### New Date Range Picker

```typescript
// date-range-picker.component.ts
import { Component } from '@angular/core';
import { FormGroup, FormControl } from '@angular/forms';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatInputModule } from '@angular/material/input';
import { MatFormFieldModule } from '@angular/material/form-field';
import { ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-date-range-picker',
  standalone: true,
  imports: [
    MatDatepickerModule,
    MatInputModule,
    MatFormFieldModule,
    ReactiveFormsModule
  ],
  template: `
    <form [formGroup]="range">
      <mat-form-field>
        <mat-label>Enter a date range</mat-label>
        <mat-date-range-input [rangePicker]="picker">
          <input matStartDate formControlName="start" placeholder="Start date">
          <input matEndDate formControlName="end" placeholder="End date">
        </mat-date-range-input>
        <mat-datepicker-toggle matIconSuffix [for]="picker"></mat-datepicker-toggle>
        <mat-date-range-picker #picker></mat-date-range-picker>
      </mat-form-field>
    </form>
    
    <p *ngIf="range.valid">
      Selected range: {{range.value.start | date}} to {{range.value.end | date}}
    </p>
  `
})
export class DateRangePickerComponent {
  range = new FormGroup({
    start: new FormControl<Date | null>(null),
    end: new FormControl<Date | null>(null)
  });
}
```

### Enhanced Accessibility

Angular Material components in Angular 18 have improved accessibility, with better ARIA attributes, keyboard navigation, and screen reader support.

```typescript
// accessible-menu.component.ts
import { Component } from '@angular/core';
import { MatMenuModule } from '@angular/material/menu';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-accessible-menu',
  standalone: true,
  imports: [MatMenuModule, MatButtonModule],
  template: `
    <button mat-button [matMenuTriggerFor]="menu" aria-label="Open user menu">
      User Menu
    </button>
    
    <mat-menu #menu="matMenu">
      <button mat-menu-item role="menuitem" aria-label="View profile">
        <mat-icon>person</mat-icon>
        <span>Profile</span>
      </button>
      
      <button mat-menu-item role="menuitem" aria-label="View settings">
        <mat-icon>settings</mat-icon>
        <span>Settings</span>
      </button>
      
      <button mat-menu-item role="menuitem" aria-label="Log out">
        <mat-icon>exit_to_app</mat-icon>
        <span>Logout</span>
      </button>
    </mat-menu>
  `
})
export class AccessibleMenuComponent { }
```

### Benefits

- Improved user experience with new and enhanced components
- Better accessibility for all users
- Smoother hydration for Material components
- More consistent design patterns

## Improved Developer Tooling

Angular 18 enhances the developer experience with improved tooling, including better error messages, enhanced debugging capabilities, and more powerful CLI features.

### Enhanced Error Messages

Angular 18 provides more detailed and helpful error messages, making it easier to identify and fix issues in your code.

### Angular DevTools Integration

Angular DevTools now has better integration with Angular 18, providing more insights into your application's performance and structure.

### CLI Improvements

The Angular CLI in version 18 includes several improvements, such as faster builds, better error reporting, and enhanced project scaffolding.

```bash
# Generate a new standalone component with signals
ng generate component user-profile --standalone --signals

# Generate a new application with SSR and hydration enabled
ng new my-ssr-app --ssr --hydration
```

### Benefits

- Improved developer productivity
- Faster debugging and issue resolution
- Better insights into application performance
- Enhanced project scaffolding

## Standalone API Improvements

Angular 18 continues to improve the standalone API, making it easier to create and use standalone components, directives, and pipes.

### Example

```typescript
// user-card.component.ts
import { Component, Input } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { UserAvatarComponent } from './user-avatar.component';
import { UserStatusPipe } from './user-status.pipe';

@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, RouterModule, UserAvatarComponent, UserStatusPipe],
  template: `
    <div class="user-card">
      <app-user-avatar [user]="user"></app-user-avatar>
      <div class="user-info">
        <h3>{{ user.name }}</h3>
        <p>Status: {{ user.status | userStatus }}</p>
        <a [routerLink]="['/users', user.id]">View Profile</a>
      </div>
    </div>
  `,
  styles: [`
    .user-card {
      display: flex;
      padding: 16px;
      border: 1px solid #ccc;
      border-radius: 4px;
      margin-bottom: 16px;
    }
    
    .user-info {
      margin-left: 16px;
    }
  `]
})
export class UserCardComponent {
  @Input() user: any;
}
```

### Benefits

- Simplified component architecture
- Reduced boilerplate code
- Better tree-shaking for smaller bundle sizes
- Improved code organization and reusability

## Performance Optimizations

Angular 18 includes several performance optimizations, resulting in faster application startup, reduced bundle sizes, and improved runtime performance.

### Ivy Improvements

The Ivy compiler and runtime have been optimized in Angular 18, resulting in smaller bundle sizes and faster compilation.

### Tree-Shaking Enhancements

Angular 18 improves tree-shaking capabilities, ensuring that only the code you actually use is included in your production bundles.

### Example: Optimized Imports

```typescript
// Before
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-button',
  standalone: true,
  imports: [MatButtonModule], // Imports the entire module
  template: `<button mat-button>Click me</button>`
})
export class ButtonComponent { }

// After
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-button',
  standalone: true,
  imports: [MatButtonModule], // Angular 18 better tree-shakes this import
  template: `<button mat-button>Click me</button>`
})
export class ButtonComponent { }
```

### Benefits

- Faster application startup
- Reduced bundle sizes
- Improved runtime performance
- Better user experience

## Conclusion

Angular 18 brings a wealth of new features and improvements that enhance both developer experience and application performance. From zoneless change detection with signals to improved server-side rendering and enhanced developer tooling, this release provides powerful tools for building modern, efficient web applications.

By adopting these new features, you can create faster, more responsive applications while improving your development workflow. As Angular continues to evolve, it remains committed to providing a robust, scalable framework for building enterprise-grade web applications.

## References

- [Angular Official Documentation](https://angular.dev/)
- [Angular Blog: Angular v18 is now available](https://blog.angular.dev/angular-v18-is-now-available-e79d5ac0affe)
- [Angular GitHub Repository](https://github.com/angular/angular)
- [Angular Material Documentation](https://material.angular.io/)
- [Angular CLI Documentation](https://angular.dev/tools/cli)
