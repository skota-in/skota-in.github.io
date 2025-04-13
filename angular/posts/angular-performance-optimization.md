# Angular Performance Optimization: Techniques and Best Practices

*Published on August 5, 2023*

## Introduction

Performance is a critical aspect of any web application. A slow, unresponsive application can lead to poor user experience and high bounce rates. Angular provides several built-in features and techniques to optimize performance. This post explores various strategies to improve the performance of your Angular applications, from simple optimizations to advanced techniques.

## Table of Contents

1. [Understanding Angular Performance](#understanding-angular-performance)
2. [Change Detection Optimization](#change-detection-optimization)
3. [Lazy Loading](#lazy-loading)
4. [AOT Compilation](#aot-compilation)
5. [Tree Shaking](#tree-shaking)
6. [Bundle Optimization](#bundle-optimization)
7. [Memory Management](#memory-management)
8. [Server-Side Rendering (SSR)](#server-side-rendering-ssr)
9. [Web Workers](#web-workers)
10. [Virtual Scrolling](#virtual-scrolling)
11. [Performance Profiling](#performance-profiling)
12. [Best Practices](#best-practices)
13. [Conclusion](#conclusion)

## Understanding Angular Performance

Before diving into specific optimization techniques, it's important to understand what affects Angular application performance:

1. **Initial Load Time**: How quickly your application loads when a user first visits it.
2. **Runtime Performance**: How responsive your application is during user interaction.
3. **Memory Usage**: How efficiently your application uses memory.
4. **Network Efficiency**: How your application handles data transfer over the network.

Each of these aspects can be optimized using different techniques, which we'll explore in this post.

## Change Detection Optimization

Angular's change detection mechanism is responsible for keeping the UI in sync with the application state. By default, Angular uses a strategy called "Zone.js-based change detection," which can be inefficient for large applications.

### OnPush Change Detection Strategy

The OnPush change detection strategy tells Angular to only check for changes when:
- An input property changes
- An event from the component or one of its children is triggered
- An Observable linked to the template via the async pipe emits a new value

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MyComponent {
  // Component logic
}
```

### Detaching Change Detector

For even more control, you can manually detach and reattach the change detector:

```typescript
import { Component, ChangeDetectorRef } from '@angular/core';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html'
})
export class MyComponent {
  constructor(private cdr: ChangeDetectorRef) {
    // Detach the change detector
    this.cdr.detach();
    
    // Some operation that doesn't require UI updates
    
    // Reattach and check for changes when needed
    this.cdr.reattach();
    // or just detect changes without reattaching
    this.cdr.detectChanges();
  }
}
```

### Using Pure Pipes

Pure pipes are only recalculated when their input values change, making them more efficient than impure pipes or component methods:

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter',
  pure: true // This is the default
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], field: string, value: any): any[] {
    if (!items) return [];
    return items.filter(item => item[field] === value);
  }
}
```

**When to use**:
- Use OnPush change detection for components that don't need frequent updates
- Detach change detectors for components that update independently of the rest of the application
- Use pure pipes instead of methods in templates for transformations

## Lazy Loading

Lazy loading is a design pattern that delays the loading of non-essential resources until they are needed. In Angular, this typically means loading feature modules only when the user navigates to their routes.

### Setting Up Lazy Loading

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', loadChildren: () => import('./home/home.module').then(m => m.HomeModule) },
  { path: 'products', loadChildren: () => import('./products/products.module').then(m => m.ProductsModule) },
  { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Preloading Strategies

Angular provides several preloading strategies to improve the user experience:

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule, PreloadAllModules } from '@angular/router';

const routes: Routes = [
  // ... routes as before
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: PreloadAllModules
  })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

### Custom Preloading Strategy

For more control, you can create a custom preloading strategy:

```typescript
// selective-preloading-strategy.ts
import { Injectable } from '@angular/core';
import { PreloadingStrategy, Route } from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data && route.data.preload ? load() : of(null);
  }
}
```

Then use it in your routing module:

```typescript
// app-routing.module.ts
const routes: Routes = [
  { 
    path: 'products', 
    loadChildren: () => import('./products/products.module').then(m => m.ProductsModule),
    data: { preload: true }
  },
  // ... other routes
];

@NgModule({
  imports: [RouterModule.forRoot(routes, {
    preloadingStrategy: SelectivePreloadingStrategy
  })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

**When to use**:
- Use lazy loading for large feature modules that aren't immediately needed
- Use preloading strategies to improve the user experience after the initial load
- Consider custom preloading strategies for fine-grained control over which modules to preload

## AOT Compilation

Ahead-of-Time (AOT) compilation compiles your Angular application during the build process rather than at runtime in the browser.

### Benefits of AOT Compilation

1. **Faster rendering**: The browser downloads a pre-compiled version of the application
2. **Fewer asynchronous requests**: The compiler inlines external HTML templates and CSS style sheets
3. **Smaller Angular framework download size**: No need to download the Angular compiler
4. **Earlier detection of template errors**: Catch template errors during development
5. **Better security**: AOT compiles HTML templates and components into JavaScript files before they are served to the client

### Enabling AOT Compilation

AOT compilation is enabled by default in production builds with Angular CLI:

```bash
ng build --prod
```

For development builds, you can enable it with:

```bash
ng build --aot
```

**When to use**:
- Always use AOT compilation for production builds
- Consider using AOT for development if you want to catch template errors early

## Tree Shaking

Tree shaking is a build optimization technique that removes unused code from the final bundle.

### How Tree Shaking Works

Tree shaking works by analyzing the import and export statements in your code and removing any code that isn't actually used.

### Enabling Tree Shaking

Tree shaking is enabled by default in production builds with Angular CLI. To make the most of it:

1. Use ES modules (import/export) syntax
2. Avoid side effects in your code
3. Use the `"sideEffects": false` flag in your package.json (or specify files with side effects)

```json
// package.json
{
  "name": "my-app",
  "version": "0.0.0",
  "sideEffects": false,
  // or
  "sideEffects": [
    "*.css",
    "*.scss"
  ]
}
```

**When to use**:
- Always enable tree shaking for production builds
- Structure your code to be tree-shakable by using ES modules and avoiding side effects

## Bundle Optimization

Optimizing your application's bundle size is crucial for improving initial load time.

### Code Splitting

Code splitting divides your code into smaller chunks that can be loaded on demand:

```typescript
// Using dynamic imports for code splitting
const loadLibrary = async () => {
  const library = await import('./heavy-library');
  return library;
};

// Use the library only when needed
button.addEventListener('click', async () => {
  const library = await loadLibrary();
  library.doSomething();
});
```

### Analyzing Bundle Size

Use tools like Webpack Bundle Analyzer to identify large dependencies:

```bash
ng build --prod --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

### Differential Loading

Angular CLI automatically creates differential builds for modern and legacy browsers:

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2015",
    // ...
  }
}
```

**When to use**:
- Always analyze your bundle size to identify optimization opportunities
- Use code splitting for large libraries that aren't needed immediately
- Enable differential loading to provide optimized bundles for modern browsers

## Memory Management

Proper memory management is essential for maintaining good runtime performance.

### Avoiding Memory Leaks

The most common cause of memory leaks in Angular applications is forgetting to unsubscribe from Observables:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';
import { DataService } from './data.service';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html'
})
export class MyComponent implements OnInit, OnDestroy {
  private subscription = new Subscription();
  
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    this.subscription.add(
      this.dataService.getData().subscribe(data => {
        // Handle data
      })
    );
  }
  
  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

### Using the Async Pipe

The async pipe automatically subscribes and unsubscribes from Observables:

```html
<div *ngIf="data$ | async as data">
  {{ data }}
</div>
```

### Using takeUntil for Cleaner Unsubscription

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
import { DataService } from './data.service';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.component.html'
})
export class MyComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  constructor(private dataService: DataService) {}
  
  ngOnInit() {
    this.dataService.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(data => {
      // Handle data
    });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**When to use**:
- Always unsubscribe from Observables in the ngOnDestroy lifecycle hook
- Use the async pipe when possible to avoid manual subscription management
- Consider using takeUntil for cleaner unsubscription logic

## Server-Side Rendering (SSR)

Server-Side Rendering (SSR) generates the initial HTML for your application on the server rather than in the browser.

### Benefits of SSR

1. **Improved SEO**: Search engines can crawl the fully rendered page
2. **Faster initial load**: Users see content sooner
3. **Better performance on low-powered devices**: Less JavaScript processing required on the client

### Setting Up Angular Universal

```bash
ng add @nguniversal/express-engine
```

This command adds the necessary dependencies and configuration for SSR with Express.

### Building and Running SSR

```bash
npm run build:ssr
npm run serve:ssr
```

**When to use**:
- For public-facing applications where SEO is important
- For applications targeting users with slow internet connections or low-powered devices
- When you need to improve perceived performance by showing content faster

## Web Workers

Web Workers allow you to run CPU-intensive tasks in a background thread, keeping the main thread free for UI updates.

### Setting Up Web Workers

```bash
ng generate web-worker app
```

This command generates a web worker and updates your application to use it.

### Using Web Workers

```typescript
// app.component.ts
export class AppComponent {
  constructor() {
    if (typeof Worker !== 'undefined') {
      // Create a new web worker
      const worker = new Worker('./app.worker', { type: 'module' });
      
      // Send data to the worker
      worker.postMessage({ data: 'hello' });
      
      // Receive data from the worker
      worker.onmessage = ({ data }) => {
        console.log(`Received from worker: ${data}`);
      };
    } else {
      // Web Workers are not supported in this environment
      // You should add a fallback so that your program still executes correctly
    }
  }
}
```

```typescript
// app.worker.ts
/// <reference lib="webworker" />

addEventListener('message', ({ data }) => {
  // Perform CPU-intensive task
  const result = performHeavyCalculation(data);
  
  // Send result back to the main thread
  postMessage(result);
});

function performHeavyCalculation(data) {
  // Your CPU-intensive code here
  return `Processed: ${data.data}`;
}
```

**When to use**:
- For CPU-intensive tasks that could block the main thread
- For complex calculations, data processing, or encryption
- When you need to maintain UI responsiveness during heavy operations

## Virtual Scrolling

Virtual scrolling renders only the items currently visible in the viewport, improving performance for long lists.

### Setting Up Virtual Scrolling

First, install the CDK:

```bash
ng add @angular/cdk
```

Then import the ScrollingModule:

```typescript
// app.module.ts
import { ScrollingModule } from '@angular/cdk/scrolling';

@NgModule({
  imports: [
    // ... other imports
    ScrollingModule
  ],
  // ... rest of the module
})
export class AppModule { }
```

### Using Virtual Scrolling

```html
<!-- virtual-scroll.component.html -->
<cdk-virtual-scroll-viewport itemSize="50" class="viewport">
  <div *cdkVirtualFor="let item of items" class="item">
    {{ item }}
  </div>
</cdk-virtual-scroll-viewport>
```

```typescript
// virtual-scroll.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-virtual-scroll',
  templateUrl: './virtual-scroll.component.html',
  styleUrls: ['./virtual-scroll.component.scss']
})
export class VirtualScrollComponent {
  items = Array.from({length: 100000}).map((_, i) => `Item #${i}`);
}
```

```scss
/* virtual-scroll.component.scss */
.viewport {
  height: 400px;
  width: 100%;
  border: 1px solid black;
}

.item {
  height: 50px;
  padding: 10px;
  border-bottom: 1px solid #ccc;
}
```

**When to use**:
- For long lists or tables with hundreds or thousands of items
- When rendering all items at once would cause performance issues
- For infinite scrolling implementations

## Performance Profiling

Identifying performance bottlenecks is the first step in optimization.

### Using Chrome DevTools

1. Open Chrome DevTools (F12 or Ctrl+Shift+I)
2. Go to the Performance tab
3. Click Record and interact with your application
4. Stop recording and analyze the results

### Using Angular DevTools

Angular DevTools is a Chrome extension that provides Angular-specific debugging and profiling tools:

1. Install the [Angular DevTools extension](https://chrome.google.com/webstore/detail/angular-devtools/ienfalfjdbdpebioblfackkekamfmbnh)
2. Open Chrome DevTools
3. Go to the Angular tab
4. Use the Profiler to record and analyze performance

### Using Lighthouse

Lighthouse is an automated tool for improving web page quality:

1. Open Chrome DevTools
2. Go to the Lighthouse tab
3. Select the categories you want to audit
4. Click "Generate report"

**When to use**:
- Regularly profile your application during development
- Use Chrome DevTools for general performance profiling
- Use Angular DevTools for Angular-specific profiling
- Use Lighthouse for overall application quality assessment

## Best Practices

### General Best Practices

1. **Use Production Builds**: Always use production builds for deployment
   ```bash
   ng build --prod
   ```

2. **Enable Gzip Compression**: Configure your server to use Gzip compression for static assets

3. **Implement Caching Strategies**: Use appropriate HTTP caching headers for your assets

4. **Optimize Images**: Use modern image formats (WebP) and responsive images

5. **Minimize Third-Party Libraries**: Only include libraries you actually need

### Angular-Specific Best Practices

1. **Use OnPush Change Detection**: Apply OnPush change detection to components that don't need frequent updates

2. **Avoid Complex Expressions in Templates**: Move complex logic to component methods or pure pipes

3. **Use TrackBy with ngFor**: Helps Angular identify which items have changed
   ```html
   <div *ngFor="let item of items; trackBy: trackByFn">{{ item.name }}</div>
   ```
   ```typescript
   trackByFn(index: number, item: any): number {
     return item.id;
   }
   ```

4. **Lazy Load Non-Critical Resources**: Use dynamic imports for libraries not needed immediately

5. **Optimize NgRx Usage**: Use entity adapters and memoized selectors for efficient state management

6. **Avoid Unnecessary Re-renders**: Be careful with object and array references to prevent unnecessary change detection

7. **Use Immutable Data Patterns**: Helps with change detection and prevents unexpected side effects

## Conclusion

Optimizing Angular applications requires a multi-faceted approach, addressing everything from initial load time to runtime performance and memory management. By implementing the techniques discussed in this post, you can significantly improve your application's performance and provide a better user experience.

Remember that performance optimization is an ongoing process. Regularly profile your application, identify bottlenecks, and apply the appropriate optimizations. Start with the simplest optimizations that provide the biggest impact, such as enabling production builds and AOT compilation, before moving on to more complex techniques like SSR or Web Workers.

By following these best practices and techniques, you can build Angular applications that are not only feature-rich but also fast and responsive.

## References

- [Angular Performance Optimization Guide](https://angular.io/guide/performance-optimization)
- [Angular Universal (SSR) Documentation](https://angular.io/guide/universal)
- [Web Workers in Angular](https://angular.io/guide/web-worker)
- [Virtual Scrolling CDK Documentation](https://material.angular.io/cdk/scrolling/overview)
- [Chrome DevTools Performance Analysis](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance)
- [Angular DevTools](https://angular.io/guide/devtools)
- [Lighthouse Documentation](https://developers.google.com/web/tools/lighthouse)
