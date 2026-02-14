# Dependency Injection Tokens in Angular

## Core Concept: Understanding Dependency Injection Tokens

Dependency Injection (DI) is a core principle in Angular, allowing you to design your application with loosely coupled components. At the heart of Angular's DI system are **tokens**. A DI token is essentially a lookup key that Angular uses to find a dependency provider in the injector tree. When you ask for a dependency (e.g., in a constructor), Angular looks up the token and returns the corresponding instance provided by a provider.

### Why do we need DI Tokens?

While TypeScript classes are often used as tokens by default (e.g., `constructor(private myService: MyService)`), there are scenarios where a simple class name isn't sufficient or appropriate:

1.  **Injecting Primitive Values, Objects, or Configuration:** You cannot use a primitive (like a string or number) or a plain JavaScript object directly as a type to inject. Tokens provide a way to associate a provider with such non-class values.
2.  **Injecting Third-Party JavaScript Libraries:** If a library doesn't export an Angular-injectable class, you need a token to provide and inject it.
3.  **Injecting Interfaces or Abstract Classes:** TypeScript interfaces and abstract classes do not exist at runtime, so they cannot be used as injection tokens directly. `InjectionToken` provides a way to create a runtime token for such compile-time constructs.
4.  **Avoiding Circular Dependencies:** In some complex scenarios, custom tokens can help break circular dependencies.
5.  **Multi-providers:** When you want to provide multiple values for a single token, which is common for extensibility points.

### Types of Tokens

Angular primarily uses three types of tokens for dependency lookup:

1.  **Class (Type) as Token (Default)**: The most common scenario. When you write `constructor(private service: MyService)`, `MyService` class itself acts as the token.

    ```typescript
    // my-service.ts
    import { Injectable } from '@angular/core';

    @Injectable({
      providedIn: 'root'
    })
    export class MyService {
      constructor() { console.log('MyService instantiated'); }
      doSomething() { return 'Something from MyService'; }
    }

    // app.component.ts
    import { Component } from '@angular/core';
    import { MyService } from './my-service';

    @Component({
      selector: 'app-root',
      template: `
        <button (click)="callService()">Call Service</button>
        <p>{{ result }}</p>
      `
    })
    export class AppComponent {
      result: string = '';
      constructor(private myService: MyService) {} // MyService class is the token

      callService() {
        this.result = this.myService.doSomething();
      }
    }
    ```

2.  **`InjectionToken`**: This is a powerful feature from `@angular/core` used to create explicit tokens for non-class dependencies, interfaces, or any scenario where a unique key is needed.

    ```typescript
    // app.config.ts
    import { InjectionToken } from '@angular/core';

    export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

    export interface AppConfig {
      apiUrl: string;
      featureEnabled: boolean;
    }

    // app.module.ts (or environment.ts if global)
    import { NgModule } from '@angular/core';
    import { BrowserModule } from '@angular/platform-browser';
    import { AppComponent } from './app.component';
    import { APP_CONFIG } from './app.config';

    const productionConfig: AppConfig = {
      apiUrl: 'https://api.prod.example.com',
      featureEnabled: true
    };

    @NgModule({
      declarations: [AppComponent],
      imports: [BrowserModule],
      providers: [
        { provide: APP_CONFIG, useValue: productionConfig } // Providing the config object
      ],
      bootstrap: [AppComponent]
    })
    export class AppModule { }

    // app.component.ts
    import { Component, Inject } from '@angular/core';
    import { APP_CONFIG, AppConfig } from './app.config';

    @Component({
      selector: 'app-root',
      template: `
        <h1>App Config</h1>
        <p>API URL: {{ config.apiUrl }}</p>
        <p>Feature Enabled: {{ config.featureEnabled }}</p>
      `
    })
    export class AppComponent {
      constructor(@Inject(APP_CONFIG) public config: AppConfig) {} // Injecting using the token
    }
    ```

3.  **`string` as Token (Legacy/Rarely Used)**: Although technically possible to use strings as tokens (e.g., `{ provide: 'API_URL', useValue: '...' }`), it's highly discouraged due to potential naming collisions and lack of type safety. `InjectionToken` is the preferred way for non-class dependencies.

### How Angular's DI System Uses Tokens

When a component or service declares a dependency in its constructor, Angular performs the following steps:

1.  **Identifies the Token**:
    *   If it's a class type (e.g., `MyService`), the class itself is the token.
    *   If an `@Inject()` decorator is used (e.g., `@Inject(APP_CONFIG)`), the value passed to `@Inject()` is the token.
2.  **Searches the Injector Hierarchy**: Angular starts looking for a provider associated with that token in the current component's injector. If not found, it moves up to its parent component's injector, then its parent's, and so on, until it reaches the `EnvironmentInjector` (often provided by `AppModule` or `root`).
3.  **Instantiates/Returns Dependency**: Once a provider is found, Angular uses it to either create a new instance of the dependency (e.g., for `useClass`, `useFactory`) or return an existing value (e.g., for `useValue`, `useExisting`).

## The "Industry" Way: Advanced Token Usage in Enterprise Angular Applications

In large-scale Angular applications, DI tokens are instrumental in building highly configurable, extensible, and maintainable architectures.

### 1. Multi-Providers for Extensibility

`multi: true` is a powerful option when providing tokens, allowing multiple providers to register for the same `InjectionToken`. When injected, instead of a single instance, you get an array of all registered values. This is ideal for extension points, plugins, or configurations that can be added incrementally.

**Example: Notification Handlers**

Imagine an application with various ways to display notifications (e.g., console, toast, modal).

```typescript
// notification.token.ts
import { InjectionToken } from '@angular/core';

export interface NotificationHandler {
  handle(message: string): void;
}

export const NOTIFICATION_HANDLERS = new InjectionToken<NotificationHandler[]>(
  'NOTIFICATION_HANDLERS'
);

// console-notification.handler.ts
import { Injectable } from '@angular/core';
import { NotificationHandler } from './notification.token';

@Injectable()
export class ConsoleNotificationHandler implements NotificationHandler {
  handle(message: string): void {
    console.log(`[Console Notification]: ${message}`);
  }
}

// toast-notification.handler.ts
import { Injectable } from '@angular/core';
import { NotificationHandler } from './notification.token';

@Injectable()
export class ToastNotificationHandler implements NotificationHandler {
  handle(message: string): void {
    // In a real app, this would use a toast library
    alert(`[Toast Notification]: ${message}`);
  }
}

// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { NOTIFICATION_HANDLERS, NotificationHandler } from './notification.token';
import { ConsoleNotificationHandler } from './console-notification.handler';
import { ToastNotificationHandler } from './toast-notification.handler';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [
    {
      provide: NOTIFICATION_HANDLERS,
      useClass: ConsoleNotificationHandler,
      multi: true // Register as a multi-provider
    },
    {
      provide: NOTIFICATION_HANDLERS,
      useClass: ToastNotificationHandler,
      multi: true // Register another one
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

// app.component.ts
import { Component, Inject } from '@angular/core';
import { NOTIFICATION_HANDLERS, NotificationHandler } from './notification.token';

@Component({
  selector: 'app-root',
  template: `
    <button (click)="sendNotification()">Send Notification</button>
  `
})
export class AppComponent {
  constructor(
    @Inject(NOTIFICATION_HANDLERS) private handlers: NotificationHandler[]
  ) {}

  sendNotification(): void {
    this.handlers.forEach(handler => handler.handle('Hello from Angular App!'));
  }
}
```
This pattern allows different modules or features to contribute their own notification handlers without knowing about each other, greatly enhancing modularity.

### 2. Feature Toggling and Environment-Specific Configurations

Tokens are excellent for managing application configurations that vary by environment (development, staging, production) or for implementing feature toggles.

```typescript
// feature-toggles.token.ts
import { InjectionToken } from '@angular/core';

export interface FeatureToggles {
  enableNewDashboard: boolean;
  enableBetaFeatures: boolean;
}

export const FEATURE_TOGGLES = new InjectionToken<FeatureToggles>('FEATURE_TOGGLES');

// app.module.ts (or environment-specific module)
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { FEATURE_TOGGLES, FeatureToggles } from './feature-toggles.token';

const devFeatureToggles: FeatureToggles = {
  enableNewDashboard: true,
  enableBetaFeatures: true
};

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [
    { provide: FEATURE_TOGGLES, useValue: devFeatureToggles }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

// my-feature.component.ts
import { Component, Inject } from '@angular/core';
import { FEATURE_TOGGLES, FeatureToggles } from './feature-toggles.token';

@Component({
  selector: 'app-my-feature',
  template: `
    <div *ngIf="toggles.enableNewDashboard">
      <h2>New Dashboard (Feature Enabled)</h2>
      <!-- New dashboard content -->
    </div>
    <div *ngIf="!toggles.enableNewDashboard">
      <h2>Old Dashboard</h2>
      <!-- Old dashboard content -->
    </div>
    <button *ngIf="toggles.enableBetaFeatures">Try Beta Features</button>
  `
})
export class MyFeatureComponent {
  constructor(@Inject(FEATURE_TOGGLES) public toggles: FeatureToggles) {}
}
```
By switching the `useValue` for `FEATURE_TOGGLES` in different build configurations, you can easily control features without changing component code.

### 3. Third-Party Library Integration

When integrating third-party JavaScript libraries that don't come with Angular services, `InjectionToken` is the standard way to make them injectable.

**Example: Lodash**

```typescript
// lodash.token.ts
import { InjectionToken } from '@angular/core';
import * as _ from 'lodash'; // Assuming lodash is installed and imported

export const LODASH = new InjectionToken<typeof _>('lodash');

// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { LODASH } from './lodash.token';
import * as _ from 'lodash'; // Import lodash here to provide it

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [
    { provide: LODASH, useValue: _ }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

// data.service.ts
import { Injectable, Inject } from '@angular/core';
import { LODASH } from './lodash.token';
import * as _ from 'lodash'; // For type inference

@Injectable({
  providedIn: 'root'
})
export class DataService {
  constructor(@Inject(LODASH) private lodash: typeof _) { }

  sortData(data: any[], key: string): any[] {
    return this.lodash.sortBy(data, key);
  }
}
```

### 4. Custom Logger Implementations (Advanced)

Using multi-providers with tokens can allow for a pluggable logging system, where different log appenders (console, server, analytics) can be registered.

```typescript
// logger.token.ts
import { InjectionToken } from '@angular/core';

export interface LogAppender {
  log(message: string, level: 'info' | 'warn' | 'error'): void;
}

export const LOG_APPENDERS = new InjectionToken<LogAppender[]>('LOG_APPENDERS');

// console-appender.ts
import { Injectable } from '@angular/core';
import { LogAppender } from './logger.token';

@Injectable()
export class ConsoleAppender implements LogAppender {
  log(message: string, level: 'info' | 'warn' | 'error'): void {
    console[level](`[Console Appender]: ${message}`);
  }
}

// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { LOG_APPENDERS } from './logger.token';
import { ConsoleAppender } from './console-appender';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  providers: [
    { provide: LOG_APPENDERS, useClass: ConsoleAppender, multi: true }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }

// custom-logger.service.ts
import { Injectable, Inject } from '@angular/core';
import { LOG_APPENDERS, LogAppender } from './logger.token';

@Injectable({
  providedIn: 'root'
})
export class CustomLoggerService {
  constructor(@Inject(LOG_APPENDERS) private appenders: LogAppender[]) {}

  info(message: string): void {
    this.appenders.forEach(appender => appender.log(message, 'info'));
  }

  error(message: string): void {
    this.appenders.forEach(appender => appender.log(message, 'error'));
  }
}
```

## Common Pitfalls: What to Avoid with Angular DI Tokens

While powerful, misusing DI tokens can lead to issues in maintainability, type safety, and debugging.

### 1. Using String Literals Directly as Tokens

**Problem**: Although `InjectionToken`'s constructor takes a string description, directly using strings as `provide` values is highly discouraged because:
*   **Lack of Type Safety**: The compiler cannot verify if the provided and injected types match.
*   **Naming Collisions**: In large applications, two different teams or modules might accidentally use the same string literal for different purposes, leading to unexpected behavior.
*   **Poor Discoverability**: Hard to refactor and search for usages.

```typescript
// BAD: Prone to errors and type issues
// module.ts
{ provide: 'API_KEY', useValue: 'some-key' }
// component.ts
constructor(@Inject('API_KEY') private apiKey: string) {}
```
**Solution**: Always use `new InjectionToken<T>('description')` for non-class tokens. The description is purely for debugging; the `InjectionToken` instance itself is the unique key.

```typescript
// GOOD: Type-safe and unique
// api-key.token.ts
export const API_KEY = new InjectionToken<string>('API_KEY');
// module.ts
{ provide: API_KEY, useValue: 'some-key' }
// component.ts
constructor(@Inject(API_KEY) private apiKey: string) {}
```

### 2. Not Understanding `multi: true` vs. Single Provider

**Problem**: Confusing `multi: true` with standard providers can lead to unexpected overwriting of dependencies or receiving an array when a single instance was expected.
*   If `multi: true` is not used, and multiple providers try to register for the same token, the *last one registered* in the injector hierarchy wins, effectively overwriting previous ones.

**Solution**:
*   Use `multi: true` explicitly when you intend to collect multiple values for a single token into an array.
*   Understand that if `multi: true` is omitted, subsequent providers for the same token will replace earlier ones at the same injector level.

### 3. Over-using Tokens for Classes

**Problem**: Sometimes developers might default to `InjectionToken` even when a class can serve as the token. While not strictly "wrong," it adds unnecessary boilerplate and can make the code less idiomatic.

```typescript
// Less Idiomatic: Extra boilerplate for a class
export const MY_SERVICE_TOKEN = new InjectionToken<MyService>('MyService');
// providers: [{ provide: MY_SERVICE_TOKEN, useClass: MyService }]
// constructor(@Inject(MY_SERVICE_TOKEN) private myService: MyService) {}
```
**Solution**: For services (classes with `@Injectable`), let the class itself be the token. It's cleaner, more readable, and Angular's DI is optimized for this.

```typescript
// Idiomatic: Simple class injection
@Injectable({ providedIn: 'root' })
export class MyService {}
// constructor(private myService: MyService) {}
```

### 4. Poorly Described `InjectionToken`s

**Problem**: The string argument in `new InjectionToken<T>('description')` is primarily for debugging. If it's vague or duplicated, it can make it harder to understand what a token represents when inspecting the injector in development tools.

**Solution**: Provide clear, unique, and descriptive strings for `InjectionToken`s.

```typescript
// GOOD: Clear description
export const FEATURE_TOGGLES = new InjectionToken<FeatureToggles>(
  'Application-wide feature toggles configuration'
);
```

### 5. Managing Global vs. Module/Component-Specific Providers

**Problem**: Inconsistent provision of tokens can lead to unexpected dependency resolution, especially with lazy-loaded modules.
*   If a token is `providedIn: 'root'`, it's a singleton application-wide.
*   If provided in a specific `@NgModule`, it's scoped to that module's injector.
*   If provided in a `@Component`, it's scoped to that component and its children, creating a new instance for each component instance.

**Solution**: Be deliberate about where you provide tokens based on their intended scope:
*   Use `providedIn: 'root'` or `providedIn: 'platform'` for application-wide singletons.
*   Provide tokens in specific feature modules if they are only relevant to that module and should have a separate instance per module.
*   Provide tokens in components for very specific, component-instance-scoped dependencies.

By understanding these nuances, developers can harness the full power of Angular's DI tokens to create robust, flexible, and scalable applications.
