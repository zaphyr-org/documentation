# CSRF Protection

[Cross-Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) is a type of security
vulnerability that allows an attacker to perform actions on behalf of a user without their consent. To mitigate this
risk, ZAPHYR Framework provides built-in CSRF protection for your application.

## How CSRF Protection Works

ZAPHYR enables CSRF protection by default for all state-changing HTTP methods: `POST`, `PUT`, `PATCH`, and `DELETE`.
This helps prevent malicious cross-site requests by verifying that requests come from trusted sources.

When a user session is created, the framework automatically generates a CSRF token and stores it in the session. For
each incoming request that modifies server state, ZAPHYR checks whether the request includes a valid CSRF token. If the
token is missing or invalid, the request is rejected with a `403 Forbidden` response.

> [!NOTE]
> During unit testing, CSRF protection is automatically disabled. This simplifies testing of routes without requiring
> CSRF tokens.

### Accessing the CSRF Token

You can access the CSRF token from the session object in your controllers. Typically, the token is passed to a view and
included in form submissions or AJAX requests to validate the request:

```php
namespace App\Controllers;

use Psr\Http\Message\ResponseInterface;
use Zaphyr\Framework\Http\Request;
use Zaphyr\Router\Attributes\Get;
use Zaphyr\Router\Attributes\Post;

class LoginController
{    
    #[Get(path: '/login/show', name: 'login.show')]
    public function showAction(Request $request): ResponseInterface
    {
        $token = $request->getSession()->getToken();

        return view('login-form', compact('token'));
    }
    
    #[Post(path: '/login/process', name: 'login.process')]
    public function processAction(Request $request): ResponseInterface
    {
        // …
    }
}
```

In the example above, the CSRF token is retrieved using `$request->getSession()->getToken()` and passed to the view. To
protect the form, include the token as a hidden field:

```html

<form method="POST" action="/login/process">
    <input type="hidden" name="_token" value="%token%">
    <!-- Other form fields here -->
</form>
```

Make sure the hidden input field name is **_token**, as this is the default key ZAPHYR expects when validating CSRF
tokens.

### X-CSRF-TOKEN

As an alternative to embedding the CSRF token in form fields, you can include it as a custom HTTP header in your AJAX
requests. This method is particularly useful for single-page applications (SPAs) and API requests where forms aren't
directly used.

To use this approach, retrieve the CSRF token from the session and include it in the `X-CSRF-TOKEN` header of your
request. Here's an example using the JavaScript `fetch` API:

```javascript
fetch('/login/process', {
    method: 'POST',
    headers: {
        'X-CSRF-TOKEN': '%token%'
    }
});
```

ZAPHYR will automatically validate this header against the token stored in the session.

### X-XSRF-TOKEN

ZAPHYR also makes the CSRF token available as an encrypted cookie named `XSRF-TOKEN`. This allows JavaScript to access
the token directly, eliminating the need to manually inject it into every form or AJAX request.

The cookie is automatically set when the session is initialized and is especially useful when using JavaScript
libraries like [axios](https://axios-http.com/), which automatically send the token in the `X-XSRF-TOKEN` header of
every request.

To enable JavaScript access to the `XSRF-TOKEN` cookie, make sure to disable the `cookie_http_only` option in your
`config/session.yaml` configuration:

```yaml
cookie_http_only: false
```

> [!NOTE]
> Disabling the`HttpOnly` flag allows JavaScript to read cookies, which may increase exposure to client-side attacks.
> Only disable this setting if you trust the environment where the application is running.

## Exclude Paths

If you want to disable CSRF protection for specific routes, such as public API endpoints or external login handlers, you
can configure exceptions in the `config/routing.yaml` file. This allows your application to skip CSRF checks only where
necessary, rather than disabling it globally.

To exclude specific paths, define them under the `csrf_ignore` section:

```yaml
csrf_ignore:
    - '/login/process'
    - '/login/*'
    - 'https://example.com/login/process'
    - 'https://example.com/login/*'
```

You can use wildcard patterns to match multiple routes, and fully qualified URLs are also supported.

> [!NOTE]
> Be cautious when excluding paths. Only omit CSRF protection where it is absolutely safe and required, such as for
> trusted API calls.

## Disable CSRF Protection

If necessary, you can disable CSRF protection across your entire application by modifying the `config/routing.yaml`
configuration file. To do so, add the `CSRFMiddleware` to the `middleware_ignore` section:

```yaml
middleware_ignore:
    - Zaphyr\Framework\Middleware\CSRFMiddleware
```

Once this is done, CSRF protection will be disabled for all routes in your application.

> [!WARNING]
> Disabling CSRF protection is strongly discouraged in production environments as it exposes your application to
> cross-site request forgery attacks. Only disable it in very specific use cases where security is otherwise ensured.
