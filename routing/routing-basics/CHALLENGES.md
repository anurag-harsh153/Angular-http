# Angular Routing Challenges

These challenges are designed to test your understanding and practical application of Angular Routing.

## Level 1 (Basic): Simple Navigation & Wildcard Route

**Goal**: Create a basic Angular application with a few pages and implement navigation between them.

### Challenge Description

1.  **Setup**: Start with a new Angular project.
2.  **Components**: Create three simple components:
    *   `HomeComponent`: Displays a welcome message.
    *   `AboutComponent`: Displays information about the application.
    *   `NotFoundComponent`: Displays a "404 Page Not Found" message.
3.  **Routing Configuration**:
    *   Configure the `AppRoutingModule` to have routes for `/home` (mapped to `HomeComponent`) and `/about` (mapped to `AboutComponent`).
    *   Set the default route (`/`) to redirect to `/home`.
    *   Implement a wildcard route (`**`) to catch any undefined routes and display the `NotFoundComponent`.
4.  **Navigation**:
    *   In the `AppComponent` template, create navigation links using `routerLink` to go to `/home` and `/about`.
    *   Verify that navigating to `/`, `/home`, `/about` works correctly.
    *   Verify that navigating to any other arbitrary URL (e.g., `/non-existent`) displays the `NotFoundComponent`.

## Level 2 (Mid): Dynamic Routing, Lazy Loading & Relative Navigation

**Goal**: Implement dynamic product detail routing, lazy load a feature module, and practice relative navigation.

### Challenge Description

1.  **Setup**: Continue with the project from Level 1 or start a new one.
2.  **Components & Modules**:
    *   Create a `ProductsModule` (lazy-loaded).
    *   Inside `ProductsModule`, create `ProductListComponent` (to display a list of products) and `ProductDetailComponent` (to display details of a single product).
    *   Create a simple `ProductService` that returns an array of mock product objects (e.g., `[{ id: 1, name: 'Product A', price: 100 }, { id: 2, name: 'Product B', price: 200 }]`).
3.  **Routing Configuration**:
    *   Configure the `AppRoutingModule` to lazy load the `ProductsModule` when the path `/products` is accessed.
    *   Inside `ProductsModule`'s routing:
        *   Define a route for `/products` that displays `ProductListComponent`.
        *   Define a child route for `/products/:id` that displays `ProductDetailComponent`, where `:id` is a route parameter for the product ID.
4.  **Implementation**:
    *   **`ProductListComponent`**:
        *   Fetch the list of products from `ProductService`.
        *   Display the products as a list. Each product item should have a `routerLink` to its `ProductDetailComponent` (e.g., `/products/1`).
    *   **`ProductDetailComponent`**:
        *   Use `ActivatedRoute` to retrieve the `id` route parameter.
        *   Fetch the corresponding product details from `ProductService` based on the `id`.
        *   Display the product's name and price.
        *   Add a "Back to Products" button that navigates back to `ProductListComponent` using `Router.navigate` with `relativeTo: this.route` to ensure correct relative navigation.

## Level 3 (Senior): Role-Based Access Control with `CanMatch` & Custom Preloading

**Goal**: Implement a role-based access control system using `CanMatch` for module loading and develop a custom preloading strategy.

### Challenge Description

1.  **Setup**: Continue with the project from Level 2 or start a new one.
2.  **Authentication & Authorization Service**:
    *   Create an `AuthService` that provides:
        *   `isLoggedIn()`: Returns `boolean` (mock a logged-in state).
        *   `hasRole(role: string)`: Returns `boolean` (mock different user roles, e.g., 'ADMIN', 'USER').
        *   `login()`: Simulates login and sets a user role.
        *   `logout()`: Simulates logout.
3.  **Admin Module**:
    *   Create an `AdminModule` (lazy-loaded) with an `AdminDashboardComponent`. This module should only be accessible by users with the 'ADMIN' role.
4.  **Custom `CanMatch` Guard**:
    *   Create a functional `CanMatch` guard (`adminMatchGuard`) that checks if the user is logged in AND has the 'ADMIN' role using the `AuthService`.
    *   If the user does not have the 'ADMIN' role, the guard should prevent the module from being loaded and potentially redirect to a 'Forbidden' page or the login page.
5.  **Custom Preloading Strategy**:
    *   Implement a custom preloading strategy (`SelectivePreloadingStrategy`).
    *   This strategy should only preload modules that have a `preload: true` property in their route `data`.
    *   The strategy should also introduce a small delay (e.g., 2-3 seconds) before preloading, simulating a more intelligent preloading mechanism.
6.  **Routing Configuration**:
    *   In `AppRoutingModule`:
        *   Add a lazy-loaded route for the `AdminModule` using `canMatch: [adminMatchGuard]`.
        *   Add the `data: { preload: true }` property to at least one other lazy-loaded module (e.g., `ProductsModule`) to test your custom preloading strategy.
        *   Configure `RouterModule.forRoot` to use `SelectivePreloadingStrategy`.
7.  **Implementation**:
    *   Add login/logout functionality in `AppComponent` to switch user roles.
    *   Verify that the `AdminModule` is only accessible when logged in as an 'ADMIN'.
    *   Verify that `ProductsModule` (or any other module with `preload: true`) is preloaded after a short delay, while other lazy-loaded modules are not.
    *   (Optional) Display a simple message in `AdminDashboardComponent` to confirm its loaded.

Good luck!