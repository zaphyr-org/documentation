# Configuration

All application configurations are stored in the `config` directory. `YAML` is used as the default configuration file
format, but you can also use `PHP`, `INI`, `JSON`, `XML`, and `NEON` files. This flexibility is possible because the
ZAPHYR framework utilizes the [Config repository](/docs/repositories/latest/config) to load the configuration files.

## Environment Variables

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

***To be continued...***
