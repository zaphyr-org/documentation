# Responses

All controller actions in a ZAPHYR application must return a [PSR-7](https://www.php-fig.org/psr/psr-7/) response
object which implements the `Psr\Http\Message\ResponseInterface`. This ensures that your application adheres to the
PSR-7 standard for HTTP messages, allowing for greater interoperability and flexibility in handling HTTP responses.

The framework provides several response types to handle different scenarios. You can use these response types
to return HTML, JSON, text, redirects, or even empty responses. Each response type is designed to simplify the process
of returning the appropriate HTTP response from your controller actions.

## Basic Response

You can return a basic response using the `Zaphyr\Framework\Http\Response` class. This class allows you to create a
response with a status code, headers, and a body. It is the most generic response type and can be used for any
purpose:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Response;
use Zaphyr\Router\Attributes\Get;

class BlogController
{
    #[Get(path: '/blog/index', name: 'blog.index')]
    public function indexAction(): Response
    {
        $response = new Response(headers: ['Content-Type' => 'text/html']);
        $response->getBody()->write('<h1>Hello World!</h1>');

        return $response;
    }
}
```

```php
return new Response(
    body: 'php://memory',
    status: 202,
    headers: ['Content-Type' => 'text/plain']
);
```

## Redirects

Sometimes, you may need to redirect a client to a different URL, for example, after a form submission or during
authentication. For this purpose, ZAPHYR provides the `Zaphyr\Framework\Http\RedirectResponse` class.

This class extends the base `Response` class and is specifically designed for handling HTTP redirects. It allows you to
set the target URL, the HTTP status code, and any additional headers, making it simple to perform both temporary and
permanent redirects.

### Basic Redirect

To perform a standard redirect (default status: `302 Found`), pass the destination URL to the constructor:

```php
return new RedirectResponse('/path/to/redirect');
```

The argument can be a relative path or an absolute URL. If you provide a relative path, it will be resolved based on the
current request's base URL. This is particularly useful for redirecting within your application without hardcoding the
full URL.

### Custom Status and Headers

If needed, you can customize the status code, such as using `301 Moved Permanently` and add custom headers:

```php
return new RedirectResponse('https://example.com', 301, ['X-Redirected' => 'true']);
```

### Redirect Using Named Routes

In larger applications, it’s best practice to generate URLs dynamically using named routes. This approach avoids
hardcoding URLs and ensures changes to routes don’t require updates across multiple controllers:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\RedirectResponse;
use Zaphyr\Router\Attributes\Get;
use Zaphyr\Router\Contracts\RouterInterface;

class RedirectController
{
    public function __construct(protected RouterInterface $router)
    {
    }

    #[Get(path: '/redirect', name: 'redirect')]
    public function redirectAction(): RedirectResponse
    {
        // Generates a URL from the named route 'welcome'
        $url = $this->router->getPathFromName('welcome');

        return new RedirectResponse($url);
    }
}
```

## Predefined Response Types

ZAPHYR provides a set of predefined response classes that extend the functionality of the base `Response` class. These
specialized responses are designed to simplify common use cases such as returning HTML views, JSON or XML data, plain
text, or empty responses.

Each predefined response type implements the `Psr\Http\Message\ResponseInterface`, ensuring full PSR-7 compliance.
Additionally, all of these response classes automatically set the appropriate `Content-Type` header along with `UTF-8`
character encoding where applicable, reducing boilerplate and improving consistency.

### HTML Response

Use the `Zaphyr\Framework\Http\HtmlResponse` class to return HTML content. It automatically sets the `Content-Type`
header
to `text/html; charset=utf-8`, making it suitable for rendering HTML documents or fragments:

```php
return new HtmlResponse('<h1>Hello, World!</h1>');
```

This response type is commonly used to return HTML views rendered by a templating engine.

### JSON Response

The `Zaphyr\Framework\Http\JsonResponse` class is designed for returning JSON-encoded data, typically for APIs or AJAX
requests. It sets the `Content-Type` header to `application/json; charset=utf-8` and handles encoding the given array or
object into a JSON string:

```php
return new JsonResponse(['message' => 'Hello, World!']);
```

This response class automatically serializes PHP arrays and objects implementing `JsonSerializable` to JSON.

If the provided data cannot be encoded into a valid JSON string (e.g., due to circular references or invalid UTF-8
characters), the `JsonResponse` class throws a `Zaphyr\Framework\Http\Exceptions\ResponseException`. This ensures that
your application can fail gracefully and respond appropriately to encoding issues.

#### Example with a custom object implementing JsonSerializable

You can return a JSON response with a custom object by implementing the `JsonSerializable` interface:

```php
class MyJsonObject implements JsonSerializable
{
    public function jsonSerialize(): mixed
    {
        return [
            'message' => 'Hello, World!',
            'status' => 'success',
        ];
    }
}

return new JsonResponse(new MyJsonObject());
```

### XML Response

To return XML-formatted data, use the `Zaphyr\Framework\Http\XmlResponse` class. It sets the `Content-Type` header to
`application/xml; charset=utf-8` and converts the provided array or object into XML:

```php
return new XmlResponse(['message' => 'Hello, World!']);
```

This is especially useful when working with services or clients that require XML rather than JSON.

### Text Response

Use the `Zaphyr\Framework\Http\TextResponse` class to return plain text. This response type sets the `Content-Type`
header to `text/plain; charset=utf-8`, making it ideal for returning raw strings or log output:

#### Basic usage

You can pass a raw string directly to the constructor:

```php
return new TextResponse('Hello, World!');
```

This is a straightforward and effective way to send plain text without any additional markup or formatting.

#### Using a Stream

Alternatively, you can pass a PSR-7 compatible stream if the text content is coming from an input stream or resource.
This is useful for returning streamed or dynamically generated text content:

```php
$stream = new Zaphyr\HttpMessage\Stream('php://input');

return new TextResponse($stream);
```

This method allows you to work with streamed data sources such as files, memory buffers, or PHP input streams, providing
greater control over response output.

### Empty Response

For cases where a response body is not needed—such as successful form submissions, background task triggers, or API
endpoints that return only a status—you can use the `Zaphyr\Framework\Http\EmptyResponse` class:

```php
return new EmptyResponse();
```

This response automatically sets the HTTP status code to `204 No Content` and omits the response body, conforming to the
semantics of RESTful APIs and reducing unnecessary response payloads.

## Status Codes

ZAPHYR provides a convenient set of predefined HTTP status codes through the `Zaphyr\Framework\Http\Utils\StatusCode`
class. These constants allow you to clearly indicate the result of an HTTP request in a readable and standardized way.

Instead of using "magic numbers" like `200`, `404`, or `500` directly, you can reference meaningful named constants such
as `StatusCode::OK`, `StatusCode::NOT_FOUND`, or `StatusCode::INTERNAL_SERVER_ERROR`.

### Using Status Code Constants

To specify a status code in a response, use the appropriate constant:

```php
$status = StatusCode::I_AM_A_TEAPOT; // 418
```

### Get the Status Message

To retrieve the standard reason phrase for a status code (e.g., "OK", "Not Found", etc.), use the static `getMessage()`
method:

```php
$message = StatusCode::getMessage(StatusCode::OK); // "OK"
```

> [!NOTE]
> All constants in the StatusCodes class map directly to their corresponding numeric HTTP status codes as defined
> in [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) and other relevant specifications.
