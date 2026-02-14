# Angular Route Guards

## Core Concept

Angular Route Guards are interfaces that a class can implement to decide if the router can activate a route, deactivate a route, load a lazy-loaded module, or even match a route. They are crucial for implementing authentication, authorization, and data pre-fetching logic in an Angular application.

Route Guards are typically used for:
-   Preventing unauthorized users from accessing certain routes.
-   Confirming with users before they leave a page with unsaved changes.
-   Fetching data before a component is activated to ensure a complete UI upon loading.
-   Conditionally loading lazy modules based on specific criteria.

### Types of Route Guards

Angular provides several types of route guards, each serving a specific purpose:

1.  **`CanActivate`**: Controls if a route can be activated. Useful for authentication checks.
    *   Signature (Class-based): `canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree`
    *   Signature (Functional Guard - preferred since Angular 14.2): `CanActivateFn`

2.  **`CanActivateChild`**: Controls if child routes can be activated. Useful for applying a guard to all children of a parent route.
    *   Signature (Class-based): `canActivateChild(childRoute: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree`
    *   Signature (Functional Guard): `CanActivateChildFn`

3.  **`CanDeactivate`**: Controls if a route can be deactivated (i.e., if the user can navigate away from the current route). Useful for prompting users about unsaved changes.
    *   Signature (Class-based): `canDeactivate(component: C, currentRoute: ActivatedRouteSnapshot, currentState: RouterStateSnapshot, nextState?: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree`
    *   Signature (Functional Guard): `CanDeactivateFn`
    *   `C` is the type of the component being deactivated.

4.  **`CanMatch`**: Controls if a route can be matched. This guard runs *before* `CanActivate` and is particularly useful for feature toggling or conditional loading of lazy modules, preventing the module from being loaded if the guard returns `false`.
    *   Signature (Functional Guard - preferred since Angular 15.2): `CanMatchFn`
    *   *Note*: `CanLoad` (which controlled if a lazy-loaded module could be loaded) is deprecated in favor of `CanMatch`.

5.  **`Resolve`**: Pre-fetches data before a route is activated. This ensures that a component is only rendered once all necessary data is available, preventing flashes of incomplete UI.
    *   Signature (Class-based): `resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<T> | Promise<T> | T`
    *   Signature (Functional Guard - preferred since Angular 15.2): `ResolveFn`
    *   `T` is the type of data being resolved.

### Functional Route Guards (Angular 14.2+)

Functional guards are the modern and recommended way to implement guards. They are tree-shakable, more concise, and leverage Angular's `inject` function for dependency injection, eliminating the need for boilerplate class structures.

```typescript
// auth.guard.ts
import { CanActivateFn, Router } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './auth.service'; // Assume an AuthService for authentication logic

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isLoggedIn()) {
    return true; // Allow activation
  } else {
    // Redirect to login page and prevent activation
    router.navigate(['/login']);
    return false;
  }
};

// In your app-routing.module.ts
const routes: Routes = [
  { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] },
  // ...
];
```

### Functional `CanActivateChild` Guard

Controls if child routes within a parent route can be activated. Useful for applying a common guard to all children of a parent.

```typescript
// auth-child.guard.ts
import { CanActivateChildFn, Router, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './auth.service'; // Assume an AuthService for permission logic

export const authChildGuard: CanActivateChildFn = (childRoute: ActivatedRouteSnapshot, state: RouterStateSnapshot) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Example: Check if the user has permission to access this specific child route
  if (authService.hasPermissionForRoute(childRoute.routeConfig?.path || '')) {
    return true;
  } else {
    // Redirect to an unauthorized page if access is denied
    return router.createUrlTree(['/forbidden']);
  }
};

// In your app-routing.module.ts or feature-routing.module.ts for a parent route:
const routes: Routes = [
  {
    path: 'admin',
    component: AdminParentComponent, // This component must have a <router-outlet>
    canActivateChild: [authChildGuard], // Apply the guard to all children
    children: [
      { path: 'settings', component: AdminSettingsComponent },
      { path: 'users', component: AdminUsersComponent }
    ]
  },
  { path: 'forbidden', component: ForbiddenComponent }, // Component to show when access is denied
  // ...
];
```

### Functional `CanDeactivate` Guard

Used to prevent users from navigating away from a route, typically to confirm unsaved changes. The component associated with the route must implement a specific interface (e.g., `CanComponentDeactivate`) for the guard to interact with it.

```typescript
// confirm-deactivate.guard.ts
import { CanDeactivateFn, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';

// Define the interface that components needing CanDeactivate protection must implement
export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree;
}

export const confirmDeactivateGuard: CanDeactivateFn<CanComponentDeactivate> = (
  component: CanComponentDeactivate,
  currentRoute,
  currentState,
  nextState
) => {
  // If the component has a canDeactivate method, call it. Otherwise, allow deactivation.
  return component.canDeactivate ? component.canDeactivate() : true;
};

```typescript


// Example Component that uses the guard (e.g., a form with unsaved changes)
/*
// user-profile-edit.component.ts


import { Component } from '@angular/core';
import { CanComponentDeactivate } from '../confirm-deactivate.guard'; // Adjust path

@Component({
  selector: 'app-user-profile-edit',
  template: `
    <h2>Edit Profile</h2>
    <input type="text" [(ngModel)]="userName" (ngModelChange)="hasUnsavedChanges = true">
    <button (click)="saveChanges()">Save</button>
    <p *ngIf="hasUnsavedChanges">You have unsaved changes!</p>
  `
})
export class UserProfileEditComponent implements CanComponentDeactivate {
  userName: string = 'John Doe';
  hasUnsavedChanges: boolean = false;

  saveChanges(): void {
    // Logic to save changes
    this.hasUnsavedChanges = false;
    alert('Changes saved!');
  }

  canDeactivate(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('WARNING: You have unsaved changes. Press Cancel to stay, OK to discard changes.');
    }
    return true;
  }
}
*/

// In your app-routing.module.ts or feature-routing.module.ts:
const routes: Routes = [
  {
    path: 'edit-profile',
    component: UserProfileEditComponent, // This component must implement CanComponentDeactivate
    canDeactivate: [confirmDeactivateGuard]
  },
  // ...
];
```

### Functional `CanMatch` Guard

Controls if a route segment can be matched. This guard runs very early in the routing process, *before* `CanActivate` and *before* a lazy-loaded module is even downloaded. It's ideal for feature toggling, A/B testing, or conditional module loading based on user roles or permissions, as it prevents unauthorized code from being sent to the client.

```typescript
// feature-flag.guard.ts
import { CanMatchFn, Route, UrlSegment, UrlTree, Router } from '@angular/router';
import { inject } from '@angular/core';
import { FeatureToggleService } from './feature-toggle.service'; // Assume a service for feature flags

export const featureFlagGuard: CanMatchFn = (route: Route, segments: UrlSegment[]) => {
  const featureToggleService = inject(FeatureToggleService);
  const router = inject(Router);

  // Example: Check a feature flag from route data to decide if the module should be matched
  const featureName = route.data?.['featureName'] as string;
  if (featureName && featureToggleService.isFeatureEnabled(featureName)) {
    return true; // Allow module to be loaded and matched
  } else {
    // Option 1: Redirect to a specific page if the feature is disabled
    return router.createUrlTree(['/feature-disabled']);
    // Option 2: Simply return false, and the router will continue looking for other routes that match
    // return false;
  }
};

// In your app-routing.module.ts (for a lazy-loaded module)
const routes: Routes = [
  {
    path: 'beta-feature',
    loadChildren: () => import('./beta-feature/beta-feature.module').then(m => m.BetaFeatureModule),
    canMatch: [featureFlagGuard], // Apply CanMatch here
    data: { featureName: 'betaFeatureEnabled' } // Pass data to the guard
  },
  { path: 'feature-disabled', component: FeatureDisabledComponent },
  // ...
];
```

### Functional `Resolve` Guard

Pre-fetches data before a route is activated, ensuring that the component only renders once all necessary data is available. This prevents flashes of incomplete UI and improves user experience by guaranteeing data presence.

```typescript
// user.resolver.ts
import { ResolveFn, Router, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { inject } from '@angular/core';
import { UserService } from './user.service'; // Assume a service to fetch user data
import { Observable, of } from 'rxjs';
import { catchError } from 'rxjs/operators';

// Define an interface for the data being resolved, e.g., a User object
interface User {
  id: string;
  name: string;
  email: string;
}

export const userResolver: ResolveFn<User | null> = (
  route: ActivatedRouteSnapshot,
  state: RouterStateSnapshot
) => {
  const userService = inject(UserService);
  const router = inject(Router);
  const userId = route.paramMap.get('id');

  if (userId) {
    return userService.getUser(userId).pipe(
      catchError(error => {
        console.error('Error fetching user data:', error);
        // Optionally redirect to an error page or return null to prevent navigation
        router.navigate(['/error-page']);
        return of(null); // Return an Observable of null to complete the resolver
      })
    );
  }
  // If no userId, or invalid, redirect or handle appropriately
  router.navigate(['/users']);
  return of(null); // Prevent activation
};

// In your app-routing.module.ts or feature-routing.module.ts:
const routes: Routes = [
  {
    path: 'user/:id',
    component: UserProfileComponent,
    resolve: { userData: userResolver } // 'userData' will be available via ActivatedRoute.data
  },
  { path: 'users', component: UserListComponent }, // Example list to redirect to
  { path: 'error-page', component: ErrorPageComponent }, // Example error page
  // ...
];
```typescript

// In UserProfileComponent (to access the resolved data):
/*
// user-profile.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { User } from './user.resolver'; // Import the User interface

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user">
      <h3>User Profile</h3>
      <p>ID: {{ user.id }}</p>
      <p>Name: {{ user.name }}</p>
      <p>Email: {{ user.email }}</p>
    </div>
    <div *ngIf="!user">
      <p>User data not found or could not be loaded.</p>
    </div>
  `
})
export class UserProfileComponent implements OnInit {
  user: User | null = null;

  constructor(private route: ActivatedRoute) { }

  ngOnInit(): void {
    this.route.data.subscribe(data => {
      this.user = data['userData']; // Access the resolved data using the key 'userData'
    });
  }
}
*/
```

### Class-based Route Guards (Older approach, still valid)

While functional guards are preferred, you might encounter class-based guards in older projects. They require the guard service to be provided.

```typescript
// auth.guard.ts (Class-based)
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot, UrlTree, Router } from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from './auth.service';

@Injectable({
  providedIn: 'root' // Or provide in a specific module
})
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
    if (this.authService.isLoggedIn()) {
      return true;
    } else {
      this.router.navigate(['/login']);
      return false;
    }
  }
}

// In your app-routing.module.ts
const routes: Routes = [
  { path: 'dashboard', component: DashboardComponent, canActivate: [AuthGuard] },
  // ...
];
```

## The "Industry" Way

In large enterprise applications, Route Guards are integral to security, user experience, and performance.

-   **Role-Based Access Control (RBAC)**: Guards are fundamental for implementing RBAC. A `CanActivate` or `CanMatch` guard can check a user's roles against roles defined in the route's `data` property (`data: { roles: ['ADMIN', 'EDITOR'] }`) to determine access. This is a common pattern for securing administrative panels or specific features.

    ```typescript
    // role.guard.ts (Functional CanActivate for RBAC)
    import { CanActivateFn, Router } from '@angular/router';
    import { inject } from '@angular/core';
    import { AuthService } from './auth.service';

    export const roleGuard: CanActivateFn = (route, state) => {
      const authService = inject(AuthService);
      const router = inject(Router);
      const requiredRoles = route.data['roles'] as string[]; // Get roles from route data

      if (authService.isLoggedIn() && authService.hasAnyRole(requiredRoles)) {
        return true;
      } else {
        router.navigate(['/unauthorized']); // Or redirect to login
        return false;
      }
    };

    // In app-routing.module.ts
    const routes: Routes = [
      {
        path: 'admin',
        component: AdminDashboardComponent,
        canActivate: [roleGuard],
        data: { roles: ['ADMIN'] } // Define required roles
      },
      // ...
    ];
    ```

-   **Asynchronous Guards**: Real-world guards often need to interact with backend services (e.g., to verify a session, fetch user permissions). This means guards will return an `Observable<boolean | UrlTree>` or `Promise<boolean | UrlTree>`. It's crucial to handle loading states and potential errors gracefully. Using RxJS operators like `take(1)` and `catchError` is common.

-   **Combining Multiple Guards**: You can apply multiple guards to a single route. Angular will execute them in the order they are provided in the `canActivate` array. If any guard returns `false` or `UrlTree`, subsequent guards are skipped.

-   **`CanMatch` for Conditional Module Loading/Feature Flagging**: The `CanMatch` guard (demonstrated in the "Functional Route Guards" section) is powerful for truly conditional feature availability. By preventing unauthorized or unneeded modules from even being matched, it offers significant security and performance benefits. If a user doesn't have access, the associated lazy module's bundle is never downloaded, saving bandwidth, reducing load times, and preventing client-side inspection of unauthorized code. This makes it ideal for feature toggling, A/B testing, or rolling out features incrementally in a controlled manner.

-   **Data Pre-fetching with `Resolve`**: The `Resolve` guard (as shown in the "Functional Route Guards" section) is critical for ensuring a smooth user experience by guaranteeing that all necessary data is available *before* a component is activated. This pattern prevents "flickering" or showing incomplete UI while data loads asynchronously. In enterprise applications, `Resolve` is frequently used for fetching initial configuration settings, user profiles, or dashboard data to ensure the view loads fully populated and ready for interaction.

## Common Pitfalls

-   **Forgetting to Provide Guards**: For class-based guards, if `providedIn: 'root'` is not used, the guard class must be added to the `providers` array of an `NgModule`. Functional guards do not require this.
-   **Complex Logic in Guards**: Guards should be lean and focused on their primary decision (e.g., isAuthenticated, hasRole). Complex data fetching or business logic should reside in services, which guards then inject and call.
-   **Subscription Leaks in Class-based Guards**: If a class-based guard subscribes to an observable (e.g., from an authentication service) and doesn't unsubscribe, it can lead to memory leaks if the guard instance persists. Functional guards, by their nature, generally avoid this as they execute and complete. For class-based, use `take(1)` for one-time checks.
-   **Poor User Experience without Feedback**: If a guard blocks navigation, the user might perceive the application as unresponsive. Always provide feedback, such as a redirect to a login page, an unauthorized access page, or a clear message.
-   **`CanDeactivate` without Component Interface**: When using `CanDeactivate`, it's best practice for the component being guarded to implement a specific interface (e.g., `CanComponentDeactivate`) with a `canDeactivate` method. This allows the guard to safely call `component.canDeactivate()` and check for unsaved changes.
    ```typescript
    // can-component-deactivate.interface.ts
    export interface CanComponentDeactivate {
      canDeactivate: () => Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree;
    }

    // my-form.component.ts
    export class MyFormComponent implements CanComponentDeactivate {
      hasUnsavedChanges = true; // For example

      canDeactivate(): boolean {
        if (this.hasUnsavedChanges) {
          return confirm('You have unsaved changes. Do you want to leave?');
        }
        return true;
      }
    }
    ```
-   **Blocking UX with Slow Resolvers**: A slow `Resolve` guard will delay the activation of a route, potentially making the application feel sluggish. Use loading indicators for pages that rely on `Resolve` if the data fetching is expected to take time. Consider if `Resolve` is truly necessary or if fetching data within `ngOnInit` with a loading spinner might provide a better perceived performance.
-   **Overusing Guards**: While powerful, not every piece of conditional logic needs to be a guard. Sometimes, hiding/showing elements with `*ngIf` or handling permissions within a component is more appropriate. Guards are best for controlling access to *routes* themselves.