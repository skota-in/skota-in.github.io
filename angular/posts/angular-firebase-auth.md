# Angular Firebase Authentication: Complete Guide with Router Guards

*Published on January 1, 2024*

This guide provides a comprehensive walkthrough of implementing Firebase Authentication in an Angular application, including router guards and role-based access control.

## Table of Contents
1. [Setup and Installation](#setup-and-installation)
2. [Authentication Service](#authentication-service)
3. [Router Guards](#router-guards)
4. [Admin Role Implementation](#admin-role-implementation)
5. [Best Practices](#best-practices)

## Setup and Installation

First, install the required dependencies:

```bash
ng add @angular/fire
```

During installation, you'll be prompted to:
1. Select features (choose Authentication)
2. Enter your Firebase project details
3. Configure your Firebase configuration

After installation, your `app.module.ts` will be updated with the necessary imports:

```typescript
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { getAuth, provideAuth } from '@angular/fire/auth';

@NgModule({
  imports: [
    provideFirebaseApp(() => initializeApp(environment.firebase)),
    provideAuth(() => getAuth()),
  ]
})
export class AppModule { }
```

## Authentication Service

Create an authentication service to handle all Firebase auth operations:

```typescript
import { Injectable } from '@angular/core';
import { Auth, signInWithEmailAndPassword, signOut, User } from '@angular/fire/auth';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private userSubject = new BehaviorSubject<User | null>(null);
  user$ = this.userSubject.asObservable();

  constructor(private auth: Auth) {
    this.auth.onAuthStateChanged(user => {
      this.userSubject.next(user);
    });
  }

  async login(email: string, password: string) {
    try {
      const result = await signInWithEmailAndPassword(this.auth, email, password);
      return result;
    } catch (error) {
      throw error;
    }
  }

  async logout() {
    try {
      await signOut(this.auth);
    } catch (error) {
      throw error;
    }
  }

  getCurrentUser(): User | null {
    return this.auth.currentUser;
  }
}
```

## Router Guards

Implement authentication and role-based guards:

```typescript
// auth.guard.ts
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from './auth.service';
import { map, take } from 'rxjs/operators';

export const authGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.user$.pipe(
    take(1),
    map(user => {
      if (user) {
        return true;
      }
      return router.createUrlTree(['/login']);
    })
  );
};

// admin.guard.ts
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { AuthService } from './auth.service';
import { map, take } from 'rxjs/operators';

export const adminGuard = () => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.user$.pipe(
    take(1),
    map(user => {
      if (user && user.uid === 'admin-uid') { // Replace with your admin check logic
        return true;
      }
      return router.createUrlTree(['/unauthorized']);
    })
  );
};
```

Use the guards in your routing configuration:

```typescript
const routes: Routes = [
  {
    path: 'admin',
    canActivate: [authGuard, adminGuard],
    component: AdminComponent
  },
  {
    path: 'dashboard',
    canActivate: [authGuard],
    component: DashboardComponent
  }
];
```

## Admin Role Implementation

To implement admin roles, you can use Firebase Custom Claims:

```typescript
// admin.service.ts
import { Injectable } from '@angular/core';
import { Functions, httpsCallable } from '@angular/fire/functions';

@Injectable({
  providedIn: 'root'
})
export class AdminService {
  constructor(private functions: Functions) {}

  async setAdminRole(uid: string) {
    const setAdminRole = httpsCallable(this.functions, 'setAdminRole');
    return setAdminRole({ uid });
  }

  async isAdmin(uid: string): Promise<boolean> {
    const user = await this.auth.currentUser;
    if (!user) return false;
    
    const idTokenResult = await user.getIdTokenResult();
    return idTokenResult.claims.admin === true;
  }
}
```

## Best Practices

1. **Security Rules**: Always implement proper Firebase Security Rules
2. **Error Handling**: Implement comprehensive error handling
3. **Loading States**: Show loading indicators during authentication operations
4. **Token Management**: Handle token refresh and expiration
5. **Route Protection**: Protect all sensitive routes with appropriate guards
6. **User State Management**: Use a state management solution for complex applications

## Conclusion

Implementing Firebase Authentication in Angular provides a secure and scalable solution for user authentication. By combining it with router guards and role-based access control, you can create a robust authentication system for your application.

Remember to:
- Keep your Firebase configuration secure
- Implement proper error handling
- Use appropriate loading states
- Test all authentication flows
- Monitor authentication events
