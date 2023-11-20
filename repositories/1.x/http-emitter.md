# HTTP Emitter

_Emits [PSR-7](https://www.php-fig.org/psr/psr-7) responses to the PHP Server API._

---

[TOC]

---

## Installation

To get started, install the http-emitter repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/http-emitter
```

---

## Emitters

Emitters play a crucial role in the process of emitting a [PSR-7](https://www.php-fig.org/psr/psr-7) response.
This essential functionality is particularly relevant when your application operates under the context of a traditional
PHP server API that utilizes output buffers, such as Apache, NGINX or php-fpm.

Usually, emitters are designed to carry out the following tasks during the response emission process:

- Emit a response status line.
- Emit all response headers.
- Emit the response body.

Emitters are responsible for efficiently handling each of these critical steps, ensuring that the HTTP response is
properly structured and compliant with the standard protocol. By executing these actions seamlessly, emitters contribute
to the smooth and reliable communication between the server and the client, delivering the requested content efficiently
and accurately.

### SapiEmitter

`Zaphyr\HttpEmitter\SapiEmitter` serves as a component that takes in the response instance and leverages the built-in
PHP function `header()` to emit both the headers and the status line. Subsequently, the emitter utilizes echo to emit
the response body. This two-step process ensures that the response is properly constructed and sent to the client using
standard PHP functionalities for handling HTTP responses:

```php
$response = new Response();
$response->getBody()->write('Hello World!');

(new Zaphyr\HttpEmitter\SapiEmitter())->emit($response);
```

> [!NOTE]
> This repository does **NOT** come with a PSR-7 implementation out of the box. For a PSR-7 implementation, check out the
> [HTTP Message](/docs/repositories/latest/http-message) repository.

Within its internal implementation, `Zaphyr\HttpEmitter\SapiEmitter` performs several crucial verifications to ensure
the proper functioning and integrity of the response emission process:

- **Prevention of Headers Overwriting:** Before sending the response headers using the `header()` function, the emitter
  checks whether any headers have already been sent earlier in the execution flow. If headers have been previously sent,
  it promptly raises an `Zaphyr\HttpEmitter\Exceptions\HttpEmitterException` exception.

- **Protection Against Output Interference:** Additionally, the emitter takes precautions to prevent any interference
  with the output buffering mechanism. It checks whether any output has been sent prior to the response emission phase.
  If output has already been sent, which might occur if the application inadvertently echoes content before the response
  is ready, the emitter responds by raising an `Zaphyr\HttpEmitter\Exceptions\HttpEmitterException` exception.
  This safeguard ensures that the response body is isolated and not mixed with any other output, maintaining the integrity
  and clarity of the final response delivered to the client.

The `Zaphyr\HttpEmitter\SapiEmitter` possesses a versatile capability, making it capable of handling any response with
ease, thereby consistently returning `true` as a result.

### SapiStreamEmitter

The `Zaphyr\HttpEmitter\SapiStreamEmitter` proves to be highly beneficial when there is a need to emit a `Content-Range`.
The emitter detects the Content-Range header and emits only the bytes specified. A typical scenario where this emitter
shines is when you want to stream a specific range of bytes from a file to a client. In such cases, the client can
provide the following header as an example:

```php
$response = (new Response())->withHeader('Content-Range', 'bytes 0-2/*');
$response->getBody()->write('hello world');

(new Zaphyr\HttpEmitter\SapiStreamEmitter())->emit($response);
```

> [!NOTE]
> This repository does **NOT** come with a PSR-7 implementation out of the box. For a PSR-7 implementation, check out the
> [HTTP Message](/docs/repositories/latest/http-message) repository.

The `Zaphyr\HttpEmitter\SapiStreamEmitter` presents a constructor that accepts an integer argument, allowing you to
specify the maximum buffer length for response emission. By default, if no value is provided, the emitter sets this
maximum buffer length to **8192 bytes**:

```php
new Zaphyr\HttpEmitter\SapiStreamEmitter(8192);
```
