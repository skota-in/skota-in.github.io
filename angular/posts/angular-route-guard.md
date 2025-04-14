# Angular Route Guards: Protecting Your Routes

*Published on February 10, 2024*

Route guards in Angular are interfaces that control access to routes based on certain conditions. They are essential for implementing authentication, authorization, and other route protection mechanisms in your Angular applications.

## Types of Route Guards

Angular provides several types of route guards:

1. **CanActivate**: Controls if a route can be activated
2. **CanActivateChild**: Controls if child routes can be activated
3. **CanDeactivate**: Controls if a route can be deactivated
4. **CanLoad**: Controls if a module can be lazy loaded
5. **Resolve**: Performs route data resolution before route activation

## Implementing a Basic Auth Guard

Here's an example of implementing a basic authentication guard:

```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(): boolean {
    if (this.authService.isAuthenticated()) {
      return true;
    }
    
    this.router.navigate(['/login']);
    return false;
  }
}
```

## Using Guards in Route Configuration

To use a guard in your route configuration:

```typescript
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard]
  }
];
```

## Common Use Cases

1. **Authentication Protection**
   - Prevent unauthorized access to protected routes
   - Redirect to login page if not authenticated

2. **Role-Based Access**
   - Restrict access based on user roles
   - Implement different views for different user types

3. **Form Protection**
   - Prevent users from leaving a form with unsaved changes
   - Show confirmation dialog before navigation

4. **Data Loading**
   - Ensure required data is loaded before route activation
   - Handle loading states and errors

## Best Practices

1. Keep guards focused and single-responsibility
2. Use dependency injection for services
3. Handle errors gracefully
4. Consider performance implications
5. Test guards thoroughly

## Example: Role-Based Guard

```typescript
@Injectable({
  providedIn: 'root'
})
export class RoleGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot): boolean {
    const expectedRole = route.data.expectedRole;
    const userRole = this.authService.getUserRole();
    
    if (userRole === expectedRole) {
      return true;
    }
    
    this.router.navigate(['/unauthorized']);
    return false;
  }
}
```

## Conclusion

Route guards are powerful tools for controlling navigation in your Angular applications. They help maintain security, improve user experience, and ensure data integrity. By understanding and implementing the appropriate guards, you can create more robust and secure Angular applications.
