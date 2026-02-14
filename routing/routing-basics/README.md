# Routing in Angular

## Core Concept

Angular's Router is a powerful module that enables the creation of Single-Page Applications (SPAs) by allowing navigation between different views without full page reloads. It maps browser URLs to specific components and manages the browser's history.

Key concepts and building blocks of Angular Routing include:

-   **`RouterModule`**: The Angular module that provides the routing functionality. You import `RouterModule.forRoot()` in your root `AppModule` and `RouterModule.forChild()` in feature modules.
-   **`Routes` array**: An array of route configurations that maps URL paths to components, redirects, or other modules.
-   **`RouterLink`**: A directive (`[routerLink]`) used in templates to create navigation links, replacing standard `href` attributes. It ensures navigation within the Angular application.
-   **`RouterOutlet`**: A directive (`<router-outlet>`) that acts as a placeholder where Angular dynamically injects the component corresponding to the current route.
-   **`RouterState`**: Represents the current state of the router, including information about the activated route tree.
-   **`ActivatedRoute`**: An injectable service that contains information about a route associated with a component that is loaded in the `RouterOutlet`. It provides access to route parameters, query parameters, fragment, and route data.

### Basic Routing Configuration

A basic `Routes` array might look like this:

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { NotFoundComponent } from './not-found/not-found.component'; // A component for 404 errors

const routes: Routes = [
  { path: '', component: HomeComponent }, // Default route
  { path: 'home', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'redirect-to-home', redirectTo: '/home', pathMatch: 'full' }, // Redirect example
  { path: '**', component: NotFoundComponent } // Wildcard route for 404 - MUST be last
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

        ```typescript
        const routes: Routes = [
          { path: '', redirectTo: '/home', pathMatch: 'full' }, // Only matches if URL is exactly '/'
          { path: 'home', component: HomeComponent },
          { path: 'users', component: UserListComponent }, // Matches /users and /users/123 (prefix)
          { path: 'users/:id', component: UserDetailComponent }, // Child concept but demonstrates prefix
        ];
        ```
  -   **`path`**: The URL path segment. An empty string `''` usually denotes the default route (e.g., your application's root). `**` is a wildcard path that matches any URL not matched by other routes.
  -   **`component`**: The component that should be rendered when this route is activated.
  -   **`redirectTo`**: Specifies a URL to redirect to. Requires `pathMatch`.
  -   **`pathMatch`**: This property defines how the router should match the URL path segments to the configured route `path`. Understanding `pathMatch` is crucial for correct routing behavior, especially with redirects.
  -   `'prefix'` (default):
      *   **Meaning**: The router checks if the URL path *starts with* the configured `path`.
      *   **When to use**: This is the default and is suitable for routes that have child routes. For example, if you have a route `{ path: 'users', component: UserListComponent }`, and the URL is `/users/123`, it will match because `/users` is a prefix of `/users/123`. The remaining part (`/123`) is then available for child routes.
      *   **Caveat**: If `pathMatch: 'prefix'` is used with a `redirectTo`, it can lead to infinite redirects or unexpected behavior if not carefully managed, as a partial match might trigger a redirect, and then the redirected path might also partially match.
  -   `'full'` (recommended for `redirectTo`):
      *   **Meaning**: The router checks if the entire URL path *exactly matches* the configured `path`. There should be no extra path segments in the URL.
      *   **When to use**:
          *   **Redirects**: Always use `pathMatch: 'full'` with `redirectTo` routes, especially for the root path (`path: ''`). This ensures that the redirect only occurs when the URL exactly matches the specified path, preventing partial matches from triggering unintended redirects.
          *   **Terminal Routes**: For routes that do not have child routes and should only match a specific URL segment exactly.
          
      If `path: '', redirectTo: '/home', pathMatch: 'prefix'` were used, navigating to `/any-other-path` would still match the empty path as a prefix and redirect to `/home`, which is likely not the desired behavior.


### Lazy Loading Modules

Lazy loading allows you to load JavaScript bundles for specific feature modules only when the user navigates to a route associated with that module. This significantly reduces the initial load time of your application.

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  {
    path: 'products',
    loadChildren: () => import('./products/products.module').then(m => m.ProductsModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

-   **`loadChildren`**: A function that uses a dynamic import to load a feature module. The module is only downloaded and parsed when the user accesses the `/products` route.

### Route Parameters, Query Parameters, and Fragments

-   **Route Parameters**: Used to pass essential data as part of the URL path.
    ```typescript
    // app-routing.module.ts
    const routes: Routes = [
      { path: 'product/:id', component: ProductDetailComponent } // :id is a route parameter
    ];

    // product-detail.component.ts
    import { Component, OnInit } from '@angular/core';
    import { ActivatedRoute } from '@angular/router';
    import { Observable } from 'rxjs';
    import { map } from 'rxjs/operators';

    @Component({ ... })
    export class ProductDetailComponent implements OnInit {
      productId$: Observable<string>;

      constructor(private route: ActivatedRoute) { }

      ngOnInit(): void {
        // Accessing parameters using a snapshot (only if component is not reused for different params)
        // const id = this.route.snapshot.paramMap.get('id');

        // Accessing parameters using an Observable (preferred, especially if component can be reused)
        this.productId$ = this.route.paramMap.pipe(
          map(params => params.get('id'))
        );
      }
    }
    // To naviagte with route params
    this.router.navigate(['/product', productId]);

    // To navigate with route params using routerLink in HTML
    // Assuming productId is available in the component's context
    // <a [routerLink]="['/product', productId]">View Product</a>
    // Or for a fixed ID:
    // <a [routerLink]="['/product', 123]">View Product 123</a>
    ```
-   **Query Parameters**: Optional parameters appended to the URL after a `?` (e.g., `/products?category=electronics&sort=price`).
    ```typescript
    // To navigate with query params
    this.router.navigate(['/products'], { queryParams: { category: 'electronics', sort: 'price' } });

    // To navigate with query params using routerLink in HTML
    // <a [routerLink]="['/products']" [queryParams]="{ category: 'electronics', sort: 'price' }">View Electronics</a>
    // To preserve current query params (useful for filtering refinement):
    // <a [routerLink]="['/products']" [queryParams]="{ newParam: 'value' }" queryParamsHandling="merge">Add Filter</a>


    // To read query params
    this.route.queryParamMap.pipe(
      map(params => params.get('category'))
    ).subscribe(category => console.log(category));
    ```
-   **Fragment**: An identifier after a `#` in the URL, often used for navigating to specific sections within a page (e.g., `/about#team`).
    ```typescript
    // To navigate with a fragment
    this.router.navigate(['/about'], { fragment: 'team' });

    // To read the fragment
    this.route.fragment.subscribe(fragment => console.log(fragment));
    ```

### Child Routes

Child routes allow you to define nested routing structures within a parent component's `RouterOutlet`.

```typescript
// user-routing.module.ts (example for a lazy-loaded 'UserModule')
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { UserSettingsComponent } from './user-settings/user-settings.component';
import { UserDashboardComponent } from './user-dashboard/user-dashboard.component';

const routes: Routes = [
  {
    path: 'users/:id',
    component: UserProfileComponent, // Parent component
    children: [ // Child routes for the UserProfileComponent's RouterOutlet
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: UserDashboardComponent },
      { path: 'settings', component: UserSettingsComponent },
      { path: 'edit', loadChildren: () => import('./user-edit/user-edit.module').then(m => m.UserEditModule) }
    ]
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class UserRoutingModule { }
```

## The "Industry" Way

In large enterprise applications, routing is critical for modularity, maintainability, and performance.

-   **Feature Module Routing**: Each major feature (e.g., authentication, products, administration) should typically have its own routing module (`<Feature>RoutingModule`). This keeps routing configurations cohesive and enables lazy loading of features.
-   **`RouterModule.forRoot()` vs. `RouterModule.forChild()`: A Deeper Dive**
    Understanding the distinction between these two methods is fundamental for building modular and performant Angular applications, especially when dealing with feature modules and lazy loading.

    -   **`RouterModule.forRoot(routes)`**
        *   **Purpose**: This static method is designed to be called **once and only once** in the root application module (typically `AppModule` or its dedicated `AppRoutingModule`).
        *   **Functionality**:
            1.  **Registers Global Services**: It registers the essential, global routing service providers (e.g., `Router`, `ActivatedRoute`, `RouterOutletMap`) into the root injector. These services are intended to be singletons throughout the application's lifecycle.
            2.  **Configures Root Routes**: It provides the initial routing configuration for the entire application.
            3.  **Sets up Router Listeners**: It sets up the listeners for browser URL changes and initiates the routing process.
        *   **Consequence of Multiple Calls**: Calling `forRoot()` more than once (e.g., in a feature module) would lead to multiple instances of the router services being registered. This can cause unpredictable behavior, such as incorrect navigation, issues with `ActivatedRoute` data, and increased memory consumption, as the router might struggle to maintain a consistent state.
        *   **Example (in `AppRoutingModule` or `AppModule`):**
            ```typescript
            import { NgModule } from '@angular/core';
            import { RouterModule, Routes } from '@angular/router';
            import { HomeComponent } from './home/home.component';

            const appRoutes: Routes = [
              { path: '', component: HomeComponent },
              { path: 'admin', loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule) },
            ];

            @NgModule({
              imports: [
                RouterModule.forRoot(appRoutes, {
                  // Optional: configure additional router features
                  preloadingStrategy: PreloadAllModules,
                  scrollPositionRestoration: 'enabled'
                })
              ],
              exports: [RouterModule]
            })
            export class AppRoutingModule { }
            ```

    -   **`RouterModule.forChild(routes)`**
        *   **Purpose**: This static method is designed to be called in **feature modules** (e.g., `AdminModule`, `ProductsModule`), especially those that are lazy-loaded.
        *   **Functionality**:
            1.  **Registers Feature Routes**: It registers additional routing configurations that are specific to that feature module. These routes are merged with the root route configuration.
            2.  **Does NOT Register Global Services**: Crucially, it does **not** register new instances of the global router services. Instead, it relies on the singletons already provided by `forRoot()` in the root injector. This prevents the "multiple router instances" problem.
        *   **Benefit**: This allows for true modularity. Each feature module can manage its own routing concerns without interfering with the global routing setup, and it enables lazy loading where the routing configuration for a module is only loaded when the module itself is loaded.
        *   **Example (in `AdminRoutingModule` within `AdminModule`):**
            ```typescript
            import { NgModule } from '@angular/core';
            import { RouterModule, Routes } from '@angular/router';
            import { AdminDashboardComponent } from './admin-dashboard/admin-dashboard.component';
            import { UserManagementComponent } from './user-management/user-management.component';

            const adminRoutes: Routes = [
              {
                path: '', // Base path is '/admin' from parent route
                component: AdminDashboardComponent,
                children: [
                  { path: 'users', component: UserManagementComponent },
                  // ... other admin sub-routes
                ]
              }
            ];

            @NgModule({
              imports: [RouterModule.forChild(adminRoutes)],
              exports: [RouterModule]
            })
            export class AdminRoutingModule { }
            ```
    In summary, `forRoot()` initializes the router and registers its global services, making it a singleton, while `forChild()` extends the routing configuration in feature modules without creating new instances of those global services, ensuring a consistent and modular routing setup.

-   **`data` property for Static Data**: You can attach arbitrary static data to a route configuration using the `data` property. This is useful for things like page titles, breadcrumbs, or permissions that don't change based on route parameters.
    ```typescript
    const routes: Routes = [
      {
        path: 'admin',
        component: AdminDashboardComponent,
        data: { title: 'Admin Dashboard', roles: ['admin'] }
      }
    ];
    // In component: this.route.data.subscribe(data => console.log(data.title));
    ```
-   **`resolve` for Dynamic Data Pre-fetching**: The `resolve` property allows you to fetch data before a route is activated. This ensures that the component only renders once all necessary data is available, preventing flashes of incomplete UI.
    ```typescript
    // user.resolver.ts
    import { Injectable } from '@angular/core';
    import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
    import { Observable } from 'rxjs';
    import { UserService } from './user.service'; // Assume a service to fetch user data

    @Injectable({ providedIn: 'root' })
    export class UserResolver implements Resolve<any> { // Replace 'any' with actual user type
      constructor(private userService: UserService) {}

      resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<any> {
        const userId = route.paramMap.get('id');
        return this.userService.getUser(userId); // Fetch user data
      }
    }

    // app-routing.module.ts
    const routes: Routes = [
      {
        path: 'user/:id',
        component: UserProfileComponent,
        resolve: { userData: UserResolver } // 'userData' will be the key in ActivatedRoute.data
      }
    ];

    // In UserProfileComponent: this.route.data.subscribe(data => this.user = data.userData);
    ```
    While `resolve` is still valid, functional resolvers (introduced in Angular 15.2+) are the modern, tree-shakeable way.
    ```typescript
    import { ResolveFn } from '@angular/router';
    import { UserService } from './user.service';
    import { inject } from '@angular/core';

    export const userResolver: ResolveFn<any> = (route, state) => {
      const userService = inject(UserService);
      const userId = route.paramMap.get('id');
      return userService.getUser(userId);
    };

    // In routes config
    const routes: Routes = [
      {
        path: 'user/:id',
        component: UserProfileComponent,
        resolve: { userData: userResolver }
      }
    ];
    ```
-   **Preloading Strategies**: Beyond basic lazy loading, preloading strategies can further optimize performance by loading feature modules in the background after the initial application load.
    -   `PreloadAllModules`: Preloads all lazy-loaded modules after the initial app bootstrap.
    -   `NoPreloading` (default): No modules are preloaded.
    -   **Custom Preloading Strategy**: You can implement your own strategy to decide which modules to preload and when (e.g., based on network conditions, user idle time, or frequently visited routes).

## Common Pitfalls

-   **Incorrect `forRoot`/`forChild` Usage**: A common mistake is using `forRoot()` in feature modules, which can lead to multiple instances of router services and unexpected behavior.
-   **`pathMatch` Mismatches**: Incorrectly using `'prefix'` when `'full'` is required (especially with `redirectTo`) can lead to unintended route matches or redirects.
-   **`ActivatedRoute` Subscription Leaks**: Not unsubscribing from `ActivatedRoute` observables (e.g., `paramMap`, `queryParamMap`) can lead to memory leaks, especially in components that are destroyed and recreated multiple times. Use `takeUntil` with a `Subject` or `async` pipe.
-   **Over-fetching in `ngOnInit`**: Fetching complex or large data sets in a component's `ngOnInit` without using a `resolve` can lead to the component rendering with incomplete data, resulting in a poor user experience.
-   **Complex, Monolithic Route Configurations**: Keeping all routes in a single `AppRoutingModule` for a large application makes it hard to manage and debug. Break down routing into feature-specific routing modules.
-   **Performance with Eager Loading**: Loading all modules eagerly in a large application dramatically increases initial bundle size and load time. Always consider lazy loading feature modules.
-   **Missing Wildcard Route**: Forgetting the `path: '**'` route can lead to white screens or confusing errors when users navigate to invalid URLs. It should always be the last route.
-   **Relative Path Issues**: When using `router.navigate()`, be mindful of relative paths. Using `relativeTo: this.route` can be crucial for correct navigation within child routes.

By understanding these concepts and avoiding common pitfalls, you can build robust, performant, and maintainable routing for your Angular applications.