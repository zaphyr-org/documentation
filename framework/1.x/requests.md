# Requests

The ZAPHYR framework includes the powerful `Zaphyr\Framework\Http\Request` class, which acts as a wrapper around
the PSR-7 `Psr\Http\Message\ServerRequestInterface`. This means that any [PSR-7](https://www.php-fig.org/psr/psr-7/)
compatible request implementation can be used seamlessly within your application.

However, ZAPHYR's own `Request` class offers many convenient utility methods that make it easier to work with HTTP
requests, such as retrieving query parameters, form data, headers, uploaded files, and more.

## Interact With the Request

To interact with the incoming HTTP request in your controllers, type-hint the request object in your action
methods. ZAPHYR will automatically inject the current request instance, giving you full access to its data and methods.
<!-- @todo add link to container/session provider – how the DI resolving works -->

This allows you to retrieve query parameters, form data, headers, the request method, and more, all directly from the
controller:

```php
namespace App\Controllers;

use Psr\Http\Message\ResponseInterface;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;

class BlogController
{
    #[Get(path: '/blog/index', name: 'blog.index')]
    public function indexAction(Request $request): ResponseInterface
    {
        $method = $request->getMethod();
        $query  = $request->getQueryParams();

        // Use request data as needed...
    }
}
```

### PSR-7 Request Objects

As mentioned earlier, the `Request` class provided by ZAPHYR is a thin wrapper around the PSR-7
`ServerRequestInterface`. This means you're free to use any PSR-7 compatible request object within your application,
including in controller methods. By doing so, you can fully leverage the flexibility and interoperability of PSR-7,
while still benefiting from the convenient helper methods provided by ZAPHYR’s `Request` class when needed.

This design strikes a balance between standard compliance and developer ergonomics, giving you the best of both worlds:

```php
namespace App\Controllers;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Zaphyr\Router\Attributes\Get;

class BlogController
{
    #[Get(path: '/blog/index', name: 'blog.index')]
    public function indexAction(ServerRequestInterface $request): ResponseInterface
    {
        // Handle request using standard PSR-7 methods
    }
}
```

## Retrieve Request Data

ZAPHYR’s `Request` class offers convenient methods to access data from various parts of the HTTP request, such as query
parameters, form input, server variables, cookies, and the request body. These methods help you write clean and
expressive code without manually parsing superglobals like `$_GET` or `$_POST`.

### Access All Request Parameters

Retrieve all available parameters (from query string, parsed body, etc.):

```php
$params = $request->getParams();
$categoryParams = $request->getParams(['category']);
```

### Access a Single Request Parameter

Get a single value from the request parameters. You can also specify a default value if the parameter is missing:

```php
$category = $request->getParam('category');
$category = $request->getParam('category', 'default value');
```

### Access Server Parameters

Fetch data from the server environment, such as headers, request method, host, etc.:

```php
$serverName = $request->getServerParam('SERVER_NAME');
$serverName = $request->getServerParam('SERVER_NAME', 'default value');
```

### Access Cookie Parameters

Retrieve a specific cookie sent with the request:

```php
$cookie = $request->getCookieParam('cookie_name');
$cookie = $request->getCookieParam('cookie_name', 'default value');
```

### Access Query Parameters

Used to access data passed in the URL query string:

```php
$query = $request->getQueryParam('query_name');
$query = $request->getQueryParam('query_name', 'default value');
```

### Access Parsed Body Parameters

Used to access form data or other body content (e.g., JSON-decoded values):

```php
$body = $request->getBodyParam('body_name');
$body = $request->getBodyParam('body_name', 'default value');
```

## Method Helpers

The `Request` class provides several methods to check the HTTP request method, making it easy to handle different
types of requests. You can use either generic method checks or specific method checks for better readability,
especially in controllers or middleware.

### Generic Method Check

You can check the request method using the `isMethod` method, which accepts a string representing the HTTP method.
This method is useful if you need to check dynamically or use non-standard/custom methods:

```php
if ($request->isMethod('GET')) {
    // Handle GET request
}
```

### Specific Method Checks

For commonly used HTTP verbs, the `Request` provides dedicated helper methods:

```php
if ($request->isGet()) {
    // Handle GET request
}

if ($request->isPost()) {
    // Handle POST request
}

if ($request->isPut()) {
    // Handle PUT request
}

if ($request->isPatch()) {
    // Handle PATCH request
}

if ($request->isDelete()) {
    // Handle DELETE request
}

if ($request->isHead()) {
    // Handle HEAD request
}

if ($request->isOptions()) {
    // Handle OPTIONS request
}
```

### XHR (AJAX) Detection

You can also detect if the request was made via JavaScript (XMLHttpRequest/AJAX):

```php
if ($request->isXhr()) {
    // Handle AJAX request
}
```

## Content Type

ZAPHYR's `Request` class provides convenient methods to extract key metadata from the HTTP `Content-Type` header and
related request headers. This is particularly helpful when validating or handling incoming payloads in various formats.

### Get Content Type

Returns the simplified content type extracted from the `Content-Type` header. This method is similar to
`getMediaType()`, but may return normalized or framework-specific values:

```php
$contentType = $request->getContentType(); // e.g., 'json', 'form', 'html'
```

This is helpful when you need to quickly branch logic based on common content types, such as `json`, `form`, `html` or
`xml`.

### Get Content Charset

Retrieves the character encoding (charset) specified in the `Content-Type` header, if available:

```php
$contentCharset = $request->getContentCharset(); // e.g., 'utf-8'
```

Knowing the charset ensures that the request body is interpreted correctly, which is especially important for text-based
payloads like JSON or XML.

### Get Content Length

Returns the value of the `Content-Length` header, indicating the size of the request body in bytes:

```php
$contentLength = $request->getContentLength(); // e.g., 3487
```

This is useful for limiting request size, validating file uploads, or detecting empty request bodies.

## Media Type

When handling HTTP requests, especially in APIs or form submissions, you often need to inspect the `Content-Type` header
to understand the format of the incoming payload. ZAPHYR's `Request` class offers convenient methods to simplify this
task.

### Get Media Type

The `getMediaType()` method extracts and returns the main media type (also known as MIME type) from the `Content-Type`
header. This saves you from manually parsing headers:

```php
$mediaType = $request->getMediaType(); // e.g., 'application/json'
```

Common examples of media types include:

- `application/json`
- `application/x-www-form-urlencoded`
- `multipart/form-data`

This is especially useful when implementing content negotiation or handling different input formats conditionally.

### Get Media Type Parameters

The `getMediaTypeParams()` method returns any additional parameters that go with the media type, as an associative
array:

```php
$mediaTypeParams = $request->getMediaTypeParams();
// e.g., ['charset' => 'utf-8'] for 'application/json; charset=utf-8'
```

These parameters can include things like:

- `charset` (character encoding)
- `boundary` (used in `multipart/form-data`)

Such information can help determine how to correctly decode and process the request body.
