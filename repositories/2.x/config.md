# Config

_Load configuration files the easy way. This configuration loader supports
[PHP](https://en.wikipedia.org/wiki/PHP),
[INI](https://en.wikipedia.org/wiki/INI_file),
[JSON](https://en.wikipedia.org/wiki/JSON),
[XML](https://en.wikipedia.org/wiki/XML),
[YAML](https://en.wikipedia.org/wiki/YAML)
and [NEON](https://ne-on.org/) file extensions._

---

[TOC]

---

## Installation

To get started, install the config repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/config
```

---

## Loading configuration files

There are several ways to load configuration files. We will take a closer look at these in the following.

### Direct instantiation

The easiest way to load configuration files, is by direct instantiation:

```php
new Zaphyr\Config\Config(['/config/path/file.yml']);
```

### Via `load()` method

Another way is to include configuration files using the `load()` method:

```php
$config = new Zaphyr\Config\Config();
$config->load(['/config/path/file.yml']);
```

> [!WARNING]
> Please do not include untrusted configuration files with `php` extension. It could contain and execute malicious code!

### Loading configuration files from a directory

It is also possible to load complete configuration file directories. This can be done in two ways.

The easiest way to do this, is to pass the configuration directory path to the constructor:

```php
$config = new Zaphyr\Config\Config(['./config/path']);
```

You can also use the `load()` method to load configuration file directories:

```php
$config = new Zaphyr\Config\Config();
$config->load(['./config/path']);
```

> [!NOTE]
> Files are parsed and loaded depending on the file extension. When loading a directory,
> the path is `glob`ed and files are loaded in by name alphabetically.

> [!IMPORTANT]
> It is not possible to load nested configuration file directories!

---

## Get configuration values

Getting configuration values can be done via the `get()` method.

Let's say you have an `app.yml` configuration file in your `/config/path` directory
and the contents of your file looks like this:

```yaml
# /config/path/app.yml
name: "My awesome app"
```

Now you load the configuration file as described above and use the `get()` method to get the name of your application:

```php
$config = new Zaphyr\Config\Config(['/config/path']);
$config->get('app.name'); // My awesome app
```

As you can see, the name of the configuration file, in your example `app.yml`, acts as the "namespace"
for calling the configuration value.

The `get()` method also provides the ability to pass default values if a variable is not present in your
configuration files. To do this, simply pass a second parameter to the `get()` method:

```php
$config->get('app.version', '1.0.0'); // 1.0.0
```

If a configuration value could not be found, the `get()` method automatically returns `null`.

---

## Determine whether a configuration value exists

If you want to know if a configuration value exists, just use the `has()` method:

```php
$config->has('app.name'); // true
```

---

## Get all configuration values

You might also want to get all available configuration values as an array. This is possible with the
`toArray()` method:

```php
$config->toArray(); // ['app' => 'name']
```

---

## Configuration value replacers

You might want to store critical data, such as your database credentials, in your configuration files.
It would be fatal to write this directly into the configuration file and then commit it to your Git instance, for
example.
So you put your critical credentials in a `.env` file and exclude them from your git commits. This repository now
offers you a way to access stored `env` values within your configuration file.

Let's imagine a scenario where your database credentials are stored in a `.env` file somewhere in your project:

```bash
DB_USER=user
DB_PASS=secret
```

Now you want to access the `env` values in your `database.yml` inside your `/config/path` directory. This works as
follows:

```yaml
# /config/path/database.yml
mysql:
    user: '%env:DB_USER%'
    password: '%env:DB_PASS%'
```

With the indicators `%` and the keyword `env:` the configuration loader knows to use a replacer. In this case the
`Zaphyr\Config\Replacers\EnvReplacer`. The corresponding replacer now takes over the resolution of your `env` value for
you.

### Create a custom replacer

You may also want to use your own replacers in your configuration files. This is also possible and is described below.

First you should create your own replacer class, which implements the `Zaphyr\Config\Contracts\ReplacerInterface` and
includes the `replace` method:

```php
class MySuperCustomReplacer implements Zaphyr\Config\Contracts\ReplacerInterface
{
    /**
     * {@inheritdoc}
     */
    public function replace(string $value): string
    {
        // Your custom replacer logic
    }
}
```

Now you need to pass your previously created replacer in the constructor of your `Zaphyr\Config\Config` instance:

```php
$replacers = ['custom' => MySuperCustomReplacer::class];

$config = new Zaphyr\Config\Config(['/config/paths'], null, $replacers);
```

Alternatively, you can also pass your own replacer to the `addReplacer()` method:

```php
$config->addReplacer('custom', MySuperCustomReplacer::class);
```

You can now use your custom replacer in your configuration files:

```yaml
config: '%custom:value%'
```

> [!NOTE]
> New replacer instances must always be added before the first use of the `load()` method.
> Otherwise, the `load()` method throws an error because it does not yet know the new replacer!

---

## Custom configuration readers

Even if this repository is already equipped with a lot of configurations file extension readers, it is still possible
to create your custom configuration readers. To do this, proceed as follows.

First you should create your own reader instance, which implements the `Zaphyr\Config\Contracts\ReaderInterface` and
contains a `read` method:

```php
class MySuperCustomReader implements Zaphyr\Config\Contracts\ReaderInterface
{
    /**
     * {@inheritdoc}
     */
    public function read(): array
    {
        // Your custom reader logic
    }
}
```

Secondly, you register your custom reader in the config class:

```php
$readers = ['custom' => MySuperCustomReader::class];

$config = new Zaphyr\Config\Config(['/config/paths'], $readers);
```

Alternatively, you can also use the `addReader()` method:

```php
$config->addReader('custom', MySuperCustomReader::class);
```

Now you can use configuration files with your own file extension.

> [!NOTE]
> New reader instances must always be added before the first use of the `load()` method.
> Otherwise, the `load()` method throws an error because it does not yet know the new reader!

---

## Dependency Injection

<span class="badge rounded-pill text-bg-primary">Available since v2.2.0</span>

The config repository supports dependency injection, by using any [PSR-11](https://www.php-fig.org/psr/psr-11/)
compatible DI container. The PSR-11 container instance can be set using the `setContainer` method:

```php
$container = new Container();
$container->set(ServiceInterface::class, new Service());

$config->setContainer($container);
```

Now your constructor parameters for custom [readers](#content-custom-configuration-readers) or
[replacers](#content-create-a-custom-replacer) are automatically resolved by the DI container:

```php
class MySuperCustomReplacer implements Zaphyr\Config\Contracts\ReplacerInterface
{
    /**
     * @var ServiceInterface
     */
     public function __construct(ServiceInterface $service)
     {
     }

    /**
     * {@inheritdoc}
     */
    public function replace(string $value): string
    {
        // Your custom replacer logic
    }
}
```

> [!NOTE]
> This repository does **NOT** come with a PSR-11 implementation out of the box. For a PSR-11 implementation, check out
> the [Container](/docs/1.x/repositories/container) repository.
