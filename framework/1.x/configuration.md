# Configuration

All application configurations are stored in the [`config` directory](/docs/framework/latest/directory-structure#the-config-directory).
`YAML` is used as the default configuration file format, but you can also use `PHP`, `INI`, `JSON`, `XML`, and `NEON`
files. This flexibility is possible because the ZAPHYR framework utilizes the [Config repository](/docs/repositories/latest/config) to load the
configuration files.

## Environment variables

While developing applications, it is common to have different configurations for various environments. For instance,
you might use distinct database credentials for your development and production environments. To manage this, ZAPHYR
supports environment variables, utilizing the [PHP dotenv](https://github.com/vlucas/phpdotenv) package.

Every new ZAPHYR skeleton application includes a `.env.dist` file in the root directory. During the [installation
via Composer](/docs/framework/latest/installation), this file is copied to create a `.env` file.

The `.env` file contains several essential environment variables needed to run the application. You are encouraged to
modify this file to meet your specific requirements.

> [!IMPORTANT]
> You should not commit the `.env` file to your repository, as it may contain sensitive information, and each developer
> might require different environment variables. Instead, commit the `.env.dist` file, which should be devoid of any
> sensitive information. Subsequently, instruct other developers to copy the `.env.dist` file to `.env` and populate it
> with their specific environment values.

Let's take a closer look at the `.env` file and its contents. Right after installation, the file typically looks
like this:

```bash
APP_NAME=ZAPHYR
APP_URL=
APP_ENV=development
APP_DEBUG=true
APP_KEY=
APP_CIPHER=AES-256-CBC
```

#### The `APP_NAME` and `APP_URL` variables

These variables are used to set the application's name and base URL. While not directly used by the framework, they can
be useful for displaying the application name in the user interface or for generating links within the application.

#### The `APP_ENV` variable

This variable sets the application's environment. It determines the environment in which the application is running,
such as `development`, `testing`, or `production`.

#### The `APP_DEBUG` variable

This variable enables or disables the application's debug mode. When set to `true`, the application displays detailed
error messages, which is useful during development. However, it is recommended to set this value to `false` in
production to avoid exposing sensitive information.

#### The `APP_KEY` variable

This variable sets the application's encryption key, which is crucial for encrypting and decrypting data within the
application. During the installation process, a random key is generated and stored in the `.env` file.

> [!TIP]
> You can generate a new key by running the `php bin/zaphyr app:key` command. This will create a new key and update it
> directly in the `.env` file.
<!-- @todo: add link to the key:generate command -->

#### The `APP_CIPHER` variable

This variable sets the application's encryption cipher. By default, ZAPHYR uses the `AES-256-CBC` cipher, but you can
also change it to `AES-128-CBC`.

## Configuration variables

The `config` directory contains all the configuration files for the application. Each file is named according to the
configuration section it represents. Configuration files are loaded using the [Config repository](/docs/repositories/latest/config),
allowing you to access settings from anywhere in your application.

Every ZAPHYR application includes a default configuration file named `app.yaml`, which provides a well-structured
starting point. While the default settings are optimized for general use, you can modify them to better fit your specific
needs.

### Configuration file replacers

To set dynamic configuration values, you can use the `%env%` replacer in your configuration files. This allows you to
reference environment variables directly. For example, to set the application name from the `APP_NAME` environment
variable, use the following configuration:

```yaml
name: '%env:APP_NAME%'
```

In addition to `%env%`, the framework also provides a `%path%` replacer, which dynamically references paths within the
application. It is recommended to use this replacer when defining paths in your configuration  files to ensure
[paths remain correct](/docs/framework/latest/directory-structure#change-directory-structure) across different environments.

```yaml
templates: '%path:resources%/views'
```

### Retrieving configuration values in your application

You can access configuration values from anywhere in your application using the [Config](/docs/repositories/latest/config)
repository. It provides a simple and convenient way to retrieve settings:

```php
$container->get(Zaphyr\Config\Contracts\ConfigInterface::class)->get('app.name');
```

### Cache configuration

Caching your application's configuration files can significantly improve performance by reducing the number of files
loaded on each request, resulting in faster response times. To cache your configuration files, run the following command:

```console
php bin/zaphyr config:cache
```
<!-- @todo: add link to the config:cache command -->

If you need to clear the configuration cache, use:

```console
php bin/zaphyr config:clear
```
<!-- @todo: add link to the config:clear command -->
