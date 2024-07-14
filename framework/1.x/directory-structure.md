# Directory structure

The default application directory structure is based on the [Standard PHP package skeleton](https://github.com/php-pds/skeleton),
designed to provide a clean and organized layout for your application. However, you are free to organize your
application however you see fit.

## The `bin/` directory

The `bin` directory contains all your executable files, such as console commands and other scripts that are not part of
the web-accessible files.
<!-- @todo add link to console commands documentation -->

## The `config/` directory

The `config` directory contains all of your application's settings, such as service providers, routing, sessions, and so
on. You can read more about the configuration settings in the [configuration documentation](/docs/framework/latest/configuration).
By default, we use a `YAML` configuration file for the application settings. Because the framework utilizes the
[Configuration repository](/docs/repositories/latest/config), you are free to use different configuration formats, such
as `PHP`, `INI`, `JSON`, `XML`, `YAML`, and `NEON` files.

## The `public/` directory

The `public` directory is the web server's document root. This is where the front controller, `index.php`, is located.
You should also place your web-accessible files, such as images, CSS, and JavaScript files, in this directory.

## The `resources/` directory

The `resources` directory contains all of your application's resources, including views and uncompiled assets.

## The `src/` directory

This is where the magic happens! The `src` directory contains all the core code for your application. This directory is
namespaced under `App` and is [autoloaded by Composer](https://getcomposer.org/doc/01-basic-usage.md#autoloading).

### The `Controllers/` directory

The `Controllers` directory contains all of your application's controllers. Controllers are responsible for handling
incoming requests, processing the input, and returning a response.
<!-- @todo add link to controllers documentation -->

### The `Exceptions/` directory

The `Exceptions` directory contains all of your application's custom exceptions. Custom exceptions can be used to handle
specific errors within your application.
<!-- @todo add link to exceptions documentation -->

### The `bootstrap.php` file

The `bootstrap.php` file serves as the entry point for your application. It is responsible for bootstrapping the
application and setting up the environment.

### The `functions.php` file

The `functions.php` file contains all of your application's helper functions. You can define custom functions here to
assist with common tasks. This file is automatically loaded by Composer.

## The `storage/` directory

The `storage` directory contains all of your application's `cache`, `log`, and `session` files. This directory is
typically writable by the web server, allowing you to store temporary files here.

## The `tests/` directory

The `tests` directory contains all of your application's tests. You can use this directory to write unit tests for your
application. By default, we use the [PHPUnit](https://phpunit.de/) testing framework, but you are free to use any
testing framework you prefer.
<!-- @todo add link to testing documentation -->

## The `vendor/` directory

The `vendor` directory contains all of your application's dependencies. This directory is managed by
[Composer](https://getcomposer.org/) and should not be modified directly.

## Change directory structure

The default directory structure is just a suggestion, and you are not required to stick to it. You can change the
directory structure as you see fit.

Let's take a look at how you can change the default directory structure. A good starting point for modifying the
directory structure is the `bootstrap.php` file located in your `src` directory. This file is responsible for
bootstrapping the application and setting up the environment.

For example, let's say you want to change the `config` directory to the `settings` directory. You can do this by
modifying the `bootstrap.php` file as follows:

```php
$application = new Zaphyr\Framework\Application(dirname(__DIR__));
$application->setConfigPath('settings');
```

The `Zaphyr\Framework\Application` class is responsible for bootstrapping the application. This class offers the
following methods to change the default directory structure:

- **setAppPath** - Sets the path to the `src` directory.
- **setConfigPath** - Sets the path to the `config` directory.
- **setPublicPath** - Sets the path to the `public` directory.
- **setResourcesPath** - Sets the path to the `resources` directory.
- **setStoragePath** - Sets the path to the `storage` directory.

Each of the above setter methods also has a corresponding getter method to retrieve the current path:

```php
$application->getAppPath();
$application->getConfigPath();
$application->getPublicPath();
$application->getResourcesPath();
$application->getStoragePath();
```

> [!NOTE]
> You should always use these getter methods inside your application's code to retrieve the current path, as this
> ensures that the path is always correct. Please do not hardcode paths inside your application code, as this can lead
> to problems when changing the directory structure.

### Change the `src` directory

If you change the `src` directory of your application, keep in mind that you also need to update the `composer.json`
file to reflect the new directory structure. You can do this by modifying the `autoload` and `scripts` sections of the
`composer.json` file:

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "path/to/new/src"
        },
        "files": [
            "path/to/new/src/functions.php"
        ]
    },
    "scripts": {
        "cs": "vendor/bin/phpcs --standard=PSR12 -n path/to/new/src",
        "cbf": "vendor/bin/phpcbf --standard=PSR12 -n path/to/new/src"
    }
}
```
