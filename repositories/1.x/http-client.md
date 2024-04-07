# HTTP Client

HTTP cURL client based on [PSR-18](https://www.php-fig.org/psr/psr-18).

## Installation

To get started, install the http-client repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/http-client
```

## Configuration

The `http-client` repository is based on [cURL](https://www.php.net/manual/en/book.curl.php). To use the cURL options,
the cURL extension must be installed!

Before we can fire a request, we first need to create a [PSR-17](https://www.php-fig.org/psr/psr-17) Response and
Stream-Factory object and then pass them to the constructor of the `Zaphyr\HttpClient\Client` object:

```php
$responseFactory = new ResponseFactory();
$streamFactory  = new StreamFactory();

$client = new Zaphyr\HttpClient\Client($responseFactory, $streamFactory);
```

> [!NOTE]
> This repository does **NOT** come with a PSR-17 implementation out of the box. For a PSR-17 implementation, check out
> the [HTTP Message](/docs/repositories/latest/http-message#factories) repository.

It is also possible to override/extend the cURL options. For this purpose, an array with the options is passed as the
third parameter to the `Zaphyr\HttpClient\Client` object:

```php
$client = new Zaphyr\HttpClient\Client($responseFactory, $streamFactory, [
    CURLOPT_SSL_VERIFYHOST => true,
]);
```

By default, the following cURL options are set:

```php
[
    CURLOPT_HEADER => true,
    CURLINFO_HEADER_OUT => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_FORBID_REUSE => true,
    CURLOPT_FRESH_CONNECT => true,
];
```

## Usage

After configuring our client object, we can now send a request to a server. To do this, we use the `sendRequest()`
method of the `Zaphyr\HttpClient\Client` object:

```php
try {
    $request = $requestFactory->createRequest('GET', 'https://example.com');

    $response = $client->sendRequest($request);

    echo $response->getBody();
} catch (Psr\Http\Client\ClientExceptionInterface $exception) {
    // handle exception
}
```

> [!NOTE]
>This repository does **NOT** come with a PSR-17 implementation out of the box. For a PSR-17 implementation, check out
the [HTTP Message](/docs/repositories/latest/http-message#factories) repository.
