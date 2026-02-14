# Angular Route Guards Challenges

These challenges are designed to test your understanding and practical application of Angular Route Guards.

## Level 1 (Basic): `CanActivate` for Authentication

**Goal**: Implement a basic authentication guard to protect a route.

### Challenge Description

1.  **Setup**: Start with a new Angular project.
2.  **Components**:
    *   `LoginComponent`: A simple component with a login button.
    *   `DashboardComponent`: Displays a message like "Welcome to the Dashboard!".
    *   `UnauthorizedComponent`: Displays "Access Denied!".
3.  **Service**: Create an `AuthService` with:
    *   `isLoggedIn()`: Returns a boolean indicating the login status (initially `false`).
    *   `login()`: Sets `isLoggedIn` to `true`.
    *   `logout()`: Sets `isLoggedIn` to `false`.
    *   (Optional) Use a `BehaviorSubject` to make login status observable.
4.  **`CanActivate` Guard**: Create a functional `CanActivate` guard named `authGuard`:
    *   It should inject `AuthService` and `Router`.
    *   If `AuthService.isLoggedIn()` returns `true`, the guard should return `true` (allow activation).
    *   If `AuthService.isLoggedIn()` returns `false`, the guard should navigate to `/unauthorized` using the `Router` and return `false` (prevent activation).
5.  **Routing Configuration**:
    *   Define routes:
        *   `/login` for `LoginComponent`.
        *   `/dashboard` for `DashboardComponent`, protected by `authGuard`.
        *   `/unauthorized` for `UnauthorizedComponent`.
        *   A default route that redirects to `/login`.
6.  **Implementation**:
    *   In `LoginComponent`, add a button to call `AuthService.login()` and then navigate to `/dashboard`.
    *   In `DashboardComponent`, add a logout button that calls `AuthService.logout()` and navigates to `/login`.
7.  **Verification**:
    *   Verify that you cannot access `/dashboard` directly without logging in.
    *   Verify that you can access `/dashboard` after logging in.
    *   Verify that logging out redirects you to `/login` and subsequent attempts to access `/dashboard` are blocked.

## Level 2 (Mid): `CanDeactivate` for Unsaved Changes

**Goal**: Implement a `CanDeactivate` guard to warn users about unsaved form changes before navigating away.

### Challenge Description

1.  **Setup**: Continue with the project from Level 1, or start new.
2.  **Components**:
    *   `FormComponent`: A component with a simple HTML form (e.g., input fields for name, email).
    *   This component should have a boolean property `hasUnsavedChanges` (initially `false`).
    *   When the user types in any input field, set `hasUnsavedChanges` to `true`.
    *   Add a "Save" button that, when clicked, sets `hasUnsavedChanges` to `false` and displays a confirmation message.
3.  **Interface**: Define an interface `CanComponentDeactivate` with a single method `canDeactivate(): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree;`. Make `FormComponent` implement this interface.
4.  **`CanDeactivate` Guard**: Create a functional `CanDeactivate` guard named `confirmDeactivateGuard`:
    *   It should accept a component of type `CanComponentDeactivate` as its first argument.
    *   Inside the guard, call the `canDeactivate()` method of the component.
    *   If the component's `canDeactivate()` returns `false` (meaning there are unsaved changes and the user doesn't want to leave), the guard should return `false`.
    *   Otherwise, return `true`.
5.  **Routing Configuration**:
    *   Define a route for `/form` that displays `FormComponent`.
    *   Apply `confirmDeactivateGuard` to the `/form` route using `canDeactivate: [confirmDeactivateGuard]`.
    *   Add another navigation link (e.g., to `/home`) in `AppComponent` to test navigation away from the form.
6.  **Verification**:
    *   Navigate to `/form`. Make some changes in the form but don't save.
    *   Try to navigate to `/home` (or any other route).
    *   Verify that the browser's native confirmation dialog (or a custom one if implemented in `canDeactivate`) appears.
    *   Verify that you stay on the form if you cancel the navigation.
    *   Verify that you can navigate away if you confirm the navigation or if there are no unsaved changes.

## Level 3 (Senior): Role-Based Access with `CanMatch` & `Resolve`

**Goal**: Implement role-based access control using `CanMatch` to prevent module loading, and pre-fetch user data using a `Resolve` guard.

### Challenge Description

1.  **Setup**: Continue with the project from Level 2, or start new.
2.  **Services**:
    *   **`AuthService`**: Extend it to include:
        *   `hasRole(role: string)`: Returns `boolean` based on a mock user role (e.g., current user is 'ADMIN' or 'USER').
        *   `getLoggedInUser()`: Returns an `Observable<User>` (mock `User` interface with `id`, `name`, `role`). Simulate a network delay (e.g., 500ms) for this observable.
    *   **`UserService`**: A simple service to simulate fetching user details by ID.
3.  **Modules & Components**:
    *   **`AdminModule` (Lazy-loaded)**: Create this module with an `AdminDashboardComponent` inside it. This module should only be loaded if the user has the 'ADMIN' role.
    *   **`UserProfileComponent`**: Displays user details.
4.  **`CanMatch` Guard**: Create a functional `CanMatch` guard named `adminMatchGuard`:
    *   It should inject `AuthService` and `Router`.
    *   It should check if `AuthService.hasRole('ADMIN')` returns `true`.
    *   If not, navigate to `/unauthorized` (or a more specific 'Forbidden' page) and return `false`. Otherwise, return `true`.
5.  **`Resolve` Guard**: Create a functional `ResolveFn` named `userResolver`:
    *   It should inject `UserService`.
    *   It should extract the `id` from `route.paramMap`.
    *   It should return `userService.getUser(id)` (which should return an Observable of user data).
6.  **Routing Configuration**:
    *   In `AppRoutingModule`:
        *   Configure a route `/admin` to lazy load `AdminModule`, applying `adminMatchGuard` using `canMatch: [adminMatchGuard]`.
        *   Configure a route `/user/:id` to display `UserProfileComponent`, using `userResolver` to pre-fetch user data (`resolve: { userData: userResolver }`).
7.  **Implementation**:
    *   In `AppComponent` or `LoginComponent`, provide a way to switch the logged-in user's role (e.g., 'ADMIN' vs 'USER').
    *   In `AdminDashboardComponent`, display a message confirming access.
    *   In `UserProfileComponent`, access the `userData` from `ActivatedRoute.data` and display the user's details. Show a loading indicator if the `Resolve` takes time.
8.  **Verification**:
    *   Log in as a 'USER'. Try to navigate to `/admin`. Verify that `AdminModule` is *not* loaded and you are redirected to `/unauthorized`.
    *   Log in as an 'ADMIN'. Verify that you can navigate to `/admin` and `AdminModule` loads.
    *   Navigate to `/user/123` (mock ID). Verify that `UserProfileComponent` renders only after the simulated delay from `userResolver`, displaying the correct user data.
    *   Ensure that if `userResolver` fails (e.g., ID not found), appropriate error handling is in place (e.g., redirect to a 404 page, display error message).

Good luck!