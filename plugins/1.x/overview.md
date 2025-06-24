# Plugins

ZAPHYR is a modular and extensible PHP framework designed with flexibility and developer productivity in mind.
Its architecture is built around a powerful plugin system that allows you to enhance and customize the framework's core
functionality with ease.

Plugins in ZAPHYR can seamlessly integrate additional features such as services, CLI commands, controllers, middleware,
and event listeners. This enables you to tailor the behavior of your application without modifying the core framework.

A growing ecosystem of official plugins is already available, offering ready-to-use solutions. However, ZAPHYR also
empowers developers to build their own custom plugins to address specific project requirements, promoting clean
separation of concerns and maintainable codebases.

Whether you're building small microservices or large-scale enterprise applications, ZAPHYR's plugin-based architecture
ensures that your code remains modular, reusable, and easy to extend.

## Plugin Installer

ZAPHYR includes a powerful plugin installer system that simplifies the process of integrating and managing plugins
within your application. This system is enabled through the `allow-plugins` section in your app’s `composer.json` file:

```json
"allow-plugins": {
    "zaphyr-org/plugin-installer": true
}
```

This configuration enables Composer to execute the ZAPHYR plugin installer during dependency installation or updates.

The plugin installer automates the integration process for any plugins defined in your composer.json. When you run
`composer install` or `composer update`, it will:

- Detect any registered ZAPHYR plugins
- Copy necessary configuration files
- Set up environment variables
- Autoload and register: service providers, console commands, controllers, middleware, and event listeners

This ensures a seamless and consistent setup of third-party plugins without requiring manual configuration.

For plugin authors, this system also allows defining integration logic directly in the plugin's `composer.json`,
enabling ZAPHYR to manage everything automatically.

## Create Custom Plugins

Creating your own plugin for ZAPHYR allows you to encapsulate and distribute reusable functionality across projects or
teams. The process is straightforward and follows a structured approach to ensure seamless integration with the
framework.

To build a custom plugin, you typically follow two main steps:

- Set up the plugin package
- Define plugin metadata and integration logic

Let's walk through the process of creating a custom plugin step by step.

### Plugin composer.json

To create a custom plugin, you must define its metadata and behavior in a `composer.json` file. This file is essential
for ZAPHYR to recognize, install, and manage your plugin correctly.

The key elements of the composer.json file include:

- `type`: Must be set to `zaphyr-plugin` so ZAPHYR treats the package as a plugin.
- `extra.zaphyr.plugin-classes`: Specifies the classes the plugin installer should register, and the environments in
  which they should be active (e.g., `all`, `development`, `production`).

```json
{
    "name": "acme/my-plugin",
    "type": "zaphyr-plugin",
    "autoload": {
        "psr-4": {
            "Acme\\MyPlugin\\": "src/"
        }
    },
    "extra": {
        "zaphyr": {
            "plugin-classes": {
                "Acme\\MyPlugin\\MyPlugin": [
                    "all"
                ]
            }
        }
    }
}
```

#### Plugin Environments

In the `extra` section of your plugin's `composer.json, you can define which environments the plugin class should be
loaded in. This gives you fine-grained control over when your plugin is active, depending on the application's runtime
environment.

You can specify one or more of the following values:

| Field         | Description                                                                  |
|---------------|------------------------------------------------------------------------------|
| `all`         | Load the plugin class in all environments.                                   |
| `development` | Load only in the development environment (e.g., during local development).   |
| `testing`     | Load only in the testing environment (e.g., for automated test runs).        |
| `production`  | Load only in the production environment (e.g., live/production deployments). |

This mechanism ensures that plugins behave appropriately in different stages of the application lifecycle.

#### Copy Files

You can specify files or directories that should be automatically copied to specific locations within your application
when the plugin is installed. This is useful for distributing configuration files, assets, or other resources required
for your plugin to function correctly.

To define copy operations, add a `copy` section under the `extra.zaphyr` key in your plugin’s `composer.json` file:

<pre><code class="language-diff-json">
{
    "name": "acme/my-plugin",
    "type": "zaphyr-plugin",
    "autoload": {
        "psr-4": {
            "Acme\\MyPlugin\\": "src/"
        }
    },
    "extra": {
        "zaphyr": {
            "plugin-classes": {
                "Acme\\MyPlugin\\MyPlugin": [
                    "all"
                ]
            },
+            "copy": {
+                "config/plugins/my-plugin.yaml": "%config%/plugins/my-plugin.yaml",
+            }
        }
    }
}
</code></pre>

In the copy section:

- The **left-hand side** defines the source file or directory within your plugin.
- The **right-hand side** defines the destination path in the host application.

ZAPHYR supports a set of **placeholders** in destination paths that will automatically resolve to the appropriate
application directories:

| Path           | Description                                                                          |
|----------------|--------------------------------------------------------------------------------------|
| `%root%/`      | Root directory of the application.                                                   |
| `%app%/`       | Application directory (typically where app logic resides).                           |
| `%bin%/`       | Directory for CLI scripts (e.g., `bin/zaphyr`).                                      |
| `%config%/`    | Configuration directory, typically where `.yaml` or `.php` config files are placed.  |
| `%public%/`    | Public directory, used for web-accessible resources.                                 |
| `%resources%/` | Resource directory, ideal for views, language files, or static assets.               |
| `%storage%/`   | Storage directory, used for cache, logs, or other writable data.                     |

> [!IMPORTANT]
> Always use the ZAPHYR-provided placeholders instead of hardcoded paths. This ensures your plugin remains compatible
> with projects that use a
> [custom directory structure](/docs/framework/latest/directory-structure#change-directory-structure).

#### Add Environment Variables

If your plugin relies on specific environment variables, you can define them under the `env` section in your plugin’s
`composer.json` file. This allows you to set default values that will be added to the application’s `.env` file during
installation.

<pre><code class="language-diff-json">
{
    "name": "acme/my-plugin",
    "type": "zaphyr-plugin",
    "autoload": {
        "psr-4": {
            "Acme\\MyPlugin\\": "src/"
        }
    },
    "extra": {
        "zaphyr": {
            "plugin-classes": {
                "Acme\\MyPlugin\\MyPlugin": [
                    "all"
                ]
            },
            "copy": {
                "config/": "%config%"
            },
+            "env": {
+                "ACME_MY_PLUGIN_SETTING": "default_value"
+            }
        }
    }
}
</code></pre>

These environment variables will be appended to the `.env` file of the application in the following format:

```bash
### start-plugin-config:acme/my-plugin ###
ACME_MY_PLUGIN_SETTING=default_value
### end-plugin-config:acme/my-plugin ###
```

The block is clearly marked using `start-plugin-config` and `end-plugin-config` comments. These markers are essential,
they allow the plugin installer to manage the configuration block reliably during updates or removals. You should
not modify or remove these comments manually.

#### Naming Environment Variables

When defining environment variables for your plugin, it's crucial to follow a consistent naming convention to ensure
to avoid conflicts with other plugins or core application variables; environment variable names should be **prefixed**
with the plugin’s namespace, typically the uppercased vendor and plugin name. For example, for a plugin named
`acme/my-plugin`, use variables like:

```bash
ACME_MY_PLUGIN_*
```

This ensures your variables remain uniquely scoped to your plugin and follow best practices for modular configuration.

### Plugin Class

The core of any ZAPHYR plugin is its plugin class. This class acts as the **entry point** for your plugin and defines
the components it contributes to the application, such as service providers, console commands, controllers, middleware,
and event listeners.

To create one, extend the `Zaphyr\Framework\Plugins\AbstractPlugin` class. Your plugin class should implement one or
more static methods to return arrays of the components it registers.

Below is a sample plugin class that includes all available component types:

```php
<?php

declare(strict_types=1);

namespace Acme\MyPlugin;

use Acme\MyPlugin\Commands\MyPluginCommand;
use Acme\MyPlugin\Controllers\MyPluginController;
use Acme\MyPlugin\Listener\MyPluginListener;
use Acme\MyPlugin\Middleware\MyPluginMiddleware;
use Acme\MyPlugin\Providers\MyPluginServiceProvider;
use Zaphyr\Framework\Events\Http\RequestStartingEvent;
use Zaphyr\Framework\Plugins\AbstractPlugin;

class MyPlugin extends AbstractPlugin
{
    /**
     * Register service providers used by this plugin.
     * 
     * {@inheritdoc}
     */
    public static function providers(): array
    { 
        return [
            MyPluginServiceProvider::class,
        ];
    }

    /**
     * Register CLI commands provided by this plugin.
     *
     * {@inheritdoc}
     */
    public static function commands(): array
    {
        return [
            MyPluginCommand::class,
        ];
    }

    /**
     * Register controllers used by this plugin.
     * 
     * {@inheritdoc}
     */
    public static function controllers(): array
    {
        return [
            MyPluginController::class,
        ];
    }

    /**
     * Register HTTP global middleware provided by this plugin.
     *
     * {@inheritdoc}
     */
    public static function middleware(): array
    {
        return [
            MyPluginMiddleware::class,
        ];
    }

    /**
     * Register event listeners for the application.
     * 
     * {@inheritdoc}
     */
    public static function events(): array
    {
        return [
            RequestStartingEvent::class => [
                MyPluginListener::class,
                
                // Listener with optional priority
                [
                 'listener' => MyPluginListener::class,
                 'priority' => 100,
                ] 
            ],
        ];
    }
}
```

> [!NOTE]
> You only need to define the methods relevant to your plugin. If your plugin does not provide, for example, middleware
> or commands, you can omit those respective methods entirely.

This modular approach keeps your plugin class clean and focused, allowing it to scale gracefully as your plugin grows.

### Register the Plugin

After you've created your plugin class and defined its metadata in the plugin’s `composer.json` file, the final step is
to register it in your application so that it can be installed and used.

#### Add the Plugin to Your Application

Include your plugin in the `require` section of your application's root `composer.json` file:

```json
{
    "require": {
        "acme/my-plugin": "^1.0"
    }
}
```

> [!NOTE]
> If you're developing the plugin locally, make sure you also define a local path repository using the `repositories`
> key so Composer knows where to find it.

After adding the plugin to your `composer.json`, you can run the following command to install it:

#### Install the Plugin

Once the dependency is added, install it via Composer:

```bash
composer install
```

This command triggers ZAPHYR’s plugin installer, which will:

- Parse your plugin’s `composer.json`
- Copy files and configuration assets (as defined under `extra.zaphyr.copy`)
- Set environment variables (defined in `extra.zaphyr.env`)
- Automatically register your plugin’s classes, such as service providers, commands, middleware, controllers, and event
  listeners

After installation is complete, your plugin is fully integrated into the application and ready to use. All declared
components will be autoloaded and registered by the framework automatically, so you can start leveraging your plugin’s
functionality right away.
