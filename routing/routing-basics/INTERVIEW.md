# Angular Routing: Interview Questions

These questions cover various aspects of Angular Routing, ranging from basic concepts to advanced scenarios and architectural considerations.

## The "Basics"

1.  **What is the purpose of Angular Routing?**
    *   **Star Answer**: Angular Routing is essential for building Single-Page Applications (SPAs). It allows developers to define navigation paths within the application, mapping URLs to specific components and views without requiring full page reloads. This provides a fluid user experience similar to desktop applications while managing browser history. It's handled by the `@angular/router` module.

2.  **Explain the key directives and services used in Angular Routing (`RouterOutlet`, `RouterLink`, `ActivatedRoute`, `Router`).**
    *   **Star Answer**:
        *   `RouterOutlet`: A directive (`<router-outlet>`) that acts as a placeholder in the HTML template where the router injects the component corresponding to the currently active route. An application can have multiple `RouterOutlets` for nested routing.
        *   `RouterLink`: A directive (`[routerLink]`) used to create navigation links. It prevents the browser's default full-page reload behavior and instead uses the Angular Router to navigate within the application.
        *   `ActivatedRoute`: An injectable service that provides access to information about a route associated with a component that is loaded in a `RouterOutlet`. This includes route parameters (`paramMap`), query parameters (`queryParamMap`), fragment, and route `data`.
        *   `Router`: An injectable service (`Router`) that allows programmatic navigation, provides information about the current router state, and allows interaction with router events.

3.  **How do you define basic routes in Angular? Provide an example.**
    *   **Star Answer**: Basic routes are defined as an array of `Routes` objects, typically within an `AppRoutingModule` or feature routing module. Each object specifies a `path` and the `component` to render. For the root module, `RouterModule.forRoot()` is used, and for feature modules, `RouterModule.forChild()`.
    ```typescript
    // app-routing.module.ts
    import { NgModule } from '@angular/core';
    import { RouterModule, Routes } from '@angular/router';
    import { HomeComponent } from './home/home.component';
    import { AboutComponent } from './about/about.component';
    import { NotFoundComponent } from './not-found/not-found.component';

    const routes: Routes = [
      { path: '', redirectTo: '/home', pathMatch: 'full' }, // Redirect root to home
      { path: 'home', component: HomeComponent },
      { path: 'about', component: AboutComponent },
      { path: '**', component: NotFoundComponent } // Wildcard route for 404
    ];

    @NgModule({
      imports: [RouterModule.forRoot(routes)],
      exports: [RouterModule]
    })
    export class AppRoutingModule { }
    ```

## The "Scenario"

4.  **A client wants to implement a dashboard with multiple sections (e.g., settings, reports, users). Each section should have its own set of sub-routes. How would you structure the routing for this, ensuring modularity and maintainability?**
    *   **Star Answer**: I would use **child routes** and **feature modules with their own routing modules**.
        1.  **Main Dashboard Route**: Define a top-level route for the dashboard (e.g., `/dashboard`) which loads a `DashboardComponent`. This `DashboardComponent` would contain a `router-outlet` for its child routes.
        2.  **Feature Modules for Sections**: For each section (settings, reports, users), I would create a separate lazy-loaded feature module (e.g., `SettingsModule`, `ReportsModule`, `UsersModule`). Each of these modules would have its own routing module (`SettingsRoutingModule`, etc.) where their specific sub-routes are defined.
        3.  **Lazy Loading Child Modules**: The main `AppRoutingModule` would then define the child routes of the dashboard, lazy-loading each feature module:
            ```typescript
            // app-routing.module.ts
            const routes: Routes = [
              {
                path: 'dashboard',
                component: DashboardComponent,
                children: [
                  { path: '', redirectTo: 'overview', pathMatch: 'full' },
                  { path: 'overview', component: DashboardOverviewComponent },
                  { path: 'settings', loadChildren: () => import('./settings/settings.module').then(m => m.SettingsModule) },
                  { path: 'reports', loadChildren: () => import('./reports/reports.module').then(m => m.ReportsModule) },
                  { path: 'users', loadChildren: () => import('./users/users.module').then(m => m.UsersModule) },
                ]
              }
            ];
            ```
        This approach ensures excellent modularity, promotes lazy loading for better performance, and makes the routing configuration easy to maintain as the application grows.

5.  **You have a complex application with many feature modules. The initial load time is slow. How would you optimize the loading of these modules using routing techniques?**
    *   **Star Answer**: The primary technique for optimizing initial load time in an application with many feature modules is **lazy loading**.
        1.  **Lazy Loading**: Configure feature modules to be loaded only when the user navigates to their routes using the `loadChildren` property in the route configuration. This ensures that only the necessary code for the initial view is downloaded, reducing the initial bundle size.
        2.  **Preloading Strategies**: To improve perceived performance after the initial load, I would implement **preloading strategies**:
            *   `PreloadAllModules`: This built-in strategy preloads all lazy-loaded modules in the background after the main application has bootstrapped. It balances initial load speed with readiness for subsequent navigations.
            *   **Custom Preloading Strategy**: For more fine-grained control, I would implement a custom preloading strategy. This could preload modules based on user behavior (e.g., frequently visited routes, hover over a link), network conditions (e.g., only preload on fast connections), or specific application logic. This allows us to intelligently anticipate user needs without overloading the initial download.
        Additionally, ensuring proper `pathMatch: 'full'` for redirects and well-defined wildcard routes helps prevent unnecessary routing computations.

## The "Comparison"

6.  **Explain the difference between `RouterModule.forRoot()` and `RouterModule.forChild()`. When and why would you use each?**
    *   **Star Answer**:
        *   **`RouterModule.forRoot(routes)`**: This method should be called **once** in the root `AppModule` (or its dedicated `AppRoutingModule`). It configures the root router, registers global router services (like `Router`, `ActivatedRoute`), and makes them available as singletons throughout the application. It also sets up global listeners for browser location changes.
        *   **`RouterModule.forChild(routes)`**: This method should be called in **feature modules** (lazy-loaded modules). It registers additional routes but does **not** re-register the global router services. This is crucial to prevent multiple instances of the router services, which could lead to inconsistent navigation state, performance issues, or unexpected behavior.
        *   **When to use**: Use `forRoot()` only in the application's root module. Use `forChild()` in all other feature modules, especially lazy-loaded ones, to extend the routing configuration without re-initializing global services.

7.  **Compare route parameters (`/:id`) with query parameters (`?param=value`). When would you use one over the other?**
    *   **Star Answer**:
        *   **Route Parameters (`/:id`)**: These are mandatory segments of the URL path that represent a specific resource or identifier. They are typically used for data that is essential to identify the resource being viewed (e.g., `/products/123` where `123` is the product ID). They are part of the route's hierarchical structure.
            *   **Use when**: The data is required for the route to be valid, identifies a unique resource, or contributes to the resource's hierarchy.
        *   **Query Parameters (`?param=value&another=value`)**: These are optional key-value pairs appended to the URL after a `?`. They are typically used for filtering, sorting, pagination, or other non-hierarchical, optional data that modifies the view of the current resource (e.g., `/products?category=electronics&sort=price`).
            *   **Use when**: The data is optional, affects the presentation or filtering of a resource, or is not part of the resource's unique identification.
        Route parameters are generally preferred for identifying resources, while query parameters are better for modifying or filtering lists of resources.

## The "Senior/Architect"

8.  **How would you implement a custom preloading strategy in a large-scale application, and what considerations would you take into account for performance and user experience?**
    *   **Star Answer**: A custom preloading strategy involves implementing the `PreloadingStrategy` interface (or using a functional strategy with `inject()`).
        *   **Implementation Steps**:
            1.  **Create a Service**: Define a class (or function) that implements `PreloadingStrategy` (or `PreloadAllModules` for functional approach) with a `preload` method. This method receives the route (`Route`) and a `load` function.
            2.  **Logic**: Inside `preload`, I'd implement custom logic. For instance:
                *   **Data-driven**: Use the `data` property of a route to specify if it should be preloaded (e.g., `data: { preload: true }`).
                *   **User Behavior**: Integrate analytics to identify frequently visited routes and prioritize their preloading.
                *   **Network Conditions**: Check `navigator.connection` for `effectiveType` or `downlink` to only preload on fast networks.
                *   **Idle Detection**: Use `requestIdleCallback` or RxJS operators (`debounceTime`, `auditTime`) to preload modules when the browser is idle.
            3.  **Conditional Loading**: The `preload` method returns an `Observable<any>` or `EMPTY` if the module should not be preloaded.
            4.  **Register Strategy**: Provide the custom strategy in `AppModule` and configure `RouterModule.forRoot(routes, { preloadingStrategy: CustomPreloadingStrategy })`.
        *   **Performance & UX Considerations**:
            *   **Avoid Over-Preloading**: Don't preload too many modules, as it can consume bandwidth and CPU, negatively impacting initial load or active user experience. Prioritize critical or likely-to-be-visited modules.
            *   **Throttling/Debouncing**: Implement delays or throttling to avoid resource contention, especially if preloading large modules or on slower networks.
            *   **User Feedback**: Consider visual cues (subtle loading indicators) for when preloading is active, though often it should be invisible.
            *   **Error Handling**: Gracefully handle failed preloads (e.g., network errors) without affecting the main application.
            *   **Tree Shaking**: Ensure the preloaded modules are still tree-shakable so unused code isn't included.

9.  **Discuss the different ways to pass data between routes and components. What are the pros and cons of each method (e.g., route params, query params, `history.state`, services, `resolve`)?**
    *   **Star Answer**:
        1.  **Route Parameters (`ActivatedRoute.paramMap`)**:
            *   **Pros**: Ideal for identifying specific resources (e.g., `product ID`), directly part of the URL path, readable, and often indexed by search engines.
            *   **Cons**: Primarily for mandatory, hierarchical data. Can become messy if too many parameters are used. Changing a route param often means navigating to a new route.
        2.  **Query Parameters (`ActivatedRoute.queryParamMap`)**:
            *   **Pros**: Optional, non-hierarchical data (e.g., filters, sorting), easily shareable via URL, can be manipulated without changing the primary route.
            *   **Cons**: Can make URLs long and less readable if many parameters exist. Can be overlooked if not explicitly handled.
        3.  **`history.state`**:
            *   **Pros**: Allows passing complex, non-URL-visible state data during programmatic navigation (`router.navigate(..., { state: ... })`). Not exposed in the URL.
            *   **Cons**: Data is lost on page refresh or direct URL access. Not indexable. Less explicit than other methods.
        4.  **Services**:
            *   **Pros**: Highly flexible for passing complex data between any components, regardless of routing. Data persists across route changes. Can use RxJS subjects for reactive data flow.
            *   **Cons**: Not reflected in the URL, so not bookmarkable or shareable. Can lead to tight coupling if not designed carefully.
        5.  **`resolve` (or Functional Resolvers)**:
            *   **Pros**: Ensures that data is fetched and available *before* the component is activated, preventing flashes of incomplete UI ("resolver first" approach). Centralizes data fetching logic.
            *   **Cons**: Can delay route activation if data fetching is slow. Can be overkill for simple data. Error handling needs to be robust to prevent navigation from blocking.

10. **How do you handle authentication and authorization in a routed Angular application, particularly regarding protecting routes and conditionally loading modules?**
    *   **Star Answer**:
        *   **Authentication (Protecting Routes with Guards)**:
            *   I would use **Route Guards**, specifically `CanActivate` (for general route protection) or `CanActivateChild` (for child routes).
            *   An `AuthGuard` (either class-based or functional) would check if a user is logged in. If not, it would redirect the user to the login page.
            *   Example functional `CanActivate` guard:
            ```typescript
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
            *   Apply to routes: `{ path: 'admin', component: AdminComponent, canActivate: [authGuard] }`.
        *   **Authorization (Role-Based Access Control and Conditional Loading)**:
            *   For **role-based authorization**, I would use another type of guard, typically a `CanActivate` or `CanMatch` guard, that checks the user's roles or permissions.
            *   **`CanActivate` for Component Access**: A `RoleGuard` could check if the logged-in user has the required role to access a specific component. Route `data` can be used to pass the required role:
                `{ path: 'admin-panel', component: AdminPanelComponent, canActivate: [roleGuard], data: { roles: ['ADMIN'] } }`.
            *   **`CanMatch` for Conditional Module Loading**: This is crucial for truly sensitive areas (like an `AdminModule`). `CanMatch` prevents the lazy-loaded module's code from even being downloaded if the user doesn't meet the criteria. This is superior to `CanActivate` for authorization, as `CanActivate` still loads the module before redirecting.
                ```typescript
                // admin.guard.ts (CanMatch functional guard)
                import { CanMatchFn, Router } from '@angular/router';
                import { inject } from '@angular/core';
                import { AuthService } from './auth.service';

                export const adminMatchGuard: CanMatchFn = (route, segments) => {
                  const authService = inject(AuthService);
                  const router = inject(Router);

                  if (authService.hasRole('ADMIN')) {
                    return true;
                  } else {
                    router.navigate(['/unauthorized']); // Or login page
                    return false;
                  }
                };

                // app-routing.module.ts
                const routes: Routes = [
                  {
                    path: 'admin',
                    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
                    canMatch: [adminMatchGuard]
                  }
                ];
                ```
            This combined approach ensures both authentication for route access and fine-grained authorization, including the prevention of module loading for unauthorized users.