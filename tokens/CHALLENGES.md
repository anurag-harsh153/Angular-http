# Dependency Injection Tokens Machine Coding Challenges

These challenges are designed to test your understanding and practical application of Angular's Dependency Injection Tokens, from basic configuration to advanced patterns using multi-providers and dynamic injection.

---

### Level 1 (Junior): Basic Configuration Injection

**Scenario**: Your Angular application needs to consume various configuration settings (e.g., API base URL, feature flags) that differ between development and production environments. You want to inject these settings cleanly into your services and components.

**Task**:
1.  Define an interface `AppConfig` with at least two properties: `apiUrl: string` and `analyticsEnabled: boolean`.
2.  Create an `InjectionToken` named `APP_CONFIG` for this `AppConfig` interface.
3.  In your `AppModule` (or `app.config.ts` if using standalone components), provide a default `AppConfig` object using `APP_CONFIG` token.
4.  Create a `SettingsService` that injects the `APP_CONFIG` token.
5.  Create an `AppComponent` that injects `SettingsService` and displays the `apiUrl` and `analyticsEnabled` values from the injected configuration.
6.  Verify that the component correctly displays the configuration values.

**Expected Output**:
*   The `AppComponent` template should render the `apiUrl` and `analyticsEnabled` values retrieved from the `APP_CONFIG` token.

---

### Level 2 (Mid): Multi-providers for Plugin Architecture

**Scenario**: You are building a data visualization application that supports multiple chart types (e.g., bar, line, pie). You want to design a pluggable architecture where new chart types can be easily added without modifying core components.

**Task**:
1.  Define an interface `ChartRenderer` with a single method: `render(data: any[], elementId: string): void`.
2.  Create an `InjectionToken` named `CHART_RENDERERS` for an array of `ChartRenderer` (`ChartRenderer[]`). This token should be configured as a `multi: true` provider.
3.  Create two concrete implementations of `ChartRenderer`:
    *   `BarChartRenderer`: Logs "Rendering Bar Chart to elementId with data: ..."
    *   `LineChartRenderer`: Logs "Rendering Line Chart to elementId with data: ..."
4.  In your `AppModule`, provide both `BarChartRenderer` and `LineChartRenderer` using the `CHART_RENDERERS` token with `multi: true`.
5.  Create a `ChartService` that injects the `CHART_RENDERERS` token (which will now be an array of `ChartRenderer` instances). It should have a method `renderAllCharts(data: any[], elementId: string)` that iterates through all injected renderers and calls their `render` method.
6.  In your `AppComponent`, call `ChartService.renderAllCharts` with some mock data and an element ID (e.g., 'chart-container').
7.  Verify that the console logs show both bar chart and line chart rendering messages.

**Expected Output**:
*   Console output showing both "Rendering Bar Chart..." and "Rendering Line Chart..." messages, demonstrating that multiple providers for the same token were successfully injected and used.

---

### Level 3 (Senior/Architect): Dynamic Logger Configuration with UseFactory

**Scenario**: In a large enterprise application, logging needs to be highly configurable. You want to dynamically decide which logging appenders (e.g., console, remote server, analytics) are active based on environment variables or runtime conditions, and also provide a global logger instance that uses these appenders.

**Task**:
1.  Define an interface `LogAppender` with a `log(message: string, level: 'info' | 'warn' | 'error'): void` method.
2.  Create two simple `LogAppender` implementations:
    *   `ConsoleAppender`: Logs to `console.log`, `console.warn`, `console.error`.
    *   `RemoteAppender`: Logs "Sending to remote: [LEVEL] message" (simulating a remote call).
3.  Create an `InjectionToken` named `ACTIVE_LOG_APPENDERS` for an array of `LogAppender` (`LogAppender[]`), also configured with `multi: true`.
4.  Create an `InjectionToken` named `GLOBAL_LOGGER_CONFIG` for an interface `{ enableConsole: boolean; enableRemote: boolean; }` to hold environment-specific flags. Provide a default value for this token in `AppModule`.
5.  Create a **`useFactory` provider** for `ACTIVE_LOG_APPENDERS`. This factory function should:
    *   Inject `GLOBAL_LOGGER_CONFIG`.
    *   Conditionally create and return instances of `ConsoleAppender` and `RemoteAppender` based on the `GLOBAL_LOGGER_CONFIG` flags.
    *   Ensure the factory correctly handles the `multi: true` aspect (it should return an array of appenders).
    *   *Hint*: The factory itself returns an array of providers, not the instances. For `multi: true`, your `useFactory` will return an array of the *instances* of `LogAppender`.
6.  Create a `LoggerService` that injects `ACTIVE_LOG_APPENDERS` and has `info()`, `warn()`, `error()` methods that delegate to all active appenders.
7.  In `AppComponent`, inject `LoggerService` and call its `info()`, `warn()`, `error()` methods.
8.  Modify the `GLOBAL_LOGGER_CONFIG` to enable/disable appenders and observe the console output.

**Expected Output**:
*   Depending on the `GLOBAL_LOGGER_CONFIG`, only the enabled appenders (console or remote, or both) should output messages when `LoggerService` methods are called. This demonstrates dynamic configuration of multi-providers using `useFactory`.

---
**Note on `providedIn` in `useFactory`**: When using `useFactory` for an `InjectionToken` with `multi: true`, you typically provide the factory at the module level in `AppModule` (or a feature module) rather than relying on `providedIn: 'root'` on the individual appenders, as the factory dynamically constructs the array of appenders. Individual appenders can still be `providedIn: 'root'` if they are also meant to be injectable as single instances elsewhere. The key is how the `useFactory` composes the `ACTIVE_LOG_APPENDERS` array.
