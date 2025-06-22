# Routing & Controllers

ZAPHYR provides a powerful routing system that allows you to define routes and associate them with controller actions.
The routing is based on the [router repository](/docs/repositories/latest/router), which is a central place for
managing routes in your application. This system is designed to be flexible and easy to use, allowing you to create
RESTful APIs, web applications, and other types of applications with ease. In this section, we will cover the basics of
routing and controllers in ZAPHYR, including how to define routes, create controllers, and manage controller caching.

## Routing

The routing system allows you to define routes that map HTTP requests to specific controller actions. Routing is
done via [PHP Attributes](https://www.php.net/manual/en/language.attributes.overview.php), which are used to annotate
controller methods with route information. This makes it easy to define routes directly in your controller classes,
keeping your code organized and maintainable. The routing system supports various HTTP methods such as GET,
POST, PUT, PATCH, DELETE, HEAD, and OPTIONS, allowing you to create routes that respond to different types of requests:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Router\Attributes\Any;
use Zaphyr\Router\Attributes\Delete;
use Zaphyr\Router\Attributes\Get;
use Zaphyr\Router\Attributes\Head;
use Zaphyr\Router\Attributes\Options;
use Zaphyr\Router\Attributes\Patch;
use Zaphyr\Router\Attributes\Post;
use Zaphyr\Router\Attributes\Put;

class UserController
{
    #[Any('/any')]
    public function anyAction(): Response
    {
        // …
    }

    #[Get('/get')]
    public function getAction(): Response
    {
        // …
    }

    #[Post('/post')]
    public function postAction(): Response
    {
        // …
    }

    #[Put('/put')]
    public function putAction(): Response
    {
        // …
    }

    #[Patch('/patch')]
    public function patchAction(): Response
    {
        // …
    }

    #[Delete('/delete')]
    public function deleteAction(): Response
    {
        // …
    }

    #[Head('/head')]
    public function headAction(): Response
    {
        // …
    }

    #[Options('/options')]
    public function optionsAction(): Response
    {
        // …
    }
}
```

You can also define routes with additional parameters such as `name`, `scheme`, `host`, and `port`. These parameters
allow you to specify the route's name, the URL scheme (HTTP or HTTPS), the host (domain), and the port number.
This is useful for creating more specific routes that can be accessed under certain conditions, such as when the
application is accessed over HTTPS or on a specific domain. The `name` parameter is particularly useful for generating
URLs in your application, as it allows you to refer to routes by name instead of hardcoding the URL paths:

```php
#[Get('/get', name: 'get', scheme: 'https', host: 'example.com', port: 8080)]
public function getAction(): Response
{
    // …
}
```

### Route Groups

ZAPHYR also supports route groups by using the `Zaphyr\Router\Attributes\Group` attribute. Route groups allow you to
define a set of routes that share common characteristics, such as a common path prefix:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;
use Zaphyr\Router\Attributes\Group;

#[Group(path: '/user')]
class UserController
{
    #[Get(path: '/list', name: 'user.list')]
    public function listAction(Request $request): Response
    {
        // …
    }
    
    #[Get(path: '/show', name: 'user.show')]
    public function showAction(Request $request): Response
    {
        // …
    }
}
```

You can also specify additional parameters for the route group, such as `scheme`, `host`, and `port`, to apply these
settings to all routes within the group. This allows you to define a common configuration for a set of routes, making
it easier to manage and maintain your routing structure. For example, if you want all routes in the `UserController`
to be accessible over HTTPS on a specific domain and port, you can define the group like this:

```php
#[Group(path: '/user', scheme: 'https', host: 'example.com', port: 8080)]
class UserController
{
    // …
}
```

### Route Patterns

Several route patterns are available in ZAPHYR to help you define routes more easily. These patterns allow you to
create routes that match specific URL structures, making it easier to handle dynamic segments in your URLs. For example,
you can use the `{id}` pattern to match a dynamic segment in the URL, such as a user ID. This pattern can be used in
controller methods to define routes that accept parameters from the URL. The parameters are automatically extracted
from the URL and passed to the controller action as an associative array, allowing you to access them easily.

```php
#[Get(path: '/user/show/{id}', name: 'user.show')]
public function showAction(Request $request, array $params): Response
{
    $id = $params['id'];
        
    // …
}
```

You can also define more specific patterns using regular expressions. For example, if you want to match a numeric ID,
you can use the `:numeric` pattern to ensure that the ID is a number. This is useful for validating parameters in your
routes and ensuring that they meet specific criteria before being passed to the controller action. The `:numeric`
pattern will only match numeric values, and if the value does not match, the route will not be matched, resulting in a
`404 Not Found response`. Here's how you can use the `:numeric` pattern in your route definition:

```php
#[Get(path: '/user/show/{id:numeric}', name: 'user.show')]
public function showAction(Request $request, array $params): Response
{
    $id = $params['id'];
        
    // …
}
```

#### Predefined Route Patterns

ZAPHYR provides several predefined route patterns that you can use to match common URL structures. These patterns
include:

- `:alpha` - Matches alphabetic characters (a-z, A-Z).
- `:alphanum` - Matches alphanumeric characters (a-z, A-Z, 0-9).
- `:alphanum_dash` - Matches alphanumeric characters and dashes (a-z, A-Z, 0-9, -).
- `:numeric` - Matches numeric values (0-9).

#### Custom Route Patterns

You can define custom route patterns by configuring the `patterns` in your `config/routing.yaml` file. The key (e.g.,
`version`) is used to define the pattern name, and the value is a regular expression that matches the desired URL
structure. For example, to define a pattern for matching version numbers, you can add the following to your
`config/routing.yaml` file:

```yaml
patterns:
    version: '\d+\.x'
```

This pattern will match a version number in the format `x.x`, where `x` is a digit. You can then use this pattern in
your route definitions to match URLs that contain version numbers. For example:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;

class ApiController
{
    #[Get(path: '/api/{version}/users', name: 'api.users')]
    public function getUsersAction(Request $request, array $params): Response
    {
        $version = $params['version'];
        
        // Handle the request based on the version
    }
}
```

### List routes

You can list all the routes defined in your application using the `routes:list` command in your terminal. This
command will display all the routes, their HTTP methods, URIs, route names, and the associated controller actions,
including details such as scheme, host, port, and added middleware. This is useful for debugging and understanding the
routing structure of your application:

```bash
php bin/zaphyr routes:list
```

## Controllers

Controllers in ZAPHYR are responsible for handling incoming requests and returning responses. They are typically
organized in a directory structure that reflects the application's namespace, making it easy to locate and manage
them. Controllers can be created using the `create:controller` command, which generates a new controller class with
the specified name and namespace. This command is useful for quickly scaffolding new controllers without having to
write boilerplate code.

Controllers are defined as classes, and each method within a controller class corresponds to a specific action that
can be performed in response to an HTTP request. The methods are annotated with route attributes to define the HTTP
methods and paths they handle. This allows you to easily map HTTP requests to specific controller actions, making your
application's routing clear and maintainable. The `Action` suffix is commonly used for controller methods but not
required.

The first argument of the controller action method is always a PSR-7 compatible `ServerRequestInterface ` object, which
contains information about the incoming request, such as headers, query parameters, and body content. You can read more
about requests in the [requests docs](/docs/framework/latest/requests). The second argument can be an array of
parameters extracted from the route, allowing you to access dynamic segments in the URL. Read more about the route
parameters in the [route patterns section](#route-patterns).

The response of a controller must be a PSR-7 compatible `ResponseInterface` object, which represents the HTTP response
that will be sent back to the client. For detailed information on how to create and manipulate responses, refer to the
[responses docs](/docs/framework/latest/responses).

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;

class BlogController
{
    #[Get(path: '/blog/index', name: 'blog.index')]
    public function indexAction(Request $request, array $params = []): Response
    {
        // Your logic to handle the request and return a response
    }
}
```

### Create Controllers

You can create a new controller in your application using the `create:controller` command. This command will
generate a new controller class for you, which you can then customize according to your application's needs.

```bash
php bin/zaphyr create:controller BlogController
```

It is also possible to specify a custom namespace for the controller. For example, if you want to place the controller
in the `App\Http\Controllers` namespace, you can use the `--namespace` option:

```bash
php bin/zaphyr create:controller BlogController --namespace="App\Http\Controllers"
```

The corresponding controller directory `app/Http/Controllers` will be created if it does not already exist. The
generated controller will be placed in the specified namespace, allowing you to organize your controllers according to
your application's structure.

#### Single Action Controllers

ZAPHYR supports single action controllers, which are controllers that contain only one method. This is useful for
creating simple controllers that handle a single action without the need for multiple methods. To create a single
action controller, you can use the `create:controller` command with the `--single` option:

```bash
php bin/zaphyr create:controller SingleActionController --single
```

This will generate a controller class with a single `__invoke` method, which can be used to handle the request.

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;

class SingleActionController
{
    #[Get(path: '/single-action', name: 'single.action')]
    public function __invoke(Request $request): Response
    {
        // Your logic to handle the request and return a response
    }
}
```

### Dependency Injection

Controllers in ZAPHYR support dependency injection, allowing you to inject services and other dependencies directly
into your controller methods. This makes it easy to manage dependencies and keep your controllers clean and focused on
handling requests. The injected dependencies are automatically resolved by the ZAPHYR PSR-11 service container. You can
inject services into your controller's constructor, and they will be automatically provided by the service container
when the controller is instantiated. This allows you to use services in your controller methods without having to
manually instantiate them, promoting better separation of concerns and making your code more testable.
<!-- @todo add link to service container docs -->

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Router\Attributes\Get;

class BlogController
{
    public function __construct(private MyService $myService)
    {
    }
    
    #[Get(path: '/blog/index', name: 'blog.index')]
    public function indexAction(): Response
    {
        // Use the injected service to handle the request
        $data = $this->myService->getData();
        
        // …
    }
}
```

### Configure Controllers

The `config/routing.yaml` file is the configuration file for routing and controllers in your application. This file
allows you to define various settings related to routing, controller directories, and patterns for route matching.
It also provides options to ignore specific controllers.

By default, all controller classes located in the `app/Controllers` directory are automatically registered as routes
in your application. This means that any controller class in this directory will be automatically scanned and registered
as a route, allowing you to define routes using attributes in your controller methods without having to manually
register them. This automatic registration simplifies the process of adding new routes and controllers to your
application, as you don't need to worry about manually configuring each route.

You can also specify which controllers to include or exclude from the automatic registration process. This is useful
if you want to have more control over which controllers are registered as routes. Simply add the controllers you want
to include in the `controllers` section of the `config/routing.yaml`. Keep in mind that when you are using the array
approach, you need to manually register every controller you want to include, as the automatic registration will
not apply to them.

```yaml
controllers:
    - App\Controllers\BlogController
    - App\Controllers\UserController
```

If you want to exclude specific controllers from being registered, you can use the `controllers_ignore` section
in the `config/routing.yaml` file. This allows you to specify controllers that should not be automatically registered
as routes, giving you more control over which controllers are included in the routing system:

```yaml
controllers_ignore:
    - App\Controllers\UserController
```

### Cache Controllers

You can cache the controllers in your application to improve performance. This is particularly useful in production
environments where you want to reduce the overhead of loading controllers on each request.

To cache the controllers, you can use the `routes:controllers:cache` command. This command will compile all the
controllers into a single file, which will be loaded on each request, reducing the overhead of loading individual
controller files:

```bash
php bin/zaphyr routes:controller:cache
```

If you need to clear the cached controllers, you can use the `routes:controllers:clear` command. This command will
remove the cached controllers file, allowing you to make changes to your controllers without having to worry about the
cached version:

```bash
php bin/zaphyr routes:controller:clear
```
