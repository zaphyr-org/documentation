# Middleware

Sitting between the incoming request and the outgoing response, middleware can inspect or modify the request before it
reaches the controller and also modify the response before it is returned to the client.

You can apply middleware globally to affect all routes in your application, assign it to groups of routes to share logic
among related endpoints, or attach it to individual routes for precise control.

This layered approach helps keep your code clean, modular, and easier to maintain by separating cross-cutting concerns
from business logic.

## Create Middleware

You can create middleware using the `create:middleware` console command. This command generates a new middleware class
in the `app/Middleware` directory by default. If the directory does not exist, it will be created automatically:

```bash
php bin/zaphyr create:middleware MyMiddleware
```

#### Custom Namespace

To generate the middleware in a custom namespace, you can use the `--namespace` option. For example, to place the
middleware under the `App\Http\Middleware` namespace, you would run:

```bash
php bin/zaphyr create:middleware MyMiddleware --namespace="App\Http\Middleware"
```

This command will create a new middleware class in the specified namespace, allowing you to organize your code according
to your application's structure.

#### Generated Middleware Class

The generated middleware implements the [PSR-15](https://www.php-fig.org/psr/psr-15/) `MiddlewareInterface`, which
requires you to implement the
`process` method. This method receives the request and a handler, which is responsible for calling the next middleware
or the controller. Here is an example of a generated middleware class:

```php
namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class MyMiddleware implements MiddlewareInterface
{
    /**
     * {@inheritdoc}
     */
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // Do something before the controller is called

        $response = $handler->handle($request);

        // Do something after the controller is called

        return $response;
    }
}
```

## Register Middleware

Middleware can be registered in three different scopes: global, group, and route-specific. Each scope serves a different
purpose and allows you to control how middleware is applied throughout your application.

### Global Middleware

Global middleware is applied to every request that your application handles. This is useful for tasks that need to be
performed on every request, such as logging, authentication, or CORS handling. You can register global middleware
by adding it to the `config/routing.yaml` file under the `middleware` section. By default, all middleware classes in the
`app/Middleware` directory are automatically registered as global middleware. However, you can customize this
behavior by explicitly listing the middleware classes you want to apply globally:

```yaml
middleware:
    - App\Middleware\MyMiddleware
    - App\Middleware\AnotherMiddleware
```

You can also specify middleware that should be ignored globally. This is useful for middleware that you want to
exclude from the global stack, such as debugging or logging middleware that you only want to apply conditionally:

```yaml
middleware_ignore:
    - App\Middleware\IgnoredMiddleware
```

### Group Middleware

In the [routing & controller docs](docs/framework/latest/routing-controllers), you learned how to group routes together.
Middleware can also be applied to groups of routes, allowing you to share logic among related endpoints. This is
particularly useful for applying middleware that is relevant to a specific set of routes, such as authentication or
authorization checks.

You can define middleware for a group of routes using the `Group` attribute. This allows you to specify middleware that
should be applied to all routes within that group. Here is an example of how to define a group with middleware:

```php
namespace App\Controllers;

use App\Middleware\MyMiddleware;
use Zaphyr\Router\Attributes\Group;

#[Group(path: '/user', middlewares: [MyMiddleware::class])]
class UserController
{
    // …
}
```

### Route Middleware

Route-specific middleware allows you to apply middleware to individual routes. This is useful for cases where you need
to apply middleware logic to a specific endpoint without affecting other routes. You can define route-specific
middleware using the `Get`, `Post`, `Put`, `Delete`, or other
[HTTP method attributes](/docs/framework/latest/routing-controllers#routing), along with the `middlewares`
parameter. Here is an example of how to apply middleware to a specific route:

```php
namespace App\Controllers;

use App\Middleware\MyMiddleware;
use Zaphyr\Framework\Http\Response;
use Zaphyr\Router\Attributes\Get;

class UserController
{
    #[Get(path: '/user/list', middlewares: [MyMiddleware::class])]
    public function listAction(): Response
    {
        // …
    }
}
```

## Framework Provided Middleware

ZAPHYR comes with several built-in middleware classes which run on the global scope by default. These middleware
classes handle common tasks such as session management, CSRF protection, and XSS filtering. You can find these classes
in the `Zaphyr\Framework\Middleware` namespace. These middleware are called in the order are shown in the table below.

| Middleware Class                                | Description                                                                                      |
|-------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `Zaphyr\Framework\Middleware\CookieMiddleware`  | Adds queued cookies to the response `'Set-Cookie'` header.                                       |
| `Zaphyr\Framework\Middleware\SessionMiddleware` | Manages session data across requests.                                                            |
| `Zaphyr\Framework\Middleware\CSRFMiddleware`    | Protects against Cross-Site Request Forgery attacks by validating CSRF tokens.                   |
| `Zaphyr\Framework\Middleware\XSSMiddleware`     | Filters out potentially harmful scripts from user input to prevent Cross-Site Scripting attacks. |

If you want to disable any of this middleware, add them to the `middleware_ignore` section in your `config/routing.yaml` file:

## Middleware Execution Order

Middleware in ZAPHYR is executed in the order they are registered. This means that the first middleware in the stack
will be executed first, followed by the next one, and so on, until the last middleware is reached. After the last
middleware processes the request, the response is returned in reverse order, meaning the last middleware to handle the
request will be the first to modify the response before it is sent back to the client.

This order of execution allows you to build a chain of middleware that can modify the request and response as needed,
enabling you to implement complex logic while keeping your code organized and maintainable.

## Dependency Injection

Middleware in ZAPHYR is under the control of the Dependency Injection (DI) container. This means you can inject
dependencies into your middleware class constructor, allowing you to use services, repositories, or any other
dependencies you need.
<!-- @todo add link to service container docs -->

```php
namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

class MyMiddleware implements MiddlewareInterface
{
    public function __construct(private MyService $myService)
    {
        $this->myService = $myService;
    }

    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // Use the injected service
        $this->myService->doSomething();

        return $handler->handle($request);
    }
}
```

## Cache Middleware

Performance is a key concern for any web application. ZAPHYR allows you to cache middleware configurations to reduce
overhead and improve request handling speed.

When middleware is cached, ZAPHYR stores the current configuration in a file. This avoids re-evaluating middleware on
every request, which can significantly boost performance, especially in applications with many routes and middleware
layers. To cache middleware configurations, run:

```bash
php bin/zaphyr routes:middleware:cache
```

This command generates a cache file in the `storage/cache` directory that contains your current middleware setup. ZAPHYR
will automatically use this cached configuration for subsequent requests. If you modify any middleware after caching, be
sure to clear the cache so your changes take effect. To clear the cached middleware configuration, use:

```bash
php bin/zaphyr routes:middleware:clear
```
