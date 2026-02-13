# HttpClient Machine Coding Challenges

These challenges are designed to test your understanding and application of Angular's `HttpClient`, focusing on various aspects from basic requests to advanced error handling and RxJS manipulations.

---

### Level 1 (Junior): Basic Product Listing with Error Display

**Scenario**: You need to display a list of products fetched from a REST API. The API might occasionally return an error.

**Task**:
1.  Create an Angular service (`ProductService`) that uses `HttpClient` to fetch a list of products from a hypothetical API endpoint (e.g., `https://api.example.com/products`). Define a `Product` interface for type safety.
2.  Implement a `ProductListComponent` that uses the `ProductService` to display the fetched products.
3.  Add basic error handling: If the API call fails, display a user-friendly error message on the screen instead of the product list. Use `catchError` from RxJS.
4.  (Bonus) Implement a loading indicator that shows "Loading products..." while the data is being fetched and disappears once data is received or an error occurs.

**Expected Output**:
*   A list of product names and prices, or
*   An error message like "Failed to load products. Please try again later."
*   (Bonus) A "Loading products..." message before content appears.

---

### Level 2 (Mid): User Authentication with Interceptors and Centralized Error Handling

**Scenario**: In a corporate application, all API requests require an authentication token. Additionally, all API errors should be handled consistently across the application, displaying a notification to the user.

**Task**:
1.  Create an `AuthService` with a method (`getAuthToken()`) that returns a dummy authentication token (e.g., `'Bearer my-super-secret-token'`).
2.  Implement an `AuthInterceptor` that automatically adds the authentication token from `AuthService` to the `Authorization` header of every outgoing HTTP request.
3.  Create an `ErrorHandlingInterceptor` that catches `HttpErrorResponse` errors globally. For any server-side error (status >= 500), it should display a generic "Server error occurred. Please try again." notification (e.g., using a simple `console.warn` or a mock notification service). For a `401 Unauthorized` error, it should log a specific message "User session expired. Please log in."
4.  Create a `UserService` that fetches user data from a protected endpoint (e.g., `https://api.example.com/user-profile`).
5.  Create a component (`UserProfileComponent`) that uses `UserService` to display user data. Ensure it works with the interceptors in place.

**Expected Output**:
*   User profile data displayed, or
*   A notification for generic server errors, or
*   A console message "User session expired. Please log in." for 401 errors.

---

### Level 3 (Senior/Architect): Debounced Search with Request Cancellation and Multiple API Calls

**Scenario**: You are building a search functionality where users type into an input field, and the application fetches search results from an API. To optimize performance and user experience, you need to:
1.  Only send a request after the user pauses typing.
2.  Cancel any pending old search requests if a new search term is entered.
3.  Combine results from two different search APIs (e.g., `products` and `categories`).

**Task**:
1.  Create a `SearchService` with a method `search(term: string)` that takes a search term.
2.  In `SearchService`, implement the logic using RxJS operators:
    *   `debounceTime(300)`: Wait for 300ms of inactivity before considering the search term.
    *   `distinctUntilChanged()`: Only proceed if the search term is different from the previous one.
    *   `switchMap`: Use `switchMap` to make the HTTP calls. This operator will automatically cancel previous pending requests if a new term comes in.
    *   Inside `switchMap`, make two concurrent HTTP calls using `forkJoin` to `https://api.example.com/search/products?q={term}` and `https://api.example.com/search/categories?q={term}`.
    *   Combine the results from both API calls into a single array of `SearchResult` objects (define this interface).
3.  Create a `SearchComponent` with an input field. As the user types, pass the input value to `SearchService.search()`.
4.  Display the combined search results in the `SearchComponent`.
5.  (Bonus) Add a visual indicator (e.g., a small loading spinner next to the search box) that appears when a search is in progress and disappears when results are received.

**Expected Output**:
*   As the user types, search results update dynamically after a short delay.
*   Only one active search request at any given time (older requests are cancelled).
*   Results from both product and category searches are displayed.
*   (Bonus) A loading spinner indicates active search.