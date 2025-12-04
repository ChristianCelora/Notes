useful links: https://github.com/weiwensangsang/nestjs-best-practices?tab=readme-ov-file

# NestJS Core Concepts Overview

| Concept                  | What It Is                                                       | Purpose                                                                | Decorators Involved                                                 |
| ------------------------ | ---------------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Module**               | Organizational unit grouping providers, controllers, and exports | Defines application structure; controls visibility via imports/exports | `@Module()`                                                         |
| **Controller**           | Class that handles incoming requests and returns responses       | Defines routing and HTTP endpoints                                     | `@Controller()`, `@Get()`, `@Post()`, `@Put()`, `@Delete()`, etc.   |
| **Provider**             | Injectable class containing business logic                       | Encapsulates reusable logic used by controllers or other providers     | `@Injectable()`                                                     |
| **Middleware**           | Functions executed before route handlers                         | Commonly used for logging, auth checks, body parsing                   | Configured via `MiddlewareConsumer`, no decorator                   |
| **Pipe**                 | Class used to transform or validate input data                   | Data validation, transformation, sanitization                          | `@Injectable()`, `@UsePipes()`                                      |
| **Guard**                | Class determining whether a request is allowed                   | Authorization, roles, route protection                                 | `@Injectable()`, `@UseGuards()`                                     |
| **Interceptor**          | Wrapper around method execution                                  | Logging, mapping results, caching, timeout control                     | `@Injectable()`, `@UseInterceptors()`                               |
| **Exception Filter**     | Class handling thrown exceptions                                 | Central error handling and custom responses                            | `@Catch()`, `@UseFilters()`                                         |
| **Parameter Decorators** | Extract data from requests                                       | Access headers, body, params, query, req/res                           | `@Body()`, `@Param()`, `@Query()`, `@Headers()`, `@Req()`, `@Res()` |
| **Custom Decorator**     | Application-specific metadata or helpers                         | Adds reusable logic or metadata inside handlers                        | `@SetMetadata()`, custom decorator functions                        |
| **Lifecycle Hooks**      | Methods called by Nest at runtime events                         | Initialization, destruction, shutdown hooks                            | `OnModuleInit`, `OnModuleDestroy`, `BeforeApplicationShutdown`      |


### Provider scope

| Concept            | What It Is                     | Purpose                                                          | Decorators Involved                       |
| ------------------ | ------------------------------ | ---------------------------------------------------------------- | ----------------------------------------- |
| **Provider Scope** | Lifetime behavior of providers | Controls instance creation: singleton, transient, request-scoped | `scope: Scope.REQUEST`, `Scope.TRANSIENT` |
Providers can have different lifetimes:

| Scope                   | Behavior                                |
| ----------------------- | --------------------------------------- |
| **Singleton (default)** | One instance shared across app.         |
| **Transient**           | New instance for every injection.       |
| **Request-scoped**      | New instance for each incoming request. |

Singletons are preferred unless you need specific request-level state.

### Dependency Injection

Nest is built around the design pattern known as **Dependency Injection**. There are several ways to define a provider you can use: 
 - plain values
 - classes
 - asynchronous or synchronous factories.

Occasionally, you may have dependencies that don't always need to be resolved. For example, your class might depend on a **configuration object**, but if none is provided, default values should be used. In such cases, the dependency is considered optional, and the absence of the configuration provider should not result in an error.
To mark a provider as optional, use the `@Optional()` decorator in the constructor's signature.

| Concept                       | What It Is                                                   | Purpose                                                       | Decorators Involved                                            |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------- | -------------------------------------------------------------- |
| **Dependency Injection (DI)** | System to automatically create and supply class dependencies | Enables loose coupling, testability, and modular architecture | `@Inject()`, `@Optional()`, constructor injection              |
| **Custom Provider**           | Non-class provider using tokens, factories, or values        | Allows advanced injection (e.g., config, external libs)       | `provide`, `useClass`, `useValue`, `useFactory`, `useExisting` |

### Request Lifecycle
How Nest processes requests:

> Middleware → Guards → Interceptors → Pipes → Controller → Service → Response

### Modules

Every Nest application has at least one module, the **root module**, which serves as the starting point for Nest to build the **application graph**. This graph is an internal structure that Nest uses to resolve relationships and dependencies between modules and providers. While small applications might only have a root module, this is generally not the case. Modules are **highly recommended** as an effective way to organize your components.

The `@Module()` decorator takes a single object with properties that describe the module:

|               |                                                                                                                                                                                                          |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `providers`   | the providers that will be instantiated by the Nest injector and that may be shared at least across this module                                                                                          |
| `controllers` | the set of controllers defined in this module which have to be instantiated                                                                                                                              |
| `imports`     | the list of imported modules that export the providers which are required in this module                                                                                                                 |
| `exports`     | the subset of `providers` that are provided by this module and should be available in other modules which import this module. You can use either the provider itself or just its token (`provide` value) |

The module **encapsulates** providers by default, meaning you can only inject providers that are either part of the current module or explicitly exported from other imported modules. The exported providers from a module essentially serve as the module's public interface or API.

Generally, we enclose in the same module elements that are closely related to each other. Example

```typescript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

# Decorators Reference Table

### 🏛️ Controllers
| Decorator | Purpose |
|----------|---------|
| `@Controller(path?)` | Declares a controller and optional base route. |
| `@Get(path?)` | HTTP GET route. |
| `@Post(path?)` | HTTP POST route. |
| `@Put(path?)` | HTTP PUT route. |
| `@Patch(path?)` | HTTP PATCH route. |
| `@Delete(path?)` | HTTP DELETE route. |
| `@Head(path?)` | HTTP HEAD route. |
| `@Options(path?)` | HTTP OPTIONS route. |
| `@All(path?)` | Handles all HTTP methods. |

### 📩 Route Parameters
| Decorator | Purpose |
|----------|---------|
| `@Param(name?)` | Extract route parameter. |
| `@Query(name?)` | Extract query parameter. |
| `@Body(name?)` | Extract request body or field. |
| `@Headers(name?)` | Extract request header. |
| `@Req()` / `@Request()` | Full request object. |
| `@Res()` / `@Response()` | Response object (disables Nest response handling). |
| `@Ip()` | Client IP. |
| `@HostParam(name)` | Host routing parameter. |
| `@Session()` | Session object. |
| `@UploadedFile()` | File upload (single). |
| `@UploadedFiles()` | Multiple file uploads. |

### 🧱 Providers / Dependency Injection
| Decorator | Purpose |
|----------|---------|
| `@Injectable()` | Declares a provider. |
| `@Inject(token?)` | Injects by custom token. |
| `@Optional()` | Makes dependency optional. |
| `@Global()` | Module becomes globally available. |

### 📦 Modules
| Decorator | Purpose |
|----------|---------|
| `@Module({})` | Defines a module. |
| `DynamicModule` | Returned from module factories (type helper). |

### 🔧 Filters / Guards / Pipes / Interceptors
| Decorator | Purpose |
|----------|---------|
| `@UseGuards(...guards)` | Apply guard(s). |
| `@UseInterceptors(...interceptors)` | Apply interceptor(s). |
| `@UsePipes(...pipes)` | Apply pipe(s). |
| `@UseFilters(...filters)` | Apply exception filter(s). |
| `@SetMetadata(key, value)` | Attach custom metadata (used by guards/interceptors). |

> **Note:** NestJS does *not* have `@UseMiddleware`. Middleware is configured inside modules via `configure(consumer)`.

### 🛡️ Guards
| Decorator                | Purpose                            |
| ------------------------ | ---------------------------------- |
| `@UseGuards(GuardClass)` | Apply a guard.                     |
| `CanActivate`            | Guard interface (not a decorator). |

### 🔁 Interceptors
| Decorator | Purpose |
|----------|---------|
| `@UseInterceptors(InterceptorClass)` | Apply interceptors. |

### 🔧 Pipes
| Decorator              | Purpose      |
| ---------------------- | ------------ |
| `@UsePipes(PipeClass)` | Apply pipes. |

### 🚨 Exception Filters
| Decorator                  | Purpose                          |
| -------------------------- | -------------------------------- |
| `@Catch(ExceptionType?)`   | Makes class an exception filter. |
| `@UseFilters(FilterClass)` | Apply filter(s).                 |

### 🔐 Auth / Roles (Passport / Custom Guards)
| Decorator                 | Purpose                                    |
| ------------------------- | ------------------------------------------ |
| `@UseGuards(AuthGuard())` | Apply Passport authentication guard.       |
| `@Roles(...roles)`        | Set metadata for role-based authorization. |
| `@Public()`               | Mark route as public (custom).             |

### 🔄 Lifecycle Hooks
| Decorator                      | Purpose                          |
| ------------------------------ | -------------------------------- |
| `@BeforeApplicationShutdown()` | Hook before shutdown.            |
| `@OnModuleInit()`              | Called at module initialization. |
| `@OnModuleDestroy()`           | Called at module destruction.    |

### 💾 WebSockets
| Decorator                  | Purpose                           |
| -------------------------- | --------------------------------- |
| `@WebSocketGateway()`      | Declares WebSocket gateway.       |
| `@SubscribeMessage(event)` | WebSocket message handler.        |
| `@WebSocketServer()`       | Inject WebSocket server instance. |

### 📡 GraphQL (if using @nestjs/graphql)
| Decorator         | Purpose                   |
| ----------------- | ------------------------- |
| `@Resolver()`     | GraphQL resolver.         |
| `@Query()`        | GraphQL query.            |
| `@Mutation()`     | GraphQL mutation.         |
| `@Args()`         | Extract GraphQL argument. |
| `@Context()`      | GraphQL context.          |
| `@Parent()`       | Parent resolver.          |
| `@ResolveField()` | Field-level resolver.     |

### 📨 Microservices
| Decorator                  | Purpose                       |
| -------------------------- | ----------------------------- |
| `@MessagePattern(pattern)` | Handles microservice message. |
| `@EventPattern(pattern)`   | Handles event patterns.       |
| `@Payload()`               | Microservice payload.         |


