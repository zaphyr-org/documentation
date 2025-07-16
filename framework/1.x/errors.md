# Error Handling

Error handling is a fundamental feature of any web application, and this framework provides a clear, robust, and
developer-friendly solution for managing both errors and exceptions.

The error handling system is built on top of the popular [Whoops](https://filp.github.io/whoops/) library, which offers
detailed and user-friendly error pages during development. It allows you to quickly identify, trace, and debug issues
with rich stack traces and contextual information.

## Configuration

The error handling system can be customized through environment variables and configuration files to suit both
development and production environments. This ensures detailed debugging during development while maintaining security
in production.

### Enable Debugging

To enable debugging and show detailed error pages powered by Whoops, set the `APP_DEBUG` variable in your `.env` file:

```bash
APP_DEBUG=true
```

When debugging is enabled, exceptions and errors will be displayed with full stack traces, code previews, and request
context, useful for fast and effective debugging.

> [!WARNING]
> **Never enable debugging in production**. Always set `APP_DEBUG=false` in production environments to avoid exposing
> stack traces, internal paths, or sensitive variables to application end users.

### Blacklist Sensitive Data

To ensure sensitive information is not shown in debug screens (even in development mode), you can blacklist specific
environment or server variables. This prevents data like API keys, tokens, or encryption secrets from being exposed.

You can define the blacklist in your `config/app.yaml` file under the `debug_blacklist` section:

```yaml
debug_blacklist:
    _ENV:
        - APP_KEY
    _SERVER:
        - APP_KEY
```

In the example above, the `APP_KEY` variable is blacklisted whether it appears in the `$_ENV` or `$_SERVER`
superglobals.

### Register Exception Handler

To customize how exceptions are reported and rendered, you can register your own exception handler. This allows you to
define logic for logging, rendering custom error pages, or transforming exceptions into specific responses.

By default, every skeleton application includes a pre-defined exception handler located at
`App\Exceptions\ExceptionHandler`. This handler is automatically registered in your `app/bootstrap.php` file by binding
it to the `Zaphyr\Framework\Contracts\Exceptions\Handlers\ExceptionHandlerInterface`:

```php
$application->getContainer()->bindSingleton(
    Zaphyr\Framework\Contracts\Exceptions\Handlers\ExceptionHandlerInterface::class,
    App\Exceptions\ExceptionHandler::class
);
```

This setup allows you to easily override or extend the behavior of the framework’s exception handling pipeline.

If you do not register a custom exception handler, the framework will fall back to its built-in default
`Zaphyr\Framework\Exceptions\Handlers\ExceptionHandler`.

## Handle Exceptions

As mentioned above, every application comes with a default exception handler located at
`App\Exceptions\ExceptionHandler`. This handler has one primary responsibility: rendering HTML views for exceptions and
errors via the `renderHtmlView` method.

Take a look at your application's `resources/views/errors` directory, which contains the HTML templates for error pages.
To customize how HTTP exceptions are rendered, simply add a file named after the corresponding HTTP status code in that
directory, for example, `404.html`, `500.html`, and so on.

The framework will automatically use these templates when rendering exceptions and errors. If no specific template is
found for a given status code, a generic error template will be used instead.

### Render Exceptions

In some cases, you may want to customize how specific exceptions are rendered instead of relying on the global exception
handler. This is useful when certain exceptions should return a tailored response, such as a custom error page or a
specific JSON structure.

To achieve this, you can define a `render()` method directly on your exception class. This method receives the current
`ServerRequestInterface` and should return a valid `Response` object. When the exception is thrown, the framework will
automatically use your custom rendering logic if a `render()` method is present.

Here’s an example of a custom exception that renders a specific error view:

```php
namespace App\Exceptions;

use Exception;
use Psr\Http\Message\ServerRequestInterface;
use Zaphyr\Framework\Http\Response;

class AppException extends Exception
{
    public function render(ServerRequestInterface $request): Response
    {
        return view('errors/general', statusCode: 500);
    }
}
```

In this example, when the `AppException` is thrown, the user will be shown the `resources/views/errors/general.html`
template with a 500 HTTP status code.

### Report Exceptions

By default, exceptions are logged using the framework's built-in logging system. However, you may want to customize
how exceptions are reported, such as sending notifications or integrating with external services. For such cases, the
framework allows you to define a `report()` method directly on your custom exception classes.

The `report()` method is automatically called by the framework when the exception is thrown. This gives you full control
over how the exception is handled behind the scenes, whether it's writing to a log, sending a Slack alert, or notifying
an external service.

Here’s an example of how to implement a custom exception that reports errors to a Slack channel:

```php
namespace App\Exceptions;

use App\Services\SlackService;
use Exception;

class AppException extends Exception
{
    public function report(): void
    {
        $slack = app()->getContainer()->get(SlackService::class);
        $slack->send(
            'An error occurred: ' . $this->getMessage(),
            [
                'context' => [
                    'file' => $this->getFile(),
                    'line' => $this->getLine(),
                    'trace' => $this->getTraceAsString(),
                ]
            ]
        );
    }
}
```

### Ignore Exceptions

You may want to avoid reporting exceptions that are expected or benign, such as validation failures, 404 errors, or
specific domain logic exceptions.

To do this, list the exception types you'd like to ignore in the `ignore` section of your `config/logging.yaml` file.
The logger will silently skip these exceptions.

See the [Ignore Errors](/docs/framework/latest/logger#ignore-errors) section in the Logger documentation for full
details.

## HTTP Exceptions

The framework provides a clean and consistent way to handle HTTP exceptions, which are used to indicate specific HTTP
status codes such as `404 Not Found`, `401 Unauthorized`, or `500 Internal Server Error`. These exceptions are
especially useful for controlling the flow of your application when a particular HTTP error condition needs to be
triggered.

To throw an HTTP exception, simply instantiate the `HttpException` class with the appropriate status code and an
optional message:

```php
throw new Zaphyr\Framework\Http\Exceptions\HttpException(404, 'Page not found');
```

When this exception is thrown, the framework will automatically handle it by returning an HTML error response using a
corresponding error template from your application's `resources/views/errors` directory (e.g., `404.html`, `500.html`).
If no specific template exists for a given status code, a generic error view will be rendered.

> [!TIP]
> You can also create your own custom exceptions that extend `HttpException` to represent domain-specific errors with
> associated status codes.

### JSON Responses

If you're building a JSON API or need to return structured error responses in JSON format (e.g., for frontend
applications or third-party consumers), you can use the `buildJsonResponse()` method available on the `HttpException`
class.

This method generates a `Zaphyr\Framework\Http\Response` object containing a properly formatted JSON body and the
appropriate HTTP status code:

```php
use Zaphyr\Framework\Http\Exceptions\HttpException;

return (new HttpException(404, 'Page not found'))->buildJsonResponse();
```

This will return a response like:

```json
{
    "error": {
        "status": 404,
        "message": "Page not found"
    }
}
```

> [!TIP]
> The JSON structure can be customized by extending the `HttpException` class and overriding the
> `buildJsonResponse()` method.

#### Custom JSON Responses

You can also pass a custom `Response` instance to the `buildJsonResponse()` method if you need to modify headers, set
cookies, or adjust other response properties before returning the error response. This is particularly useful when
building APIs that require specific headers (e.g., CORS, content type, or rate limiting).

```php
use Zaphyr\Framework\Http\Response;
use Zaphyr\Framework\Http\Exceptions\HttpException;

$response = (new Response())->withHeader('X-Custom-Header', 'ApiError');

return (new HttpException(404, 'Resource not found'))->buildJsonResponse($response);
```

In this example, the final JSON response will include your custom header alongside the error details, ensuring your API
responses remain consistent and extendable.
