# Router

Robust [PSR-7](https://www.php-fig.org/psr/psr-7) router supporting
[attribute-based](https://www.php.net/manual/en/language.attributes.overview.php) routing, complete with
[PSR-15](https://www.php-fig.org/psr/psr-15/) middleware and [PSR-11](https://www.php-fig.org/psr/psr-11) container
support, all built upon the foundation of [FastRoute](https://github.com/nikic/FastRoute).

## Installation

To get started, install the router repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/router
```

## Basic usage

The provided PHP code demonstrates the usage of the router repository to create a basic routing system in a PHP
application:

```php
$router = new Zaphyr\Router\Router();

$router->add('/', ['GET'], function (Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface {
    $response = new Response();
    $response->getBody()->write('Hello World!');

    return $response;
});

try {
    $response = $router->handle(new ServerRequest($_SERVER['REQUEST_METHOD'], $_SERVER['REQUEST_URI']));
} catch (Zaphyr\Router\Exceptions\NotFoundException $exception) {
    // …handle 404
} catch (Zaphyr\Router\Exceptions\MethodNotAllowedException $exception) {
    // …handle 405
}
```

#### Code explanation

The code begins by creating a new instance of the `Zaphyr\Router\Router` class. A route is defined using the `add` method.
In this case, a route is created for the root URL ('/') with an HTTP GET request method. The third argument is a closure
that receives an instance of `Psr\Http\Message\ServerRequestInterface` (representing the incoming request) and returns
an instance of `Psr\Http\Message\ResponseInterface` (representing the response).

The router is then used to handle incoming requests. The `handle` method takes a `Psr\Http\Message\ServerRequestInterface`
instance representing the current request. This method tries to match the request to the defined routes and invokes the
associated closure when a match is found.

The code is enclosed in a try block to catch two possible exceptions that may be thrown by the router. If a route is not
found (i.e., a 404 error), a `Zaphyr\Router\Exceptions\NotFoundException` is caught. If the HTTP request method is not
allowed for a matched route (i.e., a 405 error), a `Zaphyr\Router\Exceptions\MethodNotAllowedException` is caught. You
can customize the error handling logic in these catch blocks according to your application's requirements.

> [!NOTE]
> This repository does **NOT** come with a PSR-7 implementation out of the box. For a PSR-7 implementation, check out the
> [HTTP Message](/docs/repositories/latest/http-message) repository.

## Route callables

The router repository supports a number of different types of callables to be executed upon dispatch. By default, the
router enforces a specific signature for these callables, requiring them to accept a request object implementing
`Psr\Http\Message\ServerRequestInterface` as the first argument and an optional associative array containing
[route parameters](#route-patterns) as the second argument, and expects the return of a response object
implementing `Psr\Http\Message\ResponseInterface`. The following sections describe the different types of callables and
how to use them.

### Closure

A callable can be defined as a simple closure anonymous function:

```php
$router->add('/', ['GET'], function (Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface {
    // …
});
```

### Object implementing __invoke

A callable can be defined as an object implementing the magic `__invoke` method:

```php
class MyController
{
    public function __invoke(Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface
    {
        // …
    }
}

$router->add('/', ['GET'], new MyController());
```

### Array object method

A callable can be defined as an array containing an object and a method name. The router will then invoke the method on
the object:

```php
class MyController
{
    public function indexAction(Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface
    {
        // …
    }
}

$router->add('/', ['GET'], [new MyController(), 'indexAction']);
```

### Array class method (lazy loaded)

A callable can be defined as an array containing the fully qualified class name and method name. The router will then
lazy load the class and invoke the method. The class will not be instantiated until the route is matched:

```php
class MyController
{
    public function indexAction(Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface
    {
        // …
    }
}

$router->add('/', ['GET'], [MyController::class, 'indexAction']);
```

### Class method (lazy loaded)

A callable can be defined as a string containing the fully qualified class name and method name separated by an `@`. The
router will then lazy load the class and invoke the method. The class will not be instantiated until the route is matched:

```php
class MyController
{
    public function indexAction(Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface
    {
        // …
    }
}

$router->add('/', ['GET'], 'MyController@indexAction');
```

### Class implementing __invoke (lazy loaded)

A callable can be defined as a string containing the fully qualified class name. The router will then lazy load the class
and invoke the magic `__invoke` method. The class will not be instantiated until the route is matched:

```php
class MyController
{
    public function __invoke(Psr\Http\Message\ServerRequestInterface $request): Psr\Http\Message\ResponseInterface
    {
        // …
    }
}

$router->add('/', ['GET'], MyController::class);
```

## Routes

The router repository supports two different types of defining routes. One way is to define routes using PHP methods.
The other way is to define routes using [PHP attributes](https://www.php.net/manual/en/language.attributes.overview.php).
If you define your routes using PHP methods, simply define your routes and your good to go. If you define your routes
using PHP attributes, you need to pass the attributed-based controllers to the router constructor or add them using the
`setControllerRoutes`:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Route(path: '/', methods: ['GET'])]
    public function indexAction()
    {
        // …
    }
}

$router = new Zaphyr\Router\Router(controllers: [
    Controller::class,
]);
```

As  you can see in the example above, the attributed-based controller is passed to the router constructor as an array of
controllers. But you can also add attributed-based controllers using the `setControllerRoutes` method:

```php
$router->setControllerRoutes([
    Controller::class,
]);
```

### Route methods

The router provides a number of methods for defining routes to a corresponding HTTP method. Each method takes a URL path
and a callable as arguments. Each method also returns an instance of `Zaphyr\Router\Attributes\Route` that can be used
to further configure the route:

```php
$router->any('/any', 'Controller@anyAction');
$router->get('/get', 'Controller@getAction');
$router->post('/post', 'Controller@postAction');
$router->put('/put', 'Controller@putAction');
$router->patch('/patch', 'Controller@patchAction');
$router->delete('/delete', 'Controller@deleteAction');
$router->head('/head', 'Controller@headAction');
$router->options('/options', 'Controller@optionsAction');
```

Equivalent to the methods above, it is also possible to define routes to corresponding HTTP methods using attributes:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Any('/any')]
    public function anyAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Get('/get')]
    public function getAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Post('/post')]
    public function postAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Put('/put')]
    public function putAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Patch('/patch')]
    public function patchAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Delete('/delete')]
    public function deleteAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Head('/head')]
    public function headAction()
    {
        // …
    }

    #[Zaphyr\Router\Attributes\Options('/options')]
    public function optionsAction()
    {
        // …
    }
}
```

### Route conditions

It is also possible to add additional conditions for route matching beyond the HTTP method and URI. The router
facilitates this capability by enabling the chaining of supplementary conditions to a route.

#### Scheme

Scheme matching can be added to a route using the `setScheme` method:

```php
$router->get('/get', 'Controller@getAction')->setScheme('https');
```

Or in attribute-based routing:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Get('/get', scheme: 'https')]
    public function getAction()
    {
        // …
    }
}
```

The route will only match if the incoming request's scheme matches the specified scheme (`GET: https://example.com/get`).

#### Host

Host matching can be added to a route using the `setHost` method:

```php
$router->get('/get', 'Controller@getAction')->setHost('example.com');
```

Or in attribute-based routing:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Get('/get', host: 'example.com')]
    public function getAction()
    {
        // …
    }
}
```

The route will only match if the incoming request's host matches the specified host (`GET: //example.com/get`).

#### Port

Port matching can be added to a route using the `setPort` method:

```php
$router->get('/get', 'Controller@getAction')->setPort(8080);
```

Or in attribute-based routing:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Get('/get', port: 8080)]
    public function getAction()
    {
        // …
    }
}
```

The route will only match if the incoming request's port matches the specified port (`GET: https://example.com:8080/get`).

Conditions can also be added to route groups. See the [Group conditions](#group-conditions) section for more information.

### Route patterns

The router repository supports route patterns. If a route has dynamic route parameters, the segments will be extracted
from the URL and passed to the route's callable as an associative array of parameters:

```php
$router->get('/users/get/{id}', function (Zaphyr\HttpMessage\ServerRequest $request, array $params) {
    $id = $params['id'];
    // …
});
```

Route patterns in attributes-based routes are working in the same way:

```php
class UserController
{
    #[Zaphyr\Router\Attributes\Route(path: '/users/get/{id}')]
    public function getAction(\Psr\Http\Message\ServerRequestInterface $request, array $params)
    {
        $id = $params['id'];
        // …
    }
}
```

#### Predefined route patterns

The route patterns also allow you to define which format a route parameter must match before the route is considered a
match:

```php
$router->get('/users/get/{id:numeric}', function (Zaphyr\HttpMessage\ServerRequest $request, array $params) {
    $id = $params['id'];
    // …
});
```

The router repository provides a number of predefined route patterns that can be used to restrict the format of route
parameters:

- numeric
- alpha
- alphanum
- alphanum_dash

#### Custom route patterns

It is also possible to define custom route patterns. Route patterns are defined using regular expressions. Custom route
patterns can be defined using the `setRoutePatterns` method:

```php
$router->setRoutePatterns(['slug' => '[a-z0-9-]+']);
```

After defining a custom route pattern, it can be used in a route definition:

```php
$router->get('/article/{slug}', function (Zaphyr\HttpMessage\ServerRequest $request, array $params) {
    $slug = $params['slug'];
    // …
});
```

### Route names

Named routes have the advantage of being easily identifiable. Also, if you ever need to update a route path, you don't
have to change the route path everywhere in your application if you use a named route.

Routes can be named using the `setName` method and can be retrieved using the `getNamedRoute` method:

```php
$router->get('/home', 'Controller@homeAction')->setName('home');

$route = $router->getNamedRoute('home');

$route->getPath();
```

To set a name for a route in attribute-based routing, you can use the `name` parameter:

```php
class Controller
{
    #[Zaphyr\Router\Attributes\Get('/home', name: 'home')]
    public function homeAction()
    {
        // …
    }
}
```

If you need to retrieve the string path of a named route, you can use the `getPathFromName` method:

```php
$router->getPathFromName('home'); // "/home"
```

The `getPathFromName` method also accepts an array of parameters that will be used to replace the route parameters in
the route path:

```php
$router->get('/user/{id}', 'UserController@showAction')->setName('user.show');

$router->getPathFromName('user.show', ['id' => 1]); // "/user/1"
```

### List routes

<span class="badge__available">Available since v1.2.0</span>

You may want to list all the routes that are defined in the router. The `getRoutes` method returns an array of all
defined route instances:

```php
$router->getRoutes();
```

### Route callable name

<span class="badge__available">Available since v1.3.0</span>

The `getCallableName` method can be used to retrieve the callable name of a route. The callable name is a string that
contains the class name and method name of the route callable:

```php
$router->get('/user/{id}', [UserController::class, 'showAction']);

foreach($router->getRoutes() as $route) {
    echo $route->getCallableName(); // "UserController@showAction"
}
```

## Groups

Groups serve as a valuable means of structuring and organizing your route definitions. They afford us the ability to
apply conditions and a shared prefix to a collection of routes, enhancing the management and coherence of multiple
routes within the same context. This streamlines the process of configuring related routes while maintaining a
consistent set of conditions and URL prefixes:

```php
$router->group('/users', function (Zaphyr\Router\Attributes\Group $group) {
    $group->get('/all', 'UserController@allAction');
});
```

The above code defines a group of routes that all begin with the `/users` prefix. The group contains a single route that
responds to the `/users/all` URL path.

Groups can also be used in attribute-based routing:

```php
#[Zaphyr\Router\Attributes\Group('/users')]
class UserController
{
    #[Zaphyr\Router\Attributes\Get('/all')]
    public function allAction()
    {
        // …
    }
}
```

### Group conditions

As mentioned in the above [Route conditions](#route-conditions) section, it is also possible to add additional conditions
to groups.

#### Scheme

The `setScheme` method can be used to add scheme matching to a group:

```php
$router->group('/users', function (Zaphyr\Router\Attributes\Group $group) {
    $group->get('/all', 'UserController@allAction');
})->setScheme('https');
```

Or in attribute-based group routing:

```php
#[Zaphyr\Router\Attributes\Group('/users', scheme: 'https')]
class UserController
{
    #[Zaphyr\Router\Attributes\Get('/all')]
    public function allAction()
    {
        // …
    }
}
```

The group will only match if the incoming request's scheme matches the specified scheme
(`GET: https://example.com/users/all`).

#### Host

The `setHost` method can be used to add host matching to a group:

```php
$router->group('/users', function (Zaphyr\Router\Attributes\Group $group) {
    $group->get('/all', 'UserController@allAction');
})->setHost('example.com');
```

Or in attribute-based group routing:

```php
#[Zaphyr\Router\Attributes\Group('/users', host: 'example.com')]
class UserController
{
    #[Zaphyr\Router\Attributes\Get('/all')]
    public function allAction()
    {
        // …
    }
}
```

The group will only match if the incoming request's host matches the specified host (`GET: //example.com/users/all`).

#### Port

The `setPort` method can be used to add port matching to a group:

```php
$router->group('/users', function (Zaphyr\Router\Attributes\Group $group) {
    $group->get('/all', 'UserController@allAction');
})-setPort(8080);
```

Or in attribute-based group routing:

```php
#[Zaphyr\Router\Attributes\Group('/users', port: 8080)]
class UserController
{
    #[Zaphyr\Router\Attributes\Get('/all')]
    public function allAction()
    {
        // …
    }
}
```

The group will only match if the incoming request's port matches the specified port
(`GET: https://example.com:8080/users/all`).

## Middleware

The router contains a [PSR-15](https://www.php-fig.org/psr/psr-15/) compliant middleware dispatcher, which means that it
can handle the invocation of a stack of middleware. Because the router is build around PSR-15, middleware and controllers
are handled in a [single pass approach](https://www.php-fig.org/psr/psr-15/meta/#52-single-pass-lambda). This means that
all middleware is passed a request object and a request handler, but is expected to return its own response object or
pass off to the next middleware in the stack by calling the `handle` method on the request handler.

A good example middleware is a middleware that checks if the user is authenticated:

```php
class AuthMiddleware implements Psr\Http\Server\MiddlewareInterface
{
    public function process(Psr\Http\Message\ServerRequestInterface $request, Psr\Http\Server\RequestHandlerInterface $handler): Psr\Http\Message\ResponseInterface
    {
        // If the user is not authenticated, return a 401 response.
        if ($isAuthenticated === false) {
            return new Response(401);
        }

        // The user is authenticated, so we can continue.
        return $handler->handle($request);
    }
}
```

As you can see, the middleware implements the `Psr\Http\Server\MiddlewareInterface` interface. The request argument is
the first argument of the `process` method. The second argument is a `Psr\Http\Server\RequestHandlerInterface` instance,
so that you can continue the request handling process by calling the `handle` method on the `$handler` instance and trigger
the next middleware in the stack.

### Set middleware

Middleware can be defined to run in three different ways:

1. On the router – The middleware will run on every route that is defined on the router.
2. On a group – The middleware will run on every route that is defined in a group.
3. On a route – The middleware will run on a specific route.

Using the above example middleware, it is possible to wrap all routes with the `AuthMiddleware`:

```php
$router->setMiddleware([new AuthMiddleware()]);
```

To lock down a group of routes, you can add the middleware to the group:

```php
$router->group('/admin', function (Zaphyr\Router\Attributes\Group $group) {
    $group->get('/dashboard', 'AdminController@dashboardAction');
    $group->get('/users', 'AdminController@usersAction');
})->setMiddleware([new AuthMiddleware()]);
```

Middleware can also be added to attribute-based groups:

```php
#[Zaphyr\Router\Attributes\Group('/admin', middlewares: [AuthMiddleware::class])]
class AdminController
{
    // …
}
```

To lock down a single route, you can add the middleware to the route:

```php
$router->get('/admin/dashboard', 'AdminController@dashboardAction')->setMiddleware([new AuthMiddleware()]);
```

Or in attribute-based routing:

```php
class AdminController
{
    #[Zaphyr\Router\Attributes\Get('/admin/dashboard', middlewares: [AuthMiddleware::class])]
    public function dashboardAction()
    {
        // …
    }
}
```

### Middleware dependencies

You can pass middleware objects to the `setMiddleware` method or `middleware` attribute property, but you can also pass
class-strings. Passing class-strings has two advantages: First, the middleware is lazy-loaded, which means that the
middleware is only instantiated when it is actually needed. Second, if you are using a container, the middleware will be
resolved from the container, which means that you can inject dependencies into the middleware. Read more about
dependency injection in the [dependency injection](#dependency-injection) section.

### Middleware priority

Middleware are invoked in a specific order, yet, based on the middleware internal logic, you can control whether your
code executes before or after your controller is called into action.

The order of execution is as follows:

1. Middleware that are defined on the router.
2. Middleware that are defined on a group.
3. Middleware that are defined on a route.

To determine whether your logic runs before or after your controller, you can initiate the request handler as your
initial middleware action. It will generate a response, allowing you to manipulate it as necessary before returning it:

```php
class MyMiddleware implements Psr\Http\Server\MiddlewareInterface
{
    public function process(Psr\Http\Message\ServerRequestInterface $request, Psr\Http\Server\RequestHandlerInterface $handler): Psr\Http\Message\ResponseInterface
    {
        // Do something before the controller is called

        $response = $handler->handle($request);

        // Do something after the controller is called

        return $response;
    }
}
```

## Dependency injection

The router repository supports dependency injection, by using any [PSR-11](https://www.php-fig.org/psr/psr-11/)
compatible DI container. The PSR-11 container instance can be set using the `setContainer` method:

```php
$container = new Container();
$container->set(View::class, new View());

$router->setContainer($container);
```

You can now inject dependencies into your route controllers:

```php
class PageController
{
    public function __construct(protected View $view)
    {
    }

    public function homeAction(ServerRequest $request): Response
    {
        return $this->view->render('home');
    }
}
```

> [!NOTE]
> This repository does **NOT** come with a PSR-11 implementation out of the box. For a PSR-11 implementation, check out
> the [Container](/docs/repositories/latest/container) repository.
