# Container

Powerful auto wiring dependency injection container including [PSR-11](https://www.php-fig.org/psr/psr-11/).

## Installation

To get started, install the container repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/container
```

## Basic usage

The container enables you to conveniently register services, whether they have dependencies or not, and access
them at a later time. It functions as a registry, which, when utilized effectively, empowers you to implement the
[Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) design pattern.

Let's take a look at the following example:

```php
class Foo
{
    public function __construct(public Bar $bar)
    {
    }
}

class Bar{}
```

In the example above, we have two classes: `Foo` and `Bar`. `Foo` has a dependency on `Bar` because it expects an
instance of `Bar` to be passed to its constructor.

Normally, you would need to perform the following steps in order to return a fully configured instance of Foo:

```php
$bar = new Bar();
$foo = new Foo($bar);
```

When dealing with nested dependencies, this process can quickly become cumbersome and difficult to manage.
However, with the container, obtaining a fully configured instance of `Foo` is as straightforward as requesting
it from the container:

```php
$container = new Zaphyr\Container\Container();

$foo = $container->get(Foo::class);
$foo->bar;
```

Thanks to automatic dependency injection (auto-wiring), the container can automatically resolve the dependencies
of objects in many cases.

However, if you create classes that implement an interface, and you want to type-hint that interface, for example in a
class constructor, you need to [instruct the container on how to resolve that interface](#binding-interfaces).

Additionally, when building larger applications with many dependencies, it is advisable to use
[service providers](#service-providers).

## Binding

### Binding aliases

The container allows you to define aliases for your container bindings. To create an alias, you pass a string
as the first parameter to the `bind` method. This string becomes your `alias`. As the second parameter, you pass the
class-string or closure that should be associated with this container binding to the `bind` method:

```php
$container->bind('foo', Foo::class);
$container->get('foo');
```

### Binding factories

The container has also the ability to accept any callable function that acts as a factory to resolve your
classes. This approach offers the highest performance when resolving objects since it avoids the need for inspecting
the definitions. However, it does limit the level of flexibility you can leverage. If your container bindings become
more complex, you should [consider using service providers](#service-providers).

To illustrate this concept using the previous example, you can define it within the container in the following
manner:

```php
$container->bind(Foo::class, function (Zaphyr\Container\Container $container) {
    return new Foo($container->get(Bar::class));
});
```

### Binding interfaces

Let's take a look at the following code example:

```php
class Foo
{
    public function __construct(public BarInterface $bar)
    {
    }
}

interface BarInterface{}
class Bar implements BarInterface{}
```

The `Foo` class represents a class that has a dependency on `BarInterface`. The BarInterface interface serves as a
contract that classes must adhere to. The `Bar` class implements the `BarInterface` interface.

We now bind the `BarInterface` to the `Bar` class within the container using the `bind` method. This means that
whenever the container needs to resolve an instance of `BarInterface`, it will use the `Bar` class:

```php
$container->bind(BarInterface::class, Bar::class);
$container->get(Foo::class);
```

We now have `Foo` depending on an implementation of `BarInterface`.

### Binding singletons

The `bindSingleton` method in the container binds a class or interface that should be resolved only once. When a
singleton binding is resolved, the same object instance will be returned whenever the container is called again for
that binding:

```php
$container->bindSingleton(BarInterface::class, Bar::class);
```

To check if a container binding is a singleton object, you can use the `isSingleton` method:

```php
$container->isSingleton(BarInterface::class);
```

### Binding instances

<span class="badge badge-soft badge-info">Available since v1.1.0</span>

The `bindInstance` method in the container binds a class or interface to an already existing instance. When a binding
is resolved, the same object instance will be returned whenever the container is called again for that binding:

```php
$container->bindInstance(BarInterface::class, new Bar());
$container->get(BarInterface::class);
```

## Tagging

Container services can also be tagged. For example, if you want to associate all resolved bindings of a specific
"category", you can do so using the `tag` method:

```php
$container->bind(Foo::class);
$container->bind(Bar::class);

$container->tag([Foo::class, Bar::class], ['group']);
```

Once you have tagged all the desired services, you can easily resolve them using the `tagged` method:

```php
$container->bind(Baz::class, function (Zaphyr\Container\Container $container) {
    return new Baz(...$container->tagged('group'));
});
```

You can also add multiple tags:

```php
$container->tag(Foo::class, ['groupOne', 'groupTwo']);
```

## Extending Bindings

The `extend` method allows for modifying resolved services. When a service is resolved, you can execute additional
code to decorate or configure the service. The `extend` method accepts two arguments: the service class that you are
extending and a closure that should return the modified service. The closure receives the resolved service and the
container instance as parameters:

```php
$container->extend(Service::class, function (Service $service, Zaphyr\Container\Container $container) {
    return new Decorator($service);
});

$container->get(Service::class);
```

## Method injection

Occasionally, you may wish to call a method on an object instance while enabling the container to automatically inject
the dependencies required by that method. For instance, considering the following class:

```php
class Foo
{
    public function processBar(Bar $bar): array
    {
        return [
            // …
        ]
    }
}
```

Now you can invoke the `processBar` method using the `call` method of the container:

```php
$result = $container->call([Foo::class, 'processBar']);
```

The `call` method accepts any PHP callable. You can also use the container's call method to invoke a closure and
automatically inject its dependencies:

```php
$container->call(function (Foo $foo) {
    // …
});
```

## Service providers

Service providers offer the advantage of organizing your container definitions while also improving the performance of
larger applications. This is because the definitions registered within a service provider are lazily registered at the
moment when a service is accessed.

Creating a service provider is straightforward - you simply need to extend the
`Zaphyr\Container\AbstractServiceProvider` and specify what you want to register:

```php
class ExampleServiceProvider extends Zaphyr\Container\AbstractServiceProvider
{
    protected array $provides = [
        'key',
        'Some\Class',
    ];

    public function register(): void
    {
        $this->container
            ->bind('key', fn () => 'value')
            ->bind('Some\Class');
    }
}
```

To register this service provider with the container, you need to pass an instance of your provider to the
`registerServiceProvider` method:

```php
$container->registerServiceProvider(new ExampleServiceProvider());
```

The register method is only called when one of the aliases in the `$provides` array is requested by the container
service. As a result, the items provided by the service provider are not actually registered until they are needed.
This approach enhances the performance of larger applications, particularly as the dependency map expands.

### Bootable service providers

If there is specific functionality that needs to be executed when the service provider is added to the container,
e.g. including configuration files, we can make the service provider bootable by implementing the
`Zaphyr\Container\Contracts\BootableServiceProviderInterface`:

```php
class ExampleServiceProvider
    extends Zaphyr\Container\AbstractServiceProvider
    implements Zaphyr\Container\Contracts\BootableServiceProviderInterface
{
    protected array $provides = [];

    public function boot(): void
    {
        // …
    }

    public function register(): void
    {
        // …
    }
}
```
