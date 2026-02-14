# Dependency Injection Tokens Interview Preparation Module

Here are high-impact interview questions regarding Angular's Dependency Injection Tokens, designed to test a candidate's theoretical knowledge, practical understanding, and problem-solving skills.

---

### The "Basics"

1.  **Question**: What is an `InjectionToken` in Angular, and why do we use it instead of just a class for dependency injection?
    **STAR Answer**: An `InjectionToken` is a special type of token from `@angular/core` that serves as a lookup key for the dependency injector. While Angular typically uses class types (e.g., `MyService`) as tokens by default, `InjectionToken` is crucial for scenarios where a class type isn't suitable. This includes injecting primitive values (strings, numbers), plain JavaScript objects (like configuration objects), interfaces (which don't exist at runtime), abstract classes, or when integrating third-party JavaScript libraries that don't export Angular-injectable classes. It provides a unique, type-safe identifier for dependencies that the DI system can resolve.

2.  **Question**: Explain the significance of the `multi: true` option when providing a token. Provide a use case.
    **STAR Answer**: The `multi: true` option in a provider definition tells Angular that the `InjectionToken` can be associated with multiple values, rather than just a single one. When `multi: true` is used, instead of receiving a single instance of a dependency, the injecting component or service receives an array of all registered values for that token. This is particularly useful for building extensible architectures or "plugin" systems.
    **Use Case**: A common use case is for logging appenders or notification handlers. You could define an `InjectionToken` for `LOG_APPENDERS` and then provide `ConsoleAppender`, `FileAppender`, and `RemoteAppender` each with `multi: true`. A `LoggerService` would then inject `LOG_APPENDERS` and iterate through the array to send log messages to all registered appenders.

---

### The "Scenario"

3.  **Question**: "Your client has an Angular application with various feature modules, and each module needs slightly different configuration values (e.g., a different API endpoint, or specific feature toggles). How would you use `InjectionToken` to manage these module-specific configurations without hardcoding or creating global singletons?"
    **STAR Answer**: I would define an `InjectionToken` (e.g., `FEATURE_MODULE_CONFIG`) for a specific configuration interface. Then, within each feature module's `@NgModule`, I would provide a unique configuration object using `useValue` for that `FEATURE_MODULE_CONFIG` token. This ensures that when services or components *within that specific feature module* inject `FEATURE_MODULE_CONFIG`, they receive the configuration relevant to their module's scope, thanks to Angular's hierarchical dependency injection. This approach avoids global singletons and allows each module to manage its configuration independently, improving modularity and reusability.

    ```typescript
    // feature-module.config.ts
    export interface FeatureModuleConfig {
      moduleApiUrl: string;
      enableSpecificFeature: boolean;
    }
    export const FEATURE_MODULE_CONFIG = new InjectionToken<FeatureModuleConfig>('FeatureModuleConfig');

    // some-feature.module.ts
    @NgModule({
      // ...
      providers: [
        {
          provide: FEATURE_MODULE_CONFIG,
          useValue: { moduleApiUrl: '/api/feature-one', enableSpecificFeature: true }
        }
      ]
    })
    export class SomeFeatureModule {}

    // inside a service in SomeFeatureModule
    constructor(@Inject(FEATURE_MODULE_CONFIG) private config: FeatureModuleConfig) {}
    ```

---

### The "Comparison"

4.  **Question**: Compare using an `InjectionToken` with `useValue` to a simple class constant (`const`) for providing configuration data. When would you choose one over the other?
    **STAR Answer**:
    *   **Class Constant (`const`)**: A simple `const` variable (e.g., `export const API_URL = '...'`) is straightforward and provides type safety. It's suitable for truly static, compile-time known values that never change across environments or modules, and don't require any form of dependency injection context.
    *   **`InjectionToken` with `useValue`**: This is preferred when the configuration value:
        1.  Needs to be **dynamically switchable** (e.g., different values for dev/prod builds).
        2.  Needs to be part of Angular's **DI hierarchy** (e.g., module-specific configurations).
        3.  Might be **overridden** by other providers in child injectors.
        4.  Is a non-primitive object or interface, benefiting from the `InjectionToken`'s explicit type.
    Choosing an `InjectionToken` makes the configuration injectable, testable (mockable in tests), and allows for powerful features like `multi: true` and `useFactory` for more complex provisioning. If you need DI features, `InjectionToken` is the way to go; otherwise, a simple `const` is fine.

---

### The "Senior/Architect"

5.  **Question**: Discuss how `InjectionToken`s and `useFactory` can be leveraged to implement a sophisticated runtime configuration or feature flagging system in a large Angular application.
    **STAR Answer**: For a sophisticated runtime configuration and feature flagging system, `InjectionToken`s combined with `useFactory` are incredibly powerful.
    *   **Define Configuration Token**: First, define an `InjectionToken` (e.g., `APP_SETTINGS`) for an interface representing all global application settings and feature flags.
    *   **Dynamic `useFactory`**: The core would be a `useFactory` provider for `APP_SETTINGS`. This factory function would:
        1.  **Inject Dependencies**: It could inject other services or tokens, such as `HttpClient` to fetch configuration from a backend API, or `DOCUMENT` to read meta tags or `localStorage` for user-specific overrides.
        2.  **Environment-Specific Logic**: Based on environment variables (`process.env.NODE_ENV`), it could load different default configurations.
        3.  **Merge and Override**: It would merge default settings with fetched settings (from API) and potentially user overrides.
        4.  **Return Settings Object**: Finally, it returns the fully resolved configuration object.
    *   **Asynchronous Initialization**: If the configuration fetch is asynchronous, you'd typically need an `APP_INITIALIZER` to ensure the factory resolves before the application bootstraps, or manage the loading state with a `BehaviorSubject` in a dedicated service.
    *   **Benefits**: This architecture allows:
        *   **Centralized Configuration Logic**: All logic for loading, merging, and resolving configuration lives in one place.
        *   **Runtime Flexibility**: Configurations and feature flags can be changed without rebuilding the application (if fetched from an API).
        *   **Testability**: The factory function can be easily tested by mocking its dependencies.
        *   **Scalability**: New settings and flags can be added to the interface without impacting existing injection points.

6.  **Question**: In the context of Angular Universal (SSR), what special considerations or potential pitfalls arise when using `InjectionToken`s, especially those that might involve browser-specific objects or asynchronous factories? How would you handle them?
    **STAR Answer**: When using `InjectionToken`s with Angular Universal, developers must be extremely careful about the execution context (server vs. browser).
    *   **Browser-Specific APIs**: If a `useValue` or `useFactory` for an `InjectionToken` directly or indirectly accesses browser-specific globals like `window`, `document`, or `localStorage`, it will cause errors on the Node.js server during SSR.
        *   **Handling**: Use `isPlatformBrowser` from `@angular/common` within the `useFactory` function to conditionally provide different values or instances based on the platform. Alternatively, provide server-side mock implementations for these browser APIs using a separate `InjectionToken` and provider for the server build.
    *   **Asynchronous Factories**: If a `useFactory` fetches data asynchronously (e.g., via `HttpClient`), Universal needs to wait for these asynchronous operations to complete before rendering the HTML.
        *   **Handling**: Ensure these asynchronous tasks are properly "Zone-aware" and complete within the application's Zone.js context. `APP_INITIALIZER`s are often used for critical asynchronous setup that must complete before hydration. If not handled correctly, the server might send incomplete HTML or hydration could fail, leading to flickers or non-interactive pages.
    *   **State Transfer**: For data loaded via `useFactory` on the server that should not be re-fetched on the client, `TransferState` should be used. The factory can store the data in `TransferState` on the server, and the client-side factory can retrieve it, preventing duplicate API calls.

7.  **Question**: You are designing a core library module that provides shared services and components to multiple Angular applications. How can `InjectionToken`s be used to make this library highly configurable and extensible without forcing applications to modify the library's source code?
    **STAR Answer**: `InjectionToken`s are essential for creating flexible Angular libraries.
    *   **Configuration Token**: Provide an `InjectionToken` for library-wide configuration (e.g., `MY_LIB_CONFIG`). The library's `forRoot` static method in its module would typically accept this configuration and provide it. Applications would then call `MyLibModule.forRoot({ /* config here */ })` to configure the library.
    *   **Extension Points (Multi-providers)**: For pluggable features, define `InjectionToken`s with `multi: true`. For example, `MY_LIB_VALIDATORS` or `MY_LIB_PLUGINS`. The library's core services would then inject `MY_LIB_VALIDATORS` as an array and iterate through them. Consuming applications can then provide their custom validators or plugins using the same `InjectionToken` with `multi: true`, extending the library's functionality without altering its code.
    *   **Override Tokens**: If the library uses its own internal services that a consuming application might want to replace (e.g., a default `LoggingService`), the library can expose an `InjectionToken` for that service. The application can then provide its own `useClass` or `useFactory` for that token, effectively overriding the library's default implementation. This makes the library adaptable to diverse application needs.

---
This module covers essential aspects of Dependency Injection Tokens for various interview levels. A candidate who can answer these questions with comprehensive, code-backed explanations demonstrates a strong understanding of Angular's DI system.
