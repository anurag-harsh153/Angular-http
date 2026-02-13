# Angular Universal (SSR) Machine Coding Challenges

These challenges are designed to test your understanding and application of Server-Side Rendering (SSR) with Angular Universal, focusing on various aspects from basic setup to advanced state management and SEO optimization.

---

### Level 1 (Junior): Basic Product List with Universal Setup

**Scenario**: You have an existing Angular application that displays a list of products fetched from an API. The client now requires this application to be server-side rendered to improve initial load time and SEO.

**Task**:
1.  Start with a basic Angular application (you can create a new one using `ng new your-app`).
2.  Create a `ProductService` that fetches an array of product objects (e.g., `{ id: 1, name: 'Laptop', price: 1200 }`) from a mock API endpoint (e.g., using `HttpClient` to a local JSON file or a simple mock server).
3.  Display this list of products in a `ProductListComponent`.
4.  Integrate Angular Universal into this application using `ng add @angular/universal`.
5.  Verify that the application successfully renders on the server by running `npm run dev:ssr` and inspecting the page source in the browser (you should see the product list in the initial HTML).

**Expected Output**:
*   A functional Angular application.
*   The product list should be visible in the page's HTML source when served via `dev:ssr`, indicating successful server-side rendering.

---

### Level 2 (Mid): Dynamic Page Titles & Meta Descriptions with State Transfer

**Scenario**: You have an e-commerce product detail page that displays information for a specific product based on its ID in the URL. To enhance SEO, the page title and meta description need to be dynamically updated based on the product's data, and data fetched on the server should not be re-fetched on the client.

**Task**:
1.  Building on the Level 1 project (or a similar setup):
    *   Create a `ProductDetailComponent` that fetches a single product's details (`{ id: 1, name: 'Laptop', description: 'Powerful computing machine', price: 1200 }`) based on a route parameter (`/product/:id`).
    *   Implement `ProductService` to fetch product details using `HttpClient`.
2.  Implement dynamic SEO:
    *   Use Angular's `Title` and `Meta` services to set the page title (e.g., "Laptop - Product Details") and meta description (e.g., "Powerful computing machine...") based on the fetched product data.
3.  Implement State Transfer:
    *   Ensure that the product data fetched during server-side rendering is transferred to the client using `TransferState` so that the client-side Angular application does not re-fetch the same product data.
4.  Verify functionality by:
    *   Navigating to a product detail page (`/product/1`) with `npm run dev:ssr`.
    *   Inspecting the page source to confirm the dynamic title and meta description are present.
    *   Using browser developer tools to verify that the `HttpClient` request for product details is *not* made again on the client-side after hydration.

**Expected Output**:
*   Dynamic page title and meta description visible in the server-rendered HTML.
*   Product data fetched only once (on the server), and then reused by the client-side app.

---

### Level 3 (Senior/Architect): Handling Browser-Specific Code and Optimizing for Critical CSS

**Scenario**: Your Angular Universal application uses a third-party analytics library that relies heavily on the `window` object and a custom chat widget that manipulates the DOM directly. Additionally, you want to optimize the application's First Contentful Paint by inlining critical CSS.

**Task**:
1.  Building on the Level 2 project:
    *   Simulate a third-party analytics script (e.g., a simple service `AnalyticsService`) that tries to access `window.ga('send', ...)` in its constructor or a method.
    *   Simulate a chat widget component (`ChatWidgetComponent`) that attempts to access `document.getElementById(...)` or directly appends elements to `document.body` in `ngOnInit`.
2.  **Platform-Specific Code Handling**:
    *   Modify `AnalyticsService` and `ChatWidgetComponent` to gracefully handle execution on the server. Use `isPlatformBrowser` (or similar techniques) to ensure browser-specific code only runs on the client, preventing SSR errors. The server should ideally not execute these scripts.
3.  **Critical CSS**:
    *   (Conceptual/Research Task): Describe how you would integrate a critical CSS extraction tool (e.g., the `critical` npm package) into your Angular Universal build process to inline essential styles for the initial view. Provide a conceptual outline or a simplified `angular.json` snippet showing where you'd hook it in. (Actual implementation of a custom build step is not required, but a clear plan is).
4.  Verify functionality by:
    *   Running `npm run dev:ssr` and ensuring no server-side errors occur due to browser API access.
    *   Observing that the analytics script and chat widget functionality only activate after the client-side application has bootstrapped.

**Expected Output**:
*   Application renders successfully via Universal without server-side crashes from browser-specific code.
*   Analytics and chat features function correctly in the browser.
*   A clear explanation or conceptual build configuration for integrating critical CSS.