# Configuration

All application configurations are stored in the [
`config` directory](/docs/framework/latest/directory-structure#the-config-directory).
`YAML` is used as the default configuration file format, but you can also use `PHP`, `INI`, `JSON`, `XML`, and `NEON`
files. This flexibility is possible because the ZAPHYR framework uses the
[Config repository](/docs/repositories/latest/config) to load the configuration files.

## Environment variables

While developing applications, it is common to have different configurations for various environments. For instance,
you might use distinct database credentials for your development and production environments. To manage this, ZAPHYR
supports environment variables, using the [PHP dotenv](https://github.com/vlucas/phpdotenv) package.

Every new ZAPHYR skeleton application includes a `.env.dist` file in the root directory. During the [installation
via Composer](/docs/framework/latest/installation), this file is copied to create a `.env` file.

The `.env` file contains several essential environment variables needed to run the application. You are encouraged to
modify this file to meet your specific requirements.

> [!IMPORTANT]
> You should not commit the `.env` file to your repository, as it may contain sensitive information, and each developer
> might require different environment variables. Instead, commit the `.env.dist` file, which should be devoid of any
> sensitive information. Subsequently, instruct other developers to copy the `.env.dist` file to `.env` and populate it
> with their specific environment values.

### Application environment

ZAPHYR supports environment-based configuration, allowing you to tailor behavior for `development`, `testing`,
`production` or any custom environment.

The current environment is determined by the `APP_ENV` variable defined in your application's `.env` file:

```bash
APP_ENV=development
```

ZAPHYR supports the following predefined environments:

- `development` - for development environments
- `testing` - for testing environments
- `production` - for production environments

You can check the current environment programmatically:

```php
if (app()->isDevelopmentEnvironment()) {
    // Run dev-specific code, such as debug panels or verbose logs
}

if (app()->isTestingEnvironment()) {
    // Load mocks or disable third-party integrations
}

if (app()->isProductionEnvironment()) {
    // Enable performance optimizations or production services
}
```

#### Custom environments

ZAPHYR is flexible and allows you to define your own custom environments. For instance, you might use a
`staging` environment for pre-release testing.

Update the `.env` file accordingly:

```bash
APP_ENV=staging
```

And detect it in code:

```php
if (app()->isEnvironment('staging')) {
    // Staging-specific configurations
}
```

You can use any custom string as a valid environment name, just ensure your logic accounts for it where necessary.

#### Show environment in the terminal

To display the currently active environment in the terminal, run:

```bash
php bin/zaphyr app:environment
```

This is helpful for verifying configuration, especially in CI/CD pipelines or debugging deployments.

### Application key

The application key (`APP_KEY`) is a critical security part of your ZAPHYR application. It is used to:

- Encrypt and decrypt sensitive data (e.g. cookies, user sessions)
- Ensure data integrity and confidentiality across application boundaries

A unique, randomly generated key should be assigned to every application and **must not be shared or reused** across
environments.

#### Location of the application key

The key is stored in your `.env` file:

```bash
APP_KEY=base64:randomlyGeneratedKeyHere
```

This key is automatically generated during project creation. However, you may regenerate it manually if needed.

#### Regenerate the application key

To generate a new secure application key, run:

```bash
php bin/zaphyr app:key
```

This will replace the existing `APP_KEY` in your `.env` file with a newly generated random value.

> [!WARNING]
> Regenerating the application key will **invalidate all existing encrypted data**, including,
> user sessions, encrypted cookies and any custom encrypted values. This may force users to log
> in again or cause encrypted payloads to become inaccessible. Only perform this in development or
> during controlled maintenance windows.

#### Force the key regeneration

By default, ZAPHYR will prompt for confirmation before overwriting the key in a production environment. To skip this
confirmation and force the operation, use the `--force` flag:

```bash
php bin/zaphyr app:key --force
```

#### Show the current application key

To view the currently active application key without changing it, use the `--show` flag:

```bash
php bin/zaphyr app:key --show
```

This can be helpful for debugging or verifying your application setup across multiple environments.

## Configuration variables

The `config` directory contains all the configuration files for the application. Each file is named according to the
configuration section it represents. Configuration files are loaded using the
[Config repository](/docs/repositories/latest/config), allowing you to access settings from anywhere in your
application.

### Configuration file replacers

To set dynamic configuration values, you can use the `%env%` replacer in your configuration files. This allows you to
reference environment variables directly. For example, to set the application name from the `APP_NAME` environment
variable, use the following configuration:

```yaml
name: '%env:APP_NAME%'
```

In addition to `%env%`, the framework also provides a `%path%` replacer, which dynamically references paths within the
application. It is recommended to use this replacer when defining paths in your configuration files to ensure
[paths remain correct](/docs/framework/latest/directory-structure#change-directory-structure) across different
environments.

```yaml
templates: '%path:resources%/views'
```

### Retrieving configuration values in your application

You can access configuration values from anywhere in your application using the
[Config](/docs/repositories/latest/config) repository. It provides a simple and convenient way to retrieve settings:

```php
$container->get(Zaphyr\Config\Contracts\ConfigInterface::class)->get('app.name');
```

## Cache configuration

Caching your application's configuration files can significantly improve performance by reducing the number of files
loaded on each request, resulting in faster response times. To cache your configuration files, run the following
command:

```bash
php bin/zaphyr config:cache
```

If you need to clear the configuration cache, use:

```bash
php bin/zaphyr config:clear
```

> [!IMPORTANT]
> If the configuration is cached, the `.env` file will not be loaded, and the configuration values will
> be taken from the cached configuration files. Therefore, it is crucial to clear the cache whenever you make changes
> to your configuration files or the `.env` file to ensure that the latest settings are applied.

## List configuration via CLI

You can view all the configuration values currently loaded in your application by running the following command in the
terminal:

```bash
php bin/zaphyr config:list
```

This will output a structured list of configuration keys and their corresponding values, making it easier to debug or
inspect the active configuration at runtime.
