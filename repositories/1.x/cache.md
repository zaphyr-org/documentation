# Cache

A [PSR-16](https://www.php-fig.org/psr/psr-16/) simple cache implementation. It supports multiple cache stores
and allows you to easily switch between them.

## Installation

To get started, install the cache repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/cache
```

## Cache

Caching is a critical aspect of web application performance. It helps reduce response times, lower server load, and
improve the overall user experience by storing frequently accessed data.

The `Zaphyr\Cache\Cache` class provides a simple yet consistent interface for storing and retrieving cached data
efficiently. Designed to comply with the [PSR-16 standard](https://www.php-fig.org/psr/psr-16/), it supports
multiple [cache stores](#cache-stores). It is also possible to [add custom cache stores](#add-custom-cache-stores).

### Create a cache instance

To create a new cache instance, you need to specify both a cache store name and an instance of the cache store.
The cache store is responsible for handling the actual storage of cached data. In the following example, we use the
`Zaphyr\Cache\Stores\FileStore` as the cache store implementation. The cache store name becomes relevant when using
an [event dispatcher](#set-event-dispatcher)., as it is used to
identify the cache store in dispatched events.

```php
$storeName = 'file';
$storeInstance = new Zaphyr\Cache\Stores\FileStore('/path/to/cache');

$cache = new Zaphyr\Cache\Cache($storeName, $storeInstance);
```

### Remember cache item

The `remember` method is a convenient way to retrieve a cached item. If the item does not exist in the cache, it will
be created using the provided callback function and stored in the cache for a specified duration. This is particularly
useful for caching expensive operations or data that is not frequently updated.

```php
$value = $cache->remember('key', fn() => 'value', 3600);
```

The `remember` method can also be used without specifying a duration. In this case, the item will be cached indefinitely
until it is explicitly deleted or the cache is cleared.

```php
$value = $cache->remember('key', fn() => 'value');
```

> [!NOTE]
> The `remember` method is not part of the PSR-16 `Psr\SimpleCache\CacheInterface`.

### Set cache item

The `set` method is used to store a cache item with a specified key and value. You can also specify an optional
duration (in seconds) for which the item should be cached. If no duration is provided, the item will be cached
indefinitely.

```php
$isSet = $cache->set('key', 'value', 3600);
```

The value can be any type, including arrays or objects. The cache store will handle the serialization and
deserialization of the value as needed.

### Set multiple cache items

The `setMultiple` method allows you to store multiple cache items at once. You can provide an associative array where
the keys are the cache item keys, and the values are the corresponding values to be cached. You can also specify an
optional duration (in seconds) for which the items should be cached. If no duration is provided, the items will be
cached indefinitely.

```php
$isSet = $cache->setMultiple([
    'key1' => 'value1',
    'key2' => 'value2',
], 3600);
```

The values should be `iterable` and can be of any type, including arrays or objects. The cache store will handle the
serialization and deserialization of the values as needed.

### Get cache item

The `get` method retrieves a cached item by its key.

```php
$value = $cache->get('key');
```

If the item does not exist in the cache, it will return `null` by default. You can also provide a default value to be
returned if the item is not found. This is useful for providing fallback values or handling cases where the cache
item may not be present.

```php
$value = $cache->get('non-existent-key', 'default');
```

### Get multiple cache items

The `getMultiple` method retrieves multiple cached items at once. You can provide an array of keys, and it will return
an associative array where the keys are the cache item keys, and the values are the corresponding cached values.
The keys should be `iterable` and of the string type.

```php
$values = $cache->getMultiple(['key1', 'key2']);
```

If an item does not exist in the cache, its value will be `null` by default. You can also provide a default value to be
returned for each item that is not found.

```php
$values = $cache->getMultiple(['key1', 'key2'], 'default');
```

### Has cache item

The `has` method checks whether a cache item exists by its key. It returns `true` if the item is found in the cache,
and `false` otherwise.

```php
$exists = $cache->has('key');
```

### Delete cache item

The `delete` method removes a cached item by its key. If the item is successfully deleted, it returns `true`. If the
item does not exist in the cache, it returns `false`.

```php
$isDeleted = $cache->delete('key');
```

### Delete multiple cache items

The `deleteMultiple` method allows you to remove multiple cached items at once. You can provide an array of keys,
and it will attempt to delete each item. It returns `true` if all items were successfully deleted, and `false` if any
of the items could not be deleted. The keys should be `iterable` and of the string type.

```php
$isDeleted = $cache->deleteMultiple(['key1', 'key2']);
```

### Clear cache

The `clear` method removes all cached items from the cache store. It returns `true` if the cache was successfully
cleared, and `false` if the operation failed.

```php
$isCleared = $cache->clear();
```

### Set event dispatcher

The `Zaphyr\Cache\Cache` class can dispatch events when a [PSR-14 event dispatcher](https://www.php-fig.org/psr/psr-14/)
is registered. This allows you to listen to [cache-related events](#events) and perform actions based on them. To set
the
event dispatcher, you can pass an instance of a PSR-14 event dispatcher to the `Zaphyr\Cache\Cache` constructor.

```php
$storeName = 'file';
$storeInstance = new Zaphyr\Cache\Stores\FileStore(__DIR__ . '/cache');
$eventDispatcher = new EventDispatcher();

$cache = new Zaphyr\Cache\Cache($storeName, $storeInstance, $eventDispatcher);
```

> [!NOTE]
> This repository does **NOT** come with a PSR-14 implementation out of the box. For a PSR-14 implementation, check out
> the [Event Dispatcher](/docs/repositories/latest/event-dispatcher) repository.

## Cache Stores

As mentioned earlier, the `Zaphyr\Cache\Cache` class supports multiple cache stores. It is also possible to
[add custom cache stores](#add-custom-cache-stores). Each cache store is responsible for handling the actual storage
and retrieval of cached data. The following cache stores are available:

### Array Store

The `Zaphyr\Cache\Stores\ArrayStore` is a simple in-memory cache store that uses an array to store cached items. It is
suitable for testing purposes or when you need a temporary cache that does not persist across requests.

```php
$store = new Zaphyr\Cache\Stores\ArrayStore();
```

### File Store

The `Zaphyr\Cache\Stores\FileStore` is a file-based cache store that stores cached items in files on the filesystem.
If the specified directory does not exist, it will be created automatically with the appropriate permissions (`0777`).

```php
$store = new Zaphyr\Cache\Stores\FileStore('/path/to/cache');
```

#### Set file permissions

For the subdirectories where the cache files are stored, you can specify the permissions using the `permissions` option.

```php
$store = new Zaphyr\Cache\Stores\FileStore('/path/to/cache', permissions: 0755);
```

### Redis Store

The `Zaphyr\Cache\Stores\RedisStore` is a Redis-based cache store that uses the Redis database to store cached items.
It requires the `predis/predis` package to be installed via Composer. You can install it using the following command:

```bash
composer require predis/predis

```

The `Zaphyr\Cache\Stores\RedisStore` class requires a `Predis\Client` instance to be passed to its constructor. You can
create a `Predis\Client` instance using the following code:

```php
$connection = new Predis\Client([
    'scheme' => 'tcp',
    'host' => '127.0.0.1',
    'port' => 6379,
]);

$store = new Zaphyr\Cache\Stores\RedisStore($connection);
```

You can read more about the configuration options for the `Predis\Client` class in the
[Predis documentation](https://github.com/predis/predis/wiki/Connection-Parameters)

#### Set Redis prefix

You can set a prefix for the Redis keys used by the `Zaphyr\Cache\Stores\RedisStore` class. This is useful for
namespacing your cache keys and avoiding collisions with other applications that may be using the same Redis instance.

```php
$store = new Zaphyr\Cache\Stores\RedisStore($connection, prefix: 'zaphyr_cache');
```

### Custom Cache Store

You can create your own custom cache store by implementing the `Psr\SimpleCache\CacheInterface` interface. This
interface defines the methods that your cache store must implement to be compatible with the `Zaphyr\Cache\Cache`
class.

```php
class MyCustomStore implements Psr\SimpleCache\CacheInterface
{
    /**
     * {@inheritdoc}
     */
    public function get(string $key, mixed $default = null): mixed
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function set(string $key, mixed $value, \DateInterval|int|null $ttl = null): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function delete(string $key): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function clear(): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function getMultiple(iterable $keys, mixed $default = null): iterable
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function setMultiple(iterable $values, \DateInterval|int|null $ttl = null): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function deleteMultiple(iterable $keys): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function has(string $key): bool
    {
        // …
    }
}
```

## Cache Manager

The `Zaphyr\Cache\CacheManager` class is a convenient way to manage multiple cache stores. It allows you to easily
switch between different cache stores and provides a unified interface for interacting with them. The `CacheManager`
class is responsible for creating and managing cache store instances. It also provides a way to set the default cache
store and switch between different cache stores. The cache manager class needs to be configured with the cache store
configuration before it can be used. The configuration is an associative array where the keys are the cache store names
and the values are the [configuration options for each cache store](#cache-stores).

```php
$storeConfig = [
    Zaphyr\Cache\CacheManager::FILE_STORE => [
        'path' => __DIR__ . '/path/to/cache',
        'permissions' => 0777,
    ],
    Zaphyr\Cache\CacheManager::REDIS_STORE => [
        'scheme' => 'tcp',
        'host' => '127.0.0.1',
        'port' => 6379,
        'prefix' => 'zaphyr_cache',
    ],
];

$cacheManager = new Zaphyr\Cache\CacheManager($storeConfig);
```

#### Create a cache instance

To create a new cache instance using the `CacheManager`, you can use the `cache` method to get a cache instance.
You can specify the cache store name as an argument. If no store name is provided, the default cache store
`Zaphyr\Cache\Stores\FileStore` will be used.

```php
$cache = $cacheManager->cache();
$cache->get('key');
```

You can also specify a different cache store name to create a cache instance with that store. Simply pass the store name
as an argument to the `cache` method.

```php
$cache = $cacheManager->cache(Zaphyr\Cache\CacheManager::ARRAY_STORE);
$cache->get('key');
```

### Change the default cache store

You can change the default cache store by passing the store name as the second argument to the `CacheManager`
constructor.
The default cache store will be used when no store name is specified in the `cache` method.

```php
$cacheManager = new Zaphyr\Cache\CacheManager(
    $storeConfig,
    defaultStore: Zaphyr\Cache\CacheManager::REDIS_STORE
);
```

### Add custom cache stores

We already mentioned that you can create your own [custom cache store](#custom-cache-store). To add a custom cache store
to the
`CacheManager`, you can use the `addStore` method. This method takes two arguments: the store name and a closure that
returns an instance of the custom cache store. The store name should be a unique identifier for the cache store.

```php
$cacheManager->addStore('custom_store', fn() => new MyCustomCacheStore());
```

You can then create a cache instance using the custom store name.

```php
$cacheManager->cache('custom_store');
```

### Set event dispatcher to cache manager

The `Zaphyr\Cache\CacheManager` class can also dispatch events when
a [PSR-14 event dispatcher](https://www.php-fig.org/psr/psr-14/) is registered. This allows you to listen
to [cache-related events](#events) and perform actions based on them. To set the event dispatcher, you can pass an
instance of a PSR-14 event dispatcher to the `Zaphyr\Cache\CacheManager` constructor.

```php
$eventDispatcher = new EventDispatcher();

$cacheManager = new Zaphyr\Cache\CacheManager($storeConfig, eventDispatcher: $eventDispatcher);
```

> [!NOTE]
> This repository does **NOT** come with a PSR-14 implementation out of the box. For a PSR-14 implementation, check out
> the [Event Dispatcher](/docs/repositories/latest/event-dispatcher) repository.

## Events

The `Zaphyr\Cache\Cache` class dispatches the following events when a
[PSR-14 event dispatcher](https://www.php-fig.org/psr/psr-14/) is registered, either directly via the
`Zaphyr\Cache\Cache` constructor or through the `Zaphyr\Cache\CacheManager` constructor. These events are triggered when
their corresponding cache methods are called:

| Cache Event                                          | Description                                                                                      |
|------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `Zaphyr\Cache\Events\CacheHitEvent`                  | Triggered when a cache item is successfully retrieved using the `get` method.                    |
| `Zaphyr\Cache\Events\CacheMissedEvent`               | Triggered when a cache item is not found using the `get` method. (`$default` is `null`)          |
| `Zaphyr\Cache\Events\CacheWrittenEvent`              | Triggered when a cache item is successfully stored using the `set` method.                       |
| `Zaphyr\Cache\Events\CacheWriteMissedEvent`          | Triggered when a cache item fails to be written using the `set` method.                          |
| `Zaphyr\Cache\Events\CacheDeletedEvent`              | Triggered when a cache item is successfully removed using the `delete` method.                   |
| `Zaphyr\Cache\Events\CacheDeleteMissedEvent`         | Triggered when a cache item cannot be deleted using the `delete` method.                         |
| `Zaphyr\Cache\Events\CacheClearedEvent`              | Triggered when the entire cache is successfully cleared using the `clear` method.                |
| `Zaphyr\Cache\Events\CacheClearMissedEvent`          | Triggered when the cache fails to clear using the `clear` method.                                |
| `Zaphyr\Cache\Events\CacheMultipleHitEvent`          | Triggered for each item found using the `getMultiple` method.                                    | 
| `Zaphyr\Cache\Events\CacheMultipleMissedEvent`       | Triggered for each item not found using the `getMultiple` method.                                |
| `Zaphyr\Cache\Events\CacheMultipleWrittenEvent`      | Triggered when all specified items are successfully written using the `setMultiple` method.      |
| `Zaphyr\Cache\Events\CacheMultipleWriteMissedEvent`  | Triggered when one or more specified items fail to be written using the `setMultiple` method.    |
| `Zaphyr\Cache\Events\CacheMultipleDeletedEvent`      | Triggered when all specified items are successfully deleted using the `deleteMultiple` method.   |
| `Zaphyr\Cache\Events\CacheMultipleDeleteMissedEvent` | Triggered when one or more specified items fail to be deleted using the `deleteMultiple` method. |
| `Zaphyr\Cache\Events\CacheHasEvent`                  | Triggered when a cache item is found using the `has` method.                                     |
| `Zaphyr\Cache\Events\CacheHasMissedEvent`            | Triggered when a cache item is not found using the `has` method.                                 |

> [!NOTE]
> This repository does **NOT** come with a PSR-14 implementation out of the box. For a PSR-14 implementation, check out
> the [Event Dispatcher](/docs/repositories/latest/event-dispatcher) repository.
