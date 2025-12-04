useful links: https://github.com/weiwensangsang/nestjs-best-practices?tab=readme-ov-file
# Modules

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


