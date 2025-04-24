# Using Multiple Route Guards in Angular 18

In Angular 18, route guards are implemented as functions using the new `canActivateFn` approach, which provides a more functional and type-safe way to protect routes. This guide demonstrates how to use multiple guards for a single route.

## Basic Guard Implementation

First, let's create a simple authentication guard:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login']);
};
```

## Role-Based Guard

Now, let's create a role-based guard:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from './auth.service';

export const adminGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.hasRole('admin')) {
    return true;
  }

  return router.createUrlTree(['/unauthorized']);
};
```

## Feature Flag Guard

Here's an example of a feature flag guard:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { FeatureFlagService } from './feature-flag.service';

export const featureGuard = (featureName: string) => {
  const featureFlagService = inject(FeatureFlagService);
  const router = inject(Router);

  if (featureFlagService.isEnabled(featureName)) {
    return true;
  }

  return router.createUrlTree(['/feature-not-available']);
};
```

## Using Multiple Guards in Routes

To use multiple guards for a route, you can combine them using the `canActivate` property in your route configuration:

```typescript
import { Routes } from '@angular/router';
import { authGuard } from './guards/auth.guard';
import { adminGuard } from './guards/admin.guard';
import { featureGuard } from './guards/feature.guard';

export const routes: Routes = [
  {
    path: 'admin/dashboard',
    component: AdminDashboardComponent,
    canActivate: [
      authGuard,
      adminGuard,
      () => featureGuard('admin-dashboard')
    ]
  }
];
```

## Guard Execution Order

Guards are executed in the order they are defined in the `canActivate` array. If any guard returns `false` or a `UrlTree`, the navigation is cancelled, and subsequent guards won't be executed.

## Best Practices

1. **Order Matters**: Place the most critical guards first (e.g., authentication before role checks)
2. **Keep Guards Focused**: Each guard should have a single responsibility
3. **Use Dependency Injection**: Leverage Angular's DI system for services
4. **Handle Errors Gracefully**: Always provide a fallback route when access is denied
5. **Type Safety**: Use TypeScript to ensure type safety in your guards

## Example with Async Guards

Here's how to implement an async guard:

```typescript
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from './auth.service';

export const asyncAuthGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.checkAuth().pipe(
    map(isAuthenticated => 
      isAuthenticated ? true : router.createUrlTree(['/login'])
    )
  );
};
```

## Testing Guards

When testing guards, you can use Angular's testing utilities:

```typescript
import { TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';

describe('AuthGuard', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterTestingModule],
      providers: [AuthService]
    });
  });

  it('should allow access when authenticated', () => {
    const authService = TestBed.inject(AuthService);
    spyOn(authService, 'isAuthenticated').and.returnValue(true);
    
    expect(authGuard()).toBe(true);
  });
});
```

## Conclusion

Angular 18's functional approach to route guards makes it easier to implement and test route protection. By combining multiple guards, you can create sophisticated authorization and access control systems while maintaining clean, maintainable code.
