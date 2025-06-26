# Service Providers

Service providers are a core feature of the ZAPHYR framework. They allow you to register and configure services within
your application, serving as the central place to bind classes and interfaces to the service container. This enables
powerful dependency injection and encourages a clean, modular architecture.

In addition to improving code organization, service providers enhance performance. Services defined within a provider
are lazily loaded, they’re only initialized when needed, rather than at application startup. This reduces overhead and
speeds up the boot process, especially in larger applications.

## Basic Container Concept

The service container is a powerful feature in ZAPHYR that manages the instantiation and lifecycle of objects in your
application. It is built on top of the [container repository](/docs/repositories/latest/container), providing a flexible
and efficient way to handle dependencies. The container is fully compatible with the
[PSR-11 Container Interface](https://www.php-fig.org/psr/psr-11/).

ZAPHYR’s container is deeply integrated into the framework, allowing you to register services, resolve dependencies, and
manage object lifecycles with ease. It supports both constructor injection and method injection, making it ideal for
working with complex dependency graphs. You can also rely on type-hinting to automatically resolve services, simplifying
class dependencies significantly.

Most components in the ZAPHYR ecosystem are designed against interfaces. This means you can easily swap implementations
without changing the rest of your application. Typically, an interface is used as the key in the service container,
while its concrete implementation is registered as the value. For example, if you want to use the configuration service,
you can simply type-hint `Zaphyr\Config\Contracts\ConfigInterface`, and the container will inject the corresponding
implementation:

```php
use Zaphyr\Config\Contracts\ConfigInterface;

class MyService
{
    public function __construct(private readonly ConfigInterface $config) {
        // The $config service is automatically resolved by the container.
    }
}
```

When you retrieve the service from the container, ZAPHYR automatically resolves `ConfigInterface` to its default
implementation, `Zaphyr\Config\Config`:

```php
$myService = $container->get(MyService::class);
```

> [!NOTE]
> You can explore the [`framework/Providers/`](https://github.com/zaphyr-org/framework/tree/master/src/Providers)
> directory to see the default service provider bindings available for dependency injection in your application.

## Create Providers

To create a new service provider in ZAPHYR, you can use the framework’s built-in CLI tool. The `create:provider` command
generates a boilerplate provider class that you can customize based on your application’s needs.

To generate a new provider, run the following command:

````bash
php bin/zaphyr create:provider MyProvider
````

By default, the provider will be created in the `app/Providers` directory under the `App\Providers` namespace. You can
replace `MyProvider` with any valid PHP class name. If the specified directory doesn’t exist, it will be created
automatically.

If you prefer to use a custom namespace, you can do so with the `--namespace` option. For example, to create a provider
in the `App\Services\Providers` namespace, run:

````bash
php bin/zaphyr create:provider MyProvider --namespace="App\Services\Providers"
````

All generated providers extend the `Zaphyr\Framework\Providers\AbstractServiceProvider` base class. A typical provider
class looks like this:

```php
namespace App\Providers;

use Zaphyr\Framework\Providers\AbstractServiceProvider;

class MyProvider extends AbstractServiceProvider
{
    /**
     * {@inheritdoc}
     */
    protected array $provides = [
        // …
    ];

    /**
     * {@inheritdoc}
     */
    public function register(): void
    {
        // …
    }
}
```

#### The Register Method

The `register` method is where you define and bind services to the service container. It is executed when the provider
is registered, making it the ideal place to set up dependencies your application may require.

Within this method, you can bind interfaces or class names to concrete implementations using the container’s `bind`
method. This method typically accepts two parameters: the service identifier (usually a class or interface name) and a
factory (a class name or a closure that returns an instance of the service).

For example, to register a service called `MyService`, you can do the following:

```php
public function register(): void
{
    $this->getContainer()->bind(MyService::class, fn() => new MyService());
}
```

This ensures that any time `MyService` is requested from the container, it will resolve using the logic you define here.

For a complete overview of the available binding methods and configuration options, refer to
the [container repository binding docs](/docs/repositories/latest/container#binding).

#### The provides Property

The `provides` property is an array that defines the services made available by the provider. It allows the framework to
identify which services the provider is responsible for registering.

Importantly, the register method of the provider is only executed when one of the listed services in the `$provides`
array is requested from the container. This means that the services are registered lazily, only when they are actually
needed, helping to improve performance, especially in larger applications with many dependencies.

```php
protected array $provides = [
    MyService::class,
    // Other services…
];
```

This lazy-loading mechanism can significantly reduce the application's startup time and memory usage

#### Helper Methods

The `AbstractServiceProvider` class includes several built-in methods that simplify service registration and
configuration.

You can access the service container directly using the `getContainer()` method. This enables you to bind, retrieve, or
check for the existence of services. For example, you might check whether a service is already registered using `has()`
and retrieve it using `get()`:

```php
public function register(): void
{
    if ($this->has(MyService::class)) {
        $this->get(MyService::class);
    }
}
```

Additionally, the `config()` method allows you to retrieve values from the application's configuration files. This is
particularly helpful when services require specific configuration during setup. For instance, if your configuration
contains a key like `my.service-config`, you can retrieve it with an optional default fallback:

```php
public function register(): void
{
    $this->config('my.service-config', 'default_value');
}
```

Read more about configuration in the [configuration documentation](/docs/framework/latest/configuration).

You can also interact with the application instance itself. The `AbstractServiceProvider` grants access to the
application through the `$this->application` property, which allows you to query environment-specific details:

```php
public function register(): void
{
    $this->application->getEnvironment();
}
```

These helper methods make your service providers more concise and expressive, while keeping service logic clean and
consistent.

### Bootable Providers

Bootable providers are service providers that execute specific functionality after they are registered in the service
container. This is especially useful for initializing services or performing setup tasks that must occur once the
provider is registered.

To generate a bootable provider, use the `--bootable` option with the provider creation command:

````bash
php bin/zaphyr create:provider MyBootableProvider --bootable
````

This command creates a provider that implements the `Zaphyr\Container\Contracts\BootableServiceProviderInterface`,
requiring you to define a `boot` method. The boot method is automatically called after the provider is registered,
making it the ideal place for any initialization logic.

A bootable provider might look like this:

```php
namespace App\Providers;

use Zaphyr\Framework\Providers\AbstractServiceProvider;
use Zaphyr\Container\Contracts\BootableServiceProviderInterface;

class MyBootableProvider extends AbstractServiceProvider implements BootableServiceProviderInterface
{
    /**
     * {@inheritdoc}
     */
    protected array $provides = [
        // …
    ];

    /**
     * {@inheritdoc}
     */
    public function boot(): void
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function register(): void
    {
        // …
    }
}
```

## Configure Providers

You can configure which service providers are loaded by modifying the `config/services.yaml` file. By default, ZAPHYR
automatically loads all providers within the `App\Providers` namespace. However, you can override this behavior by
explicitly listing the providers you want to include:

```yaml
providers:
    - App\Providers\MyProvider
```

The order in which providers appear in this list matters, as it determines the order in which they are registered and
booted. When using this array-based approach, you must manually specify all providers you want the application to load.

To exclude specific providers from being loaded, use the `providers_ignore` key in the same configuration file:

```yaml
providers_ignore:
    - App\Providers\MyProvider
```

This is useful when you want to prevent certain providers from being loaded without modifying the core provider list.

## Cache Providers

To enhance performance, especially in production environments, you can cache your service providers. This reduces the
overhead of scanning and loading provider classes on every request.

To cache the providers, run the following command:

```bash
php bin/zaphyr providers:cache
```

This command generates a cache file containing the current list of service providers, allowing the framework toad load
them quickly without scanning the application.

If yo need to clear the cached providers, you can use the following command:

```bash
php bin/zaphyr providers:clear
```

Whenever you modify provider configurations or register new providers, be sure to clear the cache and regenerate it to
ensure your changes take effect.
