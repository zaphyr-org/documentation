# Installation

This guide will walk you through the process of creating a new application.

## System requirements

Before you start, ensure the following are installed on your system:

* [PHP](https://www.php.net) 8.1 or higher, along with the following PHP extensions:
    * [Ctype](https://www.php.net/book.ctype)
    * [DOM](https://www.php.net/book.dom)
    * [Fileinfo](https://www.php.net/book.fileinfo)
    * [JSON](https://www.php.net/book.json)
    * [libxml](https://www.php.net/book.libxml)
    * [Multibyte String](https://www.php.net/book.mbstring)
    * [OpenSSL](https://www.php.net/book.openssl)
    * [SimpleXML](https://www.php.net/book.simplexml)
* [Composer](https://getcomposer.org/)
* A [web server](https://www.php.net/manual/features.commandline.webserver.php) with URL rewriting capability

> [!TIP]
> ZAPHYR requires PHP 8.1 as the minimum version. However, we recommend using the
> [latest stable version of PHP](https://www.php.net/supported-versions.php) for the best experience.

## Create application

To create a new application, open a terminal (either Command Prompt or Terminal app, depending on your OS) and run the
following Composer command from the directory where you want to install your application:

```console
composer create-project zaphyr-org/app my-app
```

This command installs a fresh [skeleton application](https://github.com/zaphyr-org/app) in a directory named `my-app`.
Feel free to replace `my-app` with your desired application name.

## Setup and run application

After creating the application, navigate to your application directory (`my-app`):

```console
cd my-app
```

Set the `APP_URL` environment variable in the `.env` file at the root of your project directory:

```bash
APP_URL=http://localhost:8000
```

The `.env` file stores important configuration settings, such as environment variables. You can read more about it in
the [configuration](/docs/framework/latest/configuration) section.

To start the [built-in PHP web server](https://www.php.net/manual/features.commandline.webserver.php) from the root of
your project, run the following command:

```console
php -S localhost:8000 -t public
```

The application is now running on `http://localhost:8000`. You can access it in your browser by visiting the URL.

> [!WARNING]
> The PHP built-in web server is intended for development purposes only and is not suitable for production use.

> [!TIP]
> For local development, we recommend using [ddev](https://ddev.readthedocs.io/en/stable/), a Docker-based environment
> that simplifies setup and provides consistency across different projects.
