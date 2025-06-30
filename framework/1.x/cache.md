# Cache

ZAPHYR provides a flexible caching system that supports multiple drivers, including file-based, Redis, and in-memory
(array) caching. The system is fully configurable, allowing you to define and switch between different cache stores as
needed.

All caching operations are built on top of the [cache repository](/docs/repositories/latest/cache), which fully
implements the [PSR-16 Simple Cache](https://www.php-fig.org/psr/psr-16/) interface for maximum interoperability and
ease of use.

## Configuration

You can configure the cache system in your application by editing the `config/cache.yaml` file or using environment
variables in your `.env` file. These configurations allow you to set the default cache store and define additional cache
stores as needed.

### Default Cache Store

Set the default cache store in your `.env` file:

```bash
CACHE_DEFAULT_STORE=file
```

This store will be used when you interact with the cache without specifying a particular store. If you want to use a
different cache store for specific tasks, you can override the default in your code. Details on using specific cache
stores are provided later in this document.

### Cache Stores

You can define multiple cache stores with their respective configurations inside the `config/cache.yaml` file. Commonly
used stores include:

- `file`
- `redis`
- `array`

Refer to the `config/cache.yaml` file for configuration details of each store. For a comprehensive overview, see
the [cache stores documentation](/docs/repositories/latest/cache#cache-stores).

## Usage

You can interact with the cache system by injecting either `Zaphyr\Cache\Contracts\CacheInterface` or
`Zaphyr\Cache\Contracts\CacheManagerInterface` into your classes, or by resolving them from the service container.

The cache system provides a clean, PSR-16 compliant API for setting, retrieving, and deleting cache entries. For a full
list of available methods, refer to the [cache repository documentation](/docs/repositories/latest/cache).

### Default Cache Store

To use the default cache store, inject the `CacheInterface`, which provides the standard PSR-16 methods `get()`,
`set()`, `delete()`, etc.:

```php
use Zaphyr\Cache\Contracts\CacheInterface;

$cache = $container->get(CacheInterface::class);
$cache->get('key');
```

### Specific Cache Store

If you need to work with a particular cache store (e.g., `redis` or `array`), use the `CacheManagerInterface` and its `cache()`
method. By default, `cache()` returns the default store:

```php
use Zaphyr\Cache\Contracts\CacheManagerInterface;

$cacheManager = $container->get(CacheManagerInterface::class);
$cacheManager->cache()->get('key');
```

You can also specify a particular cache store by passing its name as an argument to the `cache()` method. For example,
to use the `redis` cache store, you would do the following:

```php
$cacheManager->cache('redis')->set('key', 'value', 3600);
```

## Events

You can enable cache events to listen for cache operations such as `set`, `get`, and `delete`. This is useful for
monitoring cache activity, logging, or triggering custom logic when cache actions occur. To enable cache events, set the
`CACHE_EVENTS` environment variable to `true` in your `.env` file:

```bash
CACHE_EVENTS=true
```

Once enabled, the application will dispatch events during cache operations, allowing you to attach listeners as needed.
You can find the complete list of available cache events in the
[cache events documentation](/docs/repositories/latest/cache#events).
