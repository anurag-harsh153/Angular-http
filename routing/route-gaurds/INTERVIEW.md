# Angular Route Guards: Interview Questions

These questions cover various aspects of Angular Route Guards, ranging from basic concepts to advanced scenarios and architectural considerations.

## The "Basics"

1.  **What is a Route Guard in Angular?**
    *   **Star Answer**: A Route Guard in Angular is a feature that allows you to control the navigation and activation lifecycle of routes. It's essentially a piece of code that executes before or after a route change, making decisions about whether a user can access a route, leave a route, or even if a lazy-loaded module should be loaded. They are crucial for implementing authentication, authorization, and data pre-fetching.

2.  **What are the different types of route guards? Explain `CanActivate` and `CanDeactivate`.**
    *   **Star Answer**: Angular provides several types of route guards:
        *   **`CanActivate`**: Determines if a route can be activated. It's commonly used for authentication, checking if a user is logged in before allowing access to a protected route.
        *   **`CanActivateChild`**: Determines if a child route can be activated. Useful for applying a guard to all child routes of a parent route.
        *   **`CanDeactivate`**: Determines if a user can navigate away from the current route. It's often used to prompt users about unsaved changes in a form before allowing them to leave the page.
        *   **`CanMatch`**: Determines if a route segment can be matched. This guard runs very early in the matching process and is ideal for conditional lazy loading of modules based on roles or feature flags, replacing the deprecated `CanLoad`.
        *   **`Resolve`**: Pre-fetches data before a route is activated. This ensures that the necessary data is available before the component is rendered, preventing flashes of incomplete UI.

3.  **How do you implement a functional route guard? What are its advantages over class-based guards?**
    *   **Star Answer**: Functional route guards were introduced in Angular 14.2 (and further refined). They are simple functions that return `boolean | UrlTree | Observable<boolean | UrlTree> | Promise<boolean | UrlTree>`. They leverage the `inject()` function to get dependencies.
    ```typescript
    // auth.guard.ts
    import { CanActivateFn, Router } from '@angular/router';
    import { inject } from '@angular/core';
    import { AuthService } from './auth.service';

    export const authGuard: CanActivateFn = (route, state) => {
      const authService = inject(AuthService);
      const router = inject(Router);

      if (authService.isLoggedIn()) {
        return true;
      } else {
        router.navigate(['/login']);
        return false;
      }
    };
    ```
    *   **Advantages**:
        *   **Tree-shakable**: Unused functional guards are easily removed by the build optimizer.
        *   **Concise**: Less boilerplate code compared to class-based guards (no class definition, constructor, or `@Injectable()` decorator).
        *   **Modern DI**: Uses `inject()` directly, which is generally preferred in newer Angular versions.
        *   **Easier Testing**: Can be easier to test as they are pure functions (though `inject` still needs mock dependencies).

## The "Scenario"

4.  **A user reports that after logging in, they are sometimes redirected back to the login page when trying to access their dashboard. The app uses a guard that checks the user's auth token. What could be the cause of this race condition, and how would you fix it?**
    *   **Star Answer**: This sounds like a race condition where the `AuthGuard` executes *before* the authentication state (e.g., auth token or user profile) has been fully loaded or verified after login.
        *   **Cause**: The `AuthService` might be asynchronously fetching or validating the token (e.g., from `localStorage` or a backend API). If the guard checks `isLoggedIn()` synchronously or before the asynchronous operation completes, it might incorrectly report `false`.
        *   **Fix**: The `AuthGuard` needs to handle asynchronous operations properly. Instead of returning a `boolean` directly, it should return an `Observable<boolean | UrlTree>` or `Promise<boolean | UrlTree>`. The `AuthService.isLoggedIn()` method (or a similar method) should emit a value only when the authentication state is definitively known.
        ```typescript
        // auth.service.ts
        // ... (example with a BehaviorSubject for login status)
        private _isLoggedIn = new BehaviorSubject<boolean>(false);
        isLoggedIn$ = this._isLoggedIn.asObservable(); // Observable for login status

        // In login process, once token/status is confirmed:
        this._isLoggedIn.next(true);

        // auth.guard.ts (Functional Guard)
        export const authGuard: CanActivateFn = (route, state) => {
          const authService = inject(AuthService);
          const router = inject(Router);

          return authService.isLoggedIn$.pipe( // Use the Observable
            take(1), // Take only the first emission and complete
            map(loggedIn => {
              if (loggedIn) {
                return true;
              } else {
                return router.createUrlTree(['/login']); // Use UrlTree for redirection
              }
            })
          );
        };
        ```
        This ensures the guard waits for the definitive login status before making a decision.

5.  **You have a complex multi-step form. If the user accidentally closes the browser tab, their progress is lost. How can you use a guard to prevent this and warn them?**
    *   **Star Answer**: I would use a `CanDeactivate` guard in conjunction with a custom interface for the form component.
        1.  **`CanComponentDeactivate` Interface**: Define an interface that the form component will implement, exposing a method like `canDeactivate()`.
            ```typescript
            interface CanComponentDeactivate {
              canDeactivate: () => Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree;
            }
            ```
        2.  **Form Component Implementation**: The form component (`MyFormComponent`) would implement `CanComponentDeactivate` and track if there are unsaved changes. The `canDeactivate()` method would prompt the user for confirmation (e.g., using `confirm()`) if `hasUnsavedChanges` is true.
            ```typescript
            // my-form.component.ts
            export class MyFormComponent implements CanComponentDeactivate {
              hasUnsavedChanges = false; // Set to true on form changes

              canDeactivate(): boolean {
                if (this.hasUnsavedChanges) {
                  return confirm('You have unsaved changes. Discard them?');
                }
                return true;
              }
            }
            ```
        3.  **`CanDeactivate` Guard**: The guard would be a functional guard that expects a component implementing `CanComponentDeactivate`. It calls the component's `canDeactivate()` method.
            ```typescript
            // confirm-deactivate.guard.ts
            import { CanDeactivateFn } from '@angular/router';
            // ... import CanComponentDeactivate interface

            export const confirmDeactivateGuard: CanDeactivateFn<CanComponentDeactivate> = (component) => {
              return component.canDeactivate ? component.canDeactivate() : true;
            };
            ```
        4.  **Route Configuration**: Apply this guard to the form route.
            ```typescript
            const routes: Routes = [
              { path: 'my-form', component: MyFormComponent, canDeactivate: [confirmDeactivateGuard] },
            ];
            ```
        This prevents accidental data loss and provides a good user experience.

## The "Comparison"

6.  **What is the difference between `CanActivate` and `CanMatch`? When would you use one over the other?**
    *   **Star Answer**:
        *   **`CanActivate`**: This guard determines if a *matched* route can be *activated*. It runs after the router has successfully identified a route configuration that matches the current URL. If `CanActivate` returns `false`, the component for that route is not rendered, and the user remains on the previous page or is redirected. The associated lazy-loaded module (if any) *will still have been loaded* before `CanActivate` runs.
        *   **`CanMatch`**: This guard determines if a route segment can be *matched at all*. It runs much earlier in the routing process, even before the router attempts to load the associated module for lazy routes. If `CanMatch` returns `false`, the router will continue searching for other matching routes; if none are found, it leads to a 404. The lazy-loaded module *will not be loaded* if `CanMatch` returns `false`.
        *   **When to use**:
            *   Use **`CanActivate`** for general access control (e.g., user is authenticated) when preventing activation of an already loaded route is sufficient.
            *   Use **`CanMatch`** for authorization-based conditional loading of lazy modules, feature toggling, or A/B testing. It's superior for authorization as it prevents unauthorized code from being downloaded to the client, enhancing security and performance. `CanMatch` effectively replaced `CanLoad`.

7.  **Compare the `Resolve` guard with fetching data directly in a component's `ngOnInit`. What are the pros and cons of each approach?**
    *   **Star Answer**:
        *   **`Resolve` Guard**:
            *   **Pros**: Ensures all necessary data is available *before* the component is activated and rendered. This prevents "flickering" or incomplete UI states, providing a smoother user experience ("resolver first" approach). Centralizes data fetching logic for a route.
            *   **Cons**: Can delay route activation if data fetching is slow, leading to perceived latency. If not handled well, a failed `resolve` can block navigation or crash the app. Requires explicit handling for loading indicators if data takes time.
        *   **Fetching data in `ngOnInit`**:
            *   **Pros**: The route activates immediately, and the component renders quickly. You can easily display loading spinners or skeleton screens while data is being fetched, giving immediate feedback to the user. Simpler to implement for basic data needs.
            *   **Cons**: Can lead to incomplete UI states if the component renders before data is available. Requires the component to manage its own loading state and error handling. If not careful, multiple components might fetch the same data.
        *   **Conclusion**: For critical data that a component absolutely *needs* to render meaningfully, `Resolve` is often preferred. For supplementary data or when immediate UI feedback with loading states is a priority, `ngOnInit` fetching is suitable. The choice often depends on the specific UX requirements and data criticality.

## The "Senior/Architect"

8.  **How would you design a scalable permissions system for a large enterprise application with dozens of user roles and permissions, using guards?**
    *   **Star Answer**: A scalable permissions system needs to be centralized, configurable, and efficient.
        1.  **Centralized Permission Service**: Create a dedicated `PermissionService` (or `AuthService`) responsible for fetching and managing user permissions/roles, typically from a backend API. This service would expose methods like `hasPermission(permissionKey: string)` or `hasAnyRole(roles: string[])`. It should cache permissions to avoid repeated API calls.
        2.  **Route Data for Permissions**: Define required permissions directly within the route's `data` property. This makes the route configuration declarative and easy to audit.
            ```typescript
            const routes: Routes = [
              {
                path: 'admin/users',
                component: UserManagementComponent,
                canActivate: [permissionGuard],
                data: { permissions: ['user:read', 'user:write'] }
              },
              {
                path: 'reports/financial',
                component: FinancialReportComponent,
                canMatch: [permissionGuard], // Using CanMatch for lazy modules
                data: { permissions: ['report:financial:view'] }
              }
            ];
            ```
        3.  **Generic `PermissionGuard`**: Implement a single, generic `CanActivate` (and/or `CanMatch`) guard that injects the `PermissionService`. This guard would:
            *   Retrieve the `permissions` array from `route.data`.
            *   Call `permissionService.hasAnyPermission(requiredPermissions)`.
            *   Return `true` if authorized, otherwise `false` (and potentially navigate to an unauthorized page or display a toast).
            *   Handle asynchronous permission checks (e.g., if permissions are fetched lazily) by returning an `Observable<boolean | UrlTree>`.
        4.  **UI Level Control**: Complement route guards with UI-level permission checks (e.g., `*ngIf="permissionService.hasPermission('button:delete')"`) to hide/show elements, preventing users from even attempting unauthorized actions.
        5.  **Backend Enforcement**: Emphasize that route guards are a client-side security measure. All sensitive actions and data access *must* also be validated on the backend for true security.

9.  **Explain how a memory leak can occur in a class-based guard and how to prevent it.**
    *   **Star Answer**: A memory leak can occur in a class-based guard if it subscribes to an observable (e.g., from a service) and that subscription is not properly unsubscribed when the guard instance is no longer needed. While guard instances might be singletons (if `providedIn: 'root'`), they could still hold references that prevent garbage collection if their observables are never completed or unsubscribed.
        *   **Scenario**: If a `CanActivate` guard subscribes to an `AuthService.authStatus$` observable that never completes, and the guard itself is not properly disposed of (which is rare for singletons, but possible in specific DI setups or if the observable chain creates new subscriptions per check), the subscription might persist, holding references.
        *   **Prevention**:
            *   **`take(1)`**: For guards that perform a one-time check (e.g., authentication status at the moment of navigation), using `take(1)` on the observable stream ensures the subscription automatically completes after the first value.
            ```typescript
            import { take } from 'rxjs/operators';
            // ...
            return this.authService.authStatus$.pipe(
              take(1), // Prevents infinite subscription
              map(status => { /* ... */ })
            );
            ```
            *   **Functional Guards**: Functional guards inherently reduce the risk of memory leaks as they execute once and complete, not maintaining state or long-lived subscriptions.
            *   **`async` pipe**: While not directly applicable to guards themselves (which return `boolean` or `UrlTree`), within components, `async` pipe handles subscriptions automatically, preventing leaks.

10. **Can a functional guard use `inject()`? How does this change dependency injection for guards, and what are the implications for testing?**
    *   **Star Answer**: Yes, a functional guard **can and should use `inject()`** to obtain dependencies. This is one of their primary advantages.
        ```typescript
        import { CanActivateFn, Router } from '@angular/router';
        import { inject } from '@angular/core'; // <-- The inject function
        import { AuthService } from './auth.service';

        export const authGuard: CanActivateFn = (route, state) => {
          const authService = inject(AuthService); // <-- Using inject()
          const router = inject(Router);
          // ... logic
        };
        ```
        *   **Change in DI**:
            *   It **eliminates the need for class constructors** in guards just for dependency injection. This makes the guard more concise and less boilerplate-heavy.
            *   It removes the requirement for the `@Injectable()` decorator and explicitly providing the guard in an `NgModule` (unless it's a class-based guard that *also* uses `inject()`).
            *   Dependencies are resolved at the time the guard function is invoked within the injection context of the route.
        *   **Implications for Testing**:
            *   Testing functional guards becomes potentially simpler, as they are essentially pure functions. You can often test the core logic by calling the function directly with mocked dependencies.
            *   However, when `inject()` is used, you still need to ensure that an appropriate `TestBed` or `inject()` context is available for Angular to resolve the injected services during unit tests. You would typically set up a `TestBed` and then use `TestBed.runInInjectionContext(() => myGuard(route, state))` or similar, providing mocks for the injected services. This makes testing more aligned with how other Angular constructs are tested.

Good luck!