# Cookie

Small repository for handling cookies.

## Installation

To get started, install the cookie repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/cookie
```

## Cookie

The cookie repository is an object-oriented alternative to the `$_COOKIE` super global, which simplifies the handling or
testing of cookies in applications or frameworks. It provides a simple way to create a cookie object and to get the
cookie string.

Let's take a look at the following example:

```php
$cookie = new Zaphyr\Cookie\Cookie(
    name: 'my-cookie',
    value: 'my-value',
    expire: time() + 3600,
    path: '/',
    domain: 'example.com',
    secure: false,
    httpOnly: true,
    raw: false,
    sameSite: Zaphyr\Cookie\Cookie::RESTRICTION_LAX
);

$response = new Response();
$response = $response->withAddedHeader('Set-Cookie', (string)$cookie);
$response->getBody()->write('Hello World!');

(new SapiEmitter())->emit($response);
```

In the above example whe create a cookie object with various attributes, including the cookie name, value, expiration
time and so on. We then create a response object and add the cookie object to the response headers. Finally, we emit the
response.

> [!NOTE]
> This repository does **NOT** come with a PSR-7 or emitter implementation out of the box. Check out the
> [HTTP Message](/docs/repositories/latest/http-message) and [HTTP Emitter](/docs/repositories/latest/http-emitter)
> repositories.

The cookie object also provides a bunch of getter methods to retrieve the cookie attributes:

````php
$cookie->getName();
$cookie->getValue();
$cookie->getExpire();
$cookie->getMaxAge();
$cookie->isCleared();
$cookie->getPath();
$cookie->getDomain();
$cookie->isSecure();
$cookie->isHttpOnly();
$cookie->isRaw();
$cookie->getSameSite();
````

## CookieManager

The cookie repository also provides a powerful cookie manager, which can be used to create (long-living) cookies, remove
cookies or get a queue of cookies. In conclusion, the cookie manager is a multifaceted tool. Whether you are developing
middleware, building applications, or working with frameworks, the cookie manager enhances your control over cookies,
enabling you to create long-lasting, reliable, and secure cookie-based interactions with client browsers.

### Configuration

The cookie manager can be configured via the `__constructor()` method. The `__constructor()` method accepts the cookie
attributes as arguments. These attributes will then be used as default values for the cookie instances created by the
cookie manager:

```php
$cookieManager = new \Zaphyr\Cookie\CookieManager(
    expire: 0,
    path: '/',
    domain: null,
    secure: false,
    httpOnly: true,
    raw: false,
    sameSite: \Zaphyr\Cookie\Cookie::RESTRICTION_LAX
);
```

You can overwrite these default values for each cookie instance individually. We will take a closer look at this later.

### Create cookie

To create a cookie, use the `create()` method and pass the cookie name and value as arguments. The `create()` method
will then create a cookie instance with the given name and value:

```php
$cookie = $cookieManager->create('my-cookie', 'my-value');
```

Optionally, you can also pass some additional arguments to the `create()` method. This will overwrite the default values,
which were set in the cookie manager `__constructor()`, for this cookie instance:

```php
$cookieManager->create(
    'my-cookie',
    'my-value',
    expire: time() + 3600,
    path: '/',
    domain: 'example.com',
    secure: false,
    httpOnly: true,
    raw: false,
    sameSite: \Zaphyr\Cookie\Cookie::RESTRICTION_STRICT
);
```

### Forever cookie

The cookie repository also provides a `forever()` method, which can be used to create a cookie with a very long life
span. The `forever()` method will then create a cookie instance with an expiration date in the future (current time plus
1 year):

```php
$cookie = $cookieManager->forever('my-cookie', 'my-value');
```

Optionally, you can also pass some additional arguments to the `forever()` method. This will overwrite the default
values, which were set in the cookie manager `__constructor()`, for this cookie instance:

```php
$cookieManager->forever(
    'my-cookie',
    'my-value',
    path: '/',
    domain: 'example.com',
    secure: false,
    httpOnly: true,
    raw: false,
    sameSite: \Zaphyr\Cookie\Cookie::RESTRICTION_NONE
);
```

### Forget cookie

To remove a cookie, use the `forget()` method and pass the cookie name as an argument. The `forget()` method will then
create a cookie instance with an expiration date in the past (current time minus 1 hour):

```php
$cookieManager->forget('my-cookie');
```

Optionally, you can also pass the cookie path and domain as second and third arguments:

```php
$cookieManager->forget('my-cookie', '/', 'example.com');
```

### Queued cookies

The cookie manager provides a queue for cookies. This allows you to add cookies to the queue and then get them
later. This is useful, for example, if you want to add cookies to the response headers in a middleware.

#### Add cookie to queue

To add a cookie to the queue, use the `addToQueue()` method and pass the cookie instance as an argument:

```php
$cookieManager->addToQueue($cookieManager->create('my-cookie', 'my-value'));
```

#### Get queued cookies

To get a queued cookie, use the `getQueued()` method and pass the cookie name as an argument:

```php
$cookieManager->getQueued('my-cookie');
```

If the given cookie name does not exist in the queue, the method returns `null` by default. However, you can also pass
a default value as a second argument:

```php
$cookieManager->getQueued('my-cookie', 'my-default-value');
```

#### Get all queued cookies

To get all queued cookies, use the `getAllQueued()` method:

```php
$cookieManager->getAllQueued();
```

The method returns an array of all queued cookie instances or an empty array if no cookies have been queued.

#### Determine if a cookie has been queued

If you want to check if a cookie has been queued, use the `hasQueued()` method:

```php
$cookieManager->hasQueued('my-cookie');
```

#### Remove a cookie from the queue

To remove a cookie from the queue, use the `removeFromQueue()` method:

```php
$cookieManager->removeFromQueue('my-cookie');
```
