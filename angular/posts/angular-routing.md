# Angular Routing Guide

## Table of Contents
1. [Basic Routing](#basic-routing)
2. [Lazy Loading](#lazy-loading)
3. [Nested Routes](#nested-routes)
4. [Route Guards](#route-guards)
5. [Route Parameters](#route-parameters)
6. [Child Routes](#child-routes)
7. [Best Practices](#best-practices)

## Basic Routing

Angular's routing system allows you to navigate between different components in your application. Here's how to set up basic routing:

1. First, import the necessary modules in your `app.module.ts`:

```typescript
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'contact', component: ContactComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppModule { }
```

2. Add the router outlet in your `app.component.html`:

```html
<router-outlet></router-outlet>
```

3. Use routerLink for navigation:

```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/about">About</a>
  <a routerLink="/contact">Contact</a>
</nav>
```

## Lazy Loading

Lazy loading helps improve application performance by loading feature modules only when needed. Here's how to implement it:

1. Create a feature module with its own routing:

```typescript
// products.module.ts
@NgModule({
  declarations: [ProductsComponent],
  imports: [
    CommonModule,
    ProductsRoutingModule
  ]
})
export class ProductsModule { }

// products-routing.module.ts
const routes: Routes = [
  { path: '', component: ProductsComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class ProductsRoutingModule { }
```

2. Update the main routing configuration:

```typescript
const routes: Routes = [
  { 
    path: 'products',
    loadChildren: () => import('./products/products.module')
      .then(m => m.ProductsModule)
  }
];
```

## Nested Routes

Nested routes allow you to create hierarchical navigation structures:

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    children: [
      { path: 'overview', component: OverviewComponent },
      { path: 'stats', component: StatsComponent },
      { path: 'settings', component: SettingsComponent }
    ]
  }
];
```

In your `dashboard.component.html`:

```html
<h1>Dashboard</h1>
<nav>
  <a routerLink="overview">Overview</a>
  <a routerLink="stats">Stats</a>
  <a routerLink="settings">Settings</a>
</nav>
<router-outlet></router-outlet>
```

## Route Guards

Route guards help control access to routes. Here's an example of an authentication guard:

```typescript
@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): boolean {
    return this.authService.isAuthenticated();
  }
}
```

Use the guard in your routes:

```typescript
const routes: Routes = [
  { 
    path: 'admin',
    component: AdminComponent,
    canActivate: [AuthGuard]
  }
];
```

## Route Parameters

Handle dynamic route parameters:

```typescript
const routes: Routes = [
  { path: 'products/:id', component: ProductDetailComponent }
];
```

Access parameters in your component:

```typescript
export class ProductDetailComponent implements OnInit {
  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.params.subscribe(params => {
      const productId = params['id'];
      // Fetch product details
    });
  }
}
```

## Child Routes

Child routes allow you to create more complex navigation structures:

```typescript
const routes: Routes = [
  {
    path: 'user',
    component: UserComponent,
    children: [
      { path: 'profile', component: ProfileComponent },
      { path: 'orders', component: OrdersComponent },
      { path: 'settings', component: UserSettingsComponent }
    ]
  }
];
```

## Best Practices

1. **Organize Routes**: Group related routes together and use feature modules
2. **Lazy Loading**: Implement lazy loading for feature modules to improve performance
3. **Route Guards**: Use guards to protect sensitive routes
4. **Error Handling**: Implement a wildcard route for 404 pages
5. **Route Parameters**: Use route parameters for dynamic content
6. **Nested Routes**: Use nested routes for complex navigation structures
7. **Route Resolvers**: Use resolvers to pre-fetch data before component activation

Example of a wildcard route:

```typescript
const routes: Routes = [
  // ... other routes
  { path: '**', component: PageNotFoundComponent }
];
```

Remember to always handle navigation errors and provide appropriate feedback to users when routes are not found or when they don't have access to certain routes.
