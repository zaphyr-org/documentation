# Request Lifecycle

At its core, your application transforms an incoming HTTP request into an appropriate HTTP response. This process is
orchestrated by passing the request through a pipeline of middleware, controllers, and event dispatchers. Each layer of
this lifecycle plays a specific role, middleware handles cross-cutting concerns like sessions or CSRF protection,
controllers contain the business logic, and events are triggered at critical stages to allow for extensibility and
custom behavior.

But let’s go beyond the basics and take a closer look at the full request lifecycle to understand how each component
works together in detail.

## Overview

The request lifecycle in a ZAPHYR application follows these high-level steps:

1. The request enters the application through the `public/index.php` file
2. The `Zaphyr\Framework\Application` class handles the request
3. The `Zaphyr\Framework\Kernel\HttpKernel` processes the request
4. The `Zaphyr\Router\Router` routes the request to the appropriate controller
5. The response is emitted back to the client

## Request lifecycle diagram

The following diagram illustrates the request lifecycle in a ZAPHYR application:

![request lifecycle](https://zaphyr.org/images/docs/request-lifecycle.svg)

## Detailed request lifecycle

### 1. Entry point

All requests to a ZAPHYR application enter through the `public/index.php` file. This file:

- Creates an instance of the `Zaphyr\Framework\Application` class
- Binds important interfaces to the container:
    - `Zaphyr\Framework\Contracts\Kernel\HttpKernelInterface` - the main interface for handling HTTP requests
    - `Zaphyr\Framework\Contracts\Kernel\ConsoleKernelInterface` - the interface for handling console commands
    - `Zaphyr\Framework\Contracts\Exceptions\Handlers\ExceptionHandlerInterface` - the interface for handling exceptions
    - `Zaphyr\HttpEmitter\Contracts\EmitterInterface` - the interface for emitting responsess
- Calls the `Application::handleRequest` method with the server request

### 2. Application handling

The `Zaphyr\Framework\Application` class is the main entry point for the ZAPHYR framework. When `handleRequest` is
called:

- It gets the `Zaphyr\Framework\Contracts\Kernel\HttpKernelInterface` from the container
- It calls the kernel's `handle` method with the request
- It uses an `Zaphyr\HttpEmitter\Contracts\EmitterInterface` to emit the response back to the client

### 3. HttpKernel processing

The `Zaphyr\Framework\Kernel\HttpKernel` class is responsible for processing the request:

- It bootstraps the application if it hasn't been bootstrapped yet
- It dispatches a `Zaphyr\Framework\Events\Http\RequestStartingEvent`
- It tries to handle the request using the `Zaphyr\Router\Router`
- If an exception occurs, it handles it with the `App\Exceptions\ExceptionHandler`and dispatches a
  `Zaphyr\Framework\Events\Http\RequestFailedEvent`
- It dispatches a `Zaphyr\Framework\Events\Http\RequestFinishedEvent` and returns the response

### 4. Bootstrapping

The bootstrapping process sets up the application by registering several boot providers:

#### 4.1 EnvironmentBootProvider

- Loads environment variables from the `.env`.
- If the [configuration files are cached](/docs/framework/latest/configuration#cache-configuration), the `.env` file is
  not loaded

#### 4.2 ConfigBootProvider

- Loads configuration files
- Sets the application environment, timezone, and encoding
- Binds the `Zaphyr\Config\Contracts\ConfigInterface` to the container

#### 4.3 ExceptionBootProvider

- Registers the `App\Exceptions\ExceptionHandler`
- Turns off `display_errors` if not in testing environment

#### 4.4 RouterBootProvider

- Creates a new `Zaphyr\Router\Router` instance
- Sets the container on the router
- Sets controller routes, middleware, and route patterns
- Binds the `Zaphyr\Router\Contracts\RouterInterface` to the container

#### 4.5 RegisterServicesBootProvider

- Registers service providers:
    - `Zaphyr\Framework\Providers\LoggingServiceProvider`
    - `Zaphyr\Framework\Providers\EncryptionServiceProvider`
    - `Zaphyr\Framework\Providers\SessionServiceProvider`
    - `Zaphyr\Framework\Providers\EventsServiceProvider`
    - And any other providers specified in the application or plugin configuration

### 5. Routing

The `Zaphyr\Router\Router` class is responsible for routing the request to the appropriate controller:

- It prepares route groups and routes
- It sets the middleware stack on the dispatcher
- It delegates the actual handling to the dispatcher

### 6. Dispatching

The `Zaphyr\Router\Dispatcher` class is responsible for dispatching the request to the appropriate handler:

- It gets the route data from the route collector
- It dispatches the request based on the HTTP method and path
- It handles different dispatch results (not found, method not allowed, found)
- If a route is found, it adds the route to the request attributes
- It sets up the middleware stack
- It processes the request through the middleware stack

### 7. Middleware processing

The request passes through several middleware:

#### 7.1 CookieMiddleware

- Processes the request
- Adds any queued cookies to the response

#### 7.2 SessionMiddleware

- Starts a session
- Adds it to the request attributes
- Processes the request
- Saves the session

#### 7.3 CSRFMiddleware

- Checks if the request is a read method, is running in test mode, is excluded from CSRF protection, or has a valid CSRF
  token
- If any of these conditions are met, it processes the request
- Adds a CSRF cookie to the response

#### 7.4 XSSMiddleware

- Cleans the query parameters and parsed body of the request
- If it detects XSS, it throws an exception
- Otherwise, it processes the request

### 8. Controller execution

After passing through the middleware stack, the request reaches the controller:

- The controller processes the request
- It returns a response

### 9. Response emission

The response is then emitted back to the client:

- The `Zaphyr\Framework\Kernel\HttpKernel` returns the response to the `Zaphyr\Framework\Application`
- The `Zaphyr\Framework\Application` uses the `Zaphyr\HttpEmitter\Contracts\EmitterInterface` to emit the response
- The response is sent back to the client

## Event System

The ZAPHYR framework uses an event system to allow for hooking into the request lifecycle:

### RequestStartingEvent

- `Zaphyr\Framework\Events\Http\RequestStartingEvent` is dispatched at the beginning of the request handling process
- Contains the request object

### RequestFailedEvent

- `Zaphyr\Framework\Events\Http\RequestFailedEvent` is dispatched when an exception occurs during the request handling
  process
- Contains the request and the exception object

### RequestFinishedEvent

- `Zaphyr\Framework\Events\Http\RequestFinishedEvent` is dispatched at the end of the request handling process
- Contains the request and the response object
