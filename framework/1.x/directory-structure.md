# Directory structure

The default application directory structure is based on
the [Standard PHP package skeleton](https://github.com/php-pds/skeleton),
designed to provide a clean and organized layout for your application. However, you are free to organize your
application however you see fit.

## The app directory

This is where the magic happens! The `app` directory contains all the core code for your application. This directory is
namespaced under `App` and is [autoloaded by Composer](https://getcomposer.org/doc/01-basic-usage.md#autoloading).

### The Controllers directory

The `Controllers` directory contains all of your application's controllers. Controllers are responsible for handling
incoming requests, processing the input, and returning a response.
<!-- @todo add link to controllers documentation -->

### The Exceptions directory

The `Exceptions` directory contains all of your application's custom exceptions. Custom exceptions can be used to handle
specific errors within your application.
<!-- @todo add link to exceptions documentation -->

### The bootstrap.php file

The `bootstrap.php` file serves as the entry point for your application. It is responsible for bootstrapping the
application and setting up the environment.

### The functions.php file

The `functions.php` file contains all of your application's helper functions. You can define custom functions here to
assist with common tasks. This file is automatically loaded by Composer.

## The bin directory

The `bin` directory contains all your executable files, such as console commands and other scripts that are not part of
the web-accessible files.
<!-- @todo add link to console commands documentation -->

## The config directory

The `config` directory contains all of your application's settings, such as service providers, routing, sessions, and so
on. You can read more about the configuration settings in
the [configuration documentation](/docs/framework/latest/configuration).
By default, we use a `YAML` configuration file for the application settings. Because the framework utilizes the
[Configuration repository](/docs/repositories/latest/config), you are free to use different configuration formats, such
as `PHP`, `INI`, `JSON`, `XML`, `YAML`, and `NEON` files.

## The public directory

The `public` directory is the web server's document root. This is where the front controller, `index.php`, is located.
You should also place your web-accessible files, such as images, CSS, and JavaScript files, in this directory.

## The resources directory

The `resources` directory contains all of your application's resources, including views and uncompiled assets.

## The storage directory

The `storage` directory contains all of your application's `cache`, `log`, and `session` files. This directory is
typically writable by the web server, allowing you to store temporary files here.

## The tests directory

The `tests` directory contains all of your application's tests. You can use this directory to write unit tests for your
application. By default, we use the [PHPUnit](https://phpunit.de/) testing framework, but you are free to use any
testing framework you prefer.
<!-- @todo add link to testing documentation -->

## The vendor directory

The `vendor` directory contains all of your application's dependencies. This directory is managed by
[Composer](https://getcomposer.org/) and should not be modified directly.

## Retrieving directory paths in your application

To get the paths of various directories in your application, use the following getter methods:

```php
$application->getRootPath(); // /path/to/root
$application->getAppPath(); // /path/to/root/app
$application->getBinPath(); // /path/to/root/bin
$application->getConfigPath(); // /path/to/root/config
$application->getPublicPath(); // /path/to/root/public
$application->getResourcesPath(); // /path/to/root/resources
$application->getStoragePath(); // /path/to/root/storage
```

### Retrieving specific file or directory paths

You can also pass parameters to these getter methods to obtain the path to a specific file or subdirectory:

```php
$application->getResourcesPath('views'); // /path/to/root/resources/views
$application->getResourcesPath('views/template.html'); // /path/to/root/resources/views/template.html
```

> [!NOTE]
> Always use these getter methods to retrieve paths within your application code. This ensures the paths remain
> accurate and adaptable to future directory structure changes. Avoid hardcoding paths, as this can lead to issues when
> modifying the application's directory layout.

## Change directory structure

The default directory structure is merely a suggestion, and you are free to customize it to suit your needs.

#### Modifying the directory structure in `composer.json`

To modify the directory structure, update the `composer.json` file. You can specify custom paths for different
directories within the `extra` section of your application's `composer.json` file.

```json
{
    "extra": {
        "zaphyr": {
            "paths": {
                "app": "path/to/app",
                "bin": "path/to/bin",
                "config": "path/to/config",
                "public": "path/to/public",
                "resources": "path/to/resources",
                "storage": "path/to/storage"
            }
        }
    }
}
```

#### Passing paths to the application constructor

Another option is to pass the paths to the constructor of the `Zaphyr\Framework\Application` class:

```php
$paths = [
    'app' => 'path/to/app',
    'bin' => 'path/to/bin',
    'config' => 'path/to/config',
    'public' => 'path/to/public',
    'resources' => 'path/to/resources',
    'storage' => 'path/to/storage'
];

$application = new Zaphyr\Framework\Application($paths);
```

> [!IMPORTANT]
> However, this approach should only be used in unit test environments, as
> the [plugin installer](/docs/plugins/latest/overview#plugin-installer)
> only reads the application paths from the `composer.json` file.

> [!NOTE]
> Paths passed to the constructor override those defined in the composer.json file.

### Change the root directory

The root directory serves as the base directory of your application. By default, it is set to the location of the
`composer.json` file. For example, if `composer.json` is located in `/var/www/html/app`, the root directory will
be `/var/www/html/app`.

#### Modifying the root directory in `composer.json`

You can change the root directory by modifying the `composer.json file`. To do this, define the root directory in the
`extra` section of your application's `composer.json` file:

<pre><code class="language-diff-json">
{
    "extra": {
        "zaphyr": {
            "paths": {
+               "root": "/path/to/new/root",
                "app": "path/to/app",
                "bin": "path/to/bin",
                "config": "path/to/config",
                "public": "path/to/public",
                "resources": "path/to/resources",
                "storage": "path/to/storage"
            }
        }
    }
}
</code></pre>

#### Setting the root directory via `$_ENV`

You can also define the root directory using the $_ENV superglobal.

```php
$_ENV['ROOT_PATH'] = '/path/to/new/root';
```

> [!NOTE]
> If the root directory is set using `$_ENV`, it wil override the value specified in the `composer.json`.

#### Passing the root directory to the application constructor

Alternatively, you can pass the root directory to the constructor of the `Zaphyr\Framework\Application` constructor:

```php
$paths = [
    'root' => '/path/to/new/root'
];

$application = new Zaphyr\Framework\Application($paths);
```

> [!NOTE]
> If a root directory is provided to the constructor, it takes precedence over values defined in both `composer.json`
> and `$_ENV`.

### Change the app directory

If you change the `app` directory of your application, keep in mind that you also need to update the `composer.json`
file to reflect the new directory structure. You can do this by modifying the `autoload` and `scripts` sections of the
`composer.json` file:

<pre><code class="language-diff-json">
{
    "autoload": {
        "psr-4": {
-           "App\\": "app"
+           "App\\": "path/to/new/app"
        },
        "files": [
-            "app/functions.php"
+            "path/to/new/app/functions.php"
        ]
    },
    "scripts": {
-       "cs": "vendor/bin/phpcs --standard=PSR12 -n app",
+       "cs": "vendor/bin/phpcs --standard=PSR12 -n path/to/new/app",
-       "cbf": "vendor/bin/phpcbf --standard=PSR12 -n app"        
+       "cbf": "vendor/bin/phpcbf --standard=PSR12 -n path/to/new/app"
    }
}
</code></pre>
