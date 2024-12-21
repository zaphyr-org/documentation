# Logger

> [!WARNING]
> You're browsing the documentation for an old version running php 7.x.
> Consider upgrading to [v2.x](/docs/repositories/2.x/logger).

You want to know what's happening "under the hood" of your application? ZAPHYR provides a robust
[PSR-3](https://www.php-fig.org/psr/psr-3/) logging service.

## Installation

To get started, install the logger repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/logger
```

## Configuration

There are basically two ways to start logging. Once by directly calling a `Zaphyr\Logger\Logger` instance and by using a
`Zaphyr\Logger\LogManager` instance. We will take a closer look at both options and their advantages and disadvantages
below.

### Configuring a Logger instance

A `Zaphyr\Logger\Logger` instance does not require much configuration effort:

```php
$logger = new Zaphyr\Logger\Logger('app', [
    new Zaphyr\Logger\Handlers\FileHandler('/path/to/log/file.log')
]);
```

However, if you want to use multiple log channels in an application, the use can quickly become confusing. You can read
more about the available log handlers [here](#handlers).

### Configuring a LogManager instance

A little more configuration work is required to use a `Zaphyr\Logger\LogManager` instance. On the other hand, using a
log manager after a successful configuration is easier and simplifies the use of multiple
[log handlers](#handlers). So let's build a `Zaphyr\Logger\LogManager` instance first:

```php
$logManager = new Zaphyr\Logger\LogManager([
    'dev',
    [
        'dev' => [
            'handlers' => [
                'file' => [
                    'storagePath' => '/path/to/log/file.log',
                ],
            ],
        ],
        'prod' => [
            'handlers' => [
                'rotate' => [
                    'interval' => 'day',
                    'storagePath' => '/path/to/log/directory',
                ],
                'mail' => [
                    'dsn' => 'smtp://user:pass@smtp.example.com:25',
                    'from' => 'noreply@example.com',
                    'to' => 'admin@example.com',
                    'subject' => 'Log message received',
                ],
            ],
        ],
    ]
]);
```

The configuration for a log manager instance may seem a bit confusing at first. So let's take a look at each part of
the configuration.

In line 2 of our `Zaphyr\Logger\LogManager` example we first define a default log channel, in this case `dev`. Whenever
we make a log entry with the log manager and do not pass a log channel to the `logger()` method, the default log channel
is used. You can read more about using the `logger()` method [here](#logging-with-a-logmanager-instance).

```php
$devLogger = $logManager->logger(); // Uses the default "dev" log channel
```

In lines 4 to 10 we now configure our `dev` log channel, which was already defined in line 2 of our example as the
default log channel. Inside the `dev` configuration we define our log handlers to be used for this log channel.
We will learn more about the various [log handlers](#handlers) later.

In lines 11 to 24 we have defined another log channel, in this case named `prod`. If we call the `logger()` method with
the parameter `prod`, the logger entries of the defined log channel are used:

```php
$prodLogger = $logManager->logger('prod'); // Uses the defined "prod" log channel
```

> [!NOTE]
> Currently, the `LogManager` only allows usage of the [available log handlers](#handlers). It is currently not
> possible to use custom log handlers without extending the `LogManager`. However, such a functionality is planned
> and will be implemented in the near future!

## Logging

When it comes to logging in your application, this logging service offers you different log levels
as [described below](#log-levels).

### Logging with a Logger instance

If you have created a logging stack with a `Zaphyr\Logger\Logger` instance, you can easily create log messages:

```php
$logger->alert("Something went wrong!");
```

You can also pass an array as context data to log methods. This is meant to hold extra information that does not fit
well in a string:

```php
$logger->info('User {username} created', ['username' => 'John Doe']);
```

### Logging with a LogManager instance

Once you have created a logging stack with a `Zaphyr\Logger\LogManager` instance, you can call the `logger()` method and
start logging. The `Zaphyr\Logger\LogManager` instance also offers you the possibility to select different `Logger`
instances for the respective logging.

#### Use the default logger

If you want to call the default logger, just use the `logger()` method without any additional parameters. After calling
the `logger()` method you can use any [available log level method](#log-levels):

```php
$logManager->logger()->alert("Something went wrong!");
```

#### Use a non-default logger

If you want to use a different logger instance than default, pass the name of the desired logger to the `logger()` method:

```php
$logManager->logger('app')->info('User {username} created', ['username' => 'John Doe']);
```

You can read how to configure different loggers with the `Zaphyr\Logger\LogManager`
[here](#configuring-a-logmanager-instance).

### Log levels

The logging service, offers all the log levels defined in the
[RFC 5424 specification](https://www.rfc-editor.org/rfc/rfc5424#page-11). In descending order of severity, these log
levels are: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, and `debug`.

| Level                  | Description                      |
|:-----------------------|:---------------------------------|
| `$logger->emergency()` | System is unusable               |
| `$logger->alert()`     | Action must be taken immediately |
| `$logger->critical()`  | Critical conditions              |
| `$logger->error()`     | Error conditions                 |
| `$logger->warning()`   | Warning conditions               |
| `$logger->notice()`    | Normal but significant condition |
| `$logger->info()`      | Informational messages           |
| `$logger->debug()`     | Debug-level messages             |

## Handlers

The logging service comes with a handful of log handlers. They decide how log messages reach you or the responsible
project owner. If the default handlers are not enough, you can
[simply create a custom handler](#create-a-custom-handler).

### FileHandler

The `Zaphyr\Logger\Handlers\FileHandler` is the simplest of all handlers. This handler simply writes log messages to the
log file passed in the constructor. A new log entry is written at the end of the file:

```php
$fileHandler = new Zaphyr\Logger\Handlers\FileHandler('/path/to/log/file.log');
```

If the log file specified in the constructor does not exist, the `Zaphyr\Logger\Handlers\FileHandler` instance creates
it on its own.

> [!WARNING]
> The `FileHandler` stores log messages in a single file. So this handler should either be used in small projects
> or it should be ensured that the log file is deleted or overwritten at regular intervals.
> Otherwise, this log file can reach a considerable size and may overload the available storage space of your server!

### MailHandler

The `Zaphyr\Logger\Handlers\MailHandler` does exactly what the name says – sending log messages as emails. This handler
uses the `symfony\mail` package to send messages. You can read more about the symfony mailer package
[here](https://symfony.com/doc/5.4/mailer.html).

So you first create the email configuration and then inject it to your `Zaphyr\Logger\Handlers\MailHandler` instance:

```php
$transport = Symfony\Component\Mailer\Transport::fromDsn('smtp://localhost');
$mailer = new Symfony\Component\Mailer\Mailer($transport);
$email = (new Symfony\Component\Mime\Email())
    ->from('noreply@example.com')
    ->to('admin@example.com')
    ->subject('Log message received');

$mailHandler = new Zaphyr\Logger\Handlers\MailHandler($mailer, $email);
```

Now you will get all log messages sent to the email address specified in your email configuration.

### RotateHandler

The `Zaphyr\Logger\Handlers\RotateHandler` is also a log file handler. However, an interval can be set for this handler,
at which intervals a new log file should be created. By default, this handler creates a new log file for each day.

Simply inject the path to the folder in which the log files should be saved and enter the desired interval:

```php
$rotateHandler = new Zaphyr\Logger\Handlers\RotateHandler('/path/to/log/directory', 'day');
```

If the directory for the log files does not exist, the `Zaphyr\Logger\Handlers\RotateHandler` creates this directory
independently.

#### Intervals

The following intervals can be used to create log files:

| Interval | Description                         |
|:---------|:------------------------------------|
| `hour`   | Creates a new log file every hour   |
| `day`    | Creates a new log file every day    |
| `week`   | Creates a new log file every week   |
| `month`  | Creates a new log file every month  |
| `year`   | Creates a new log file every year   |

### Create a custom handler

It is also possible to create a custom log handler. To do this, you first create your own handler class which
implements the `Zaphyr\Logger\Contracts\HandlerInterface`:

```php
class MyCustomHandler implements Zaphyr\Logger\Contracts\HandlerInterface
{
    /**
     * {@inheritdoc}
     */
    public function add(string $name, string $level, string $message, array $context = []): void
    {
        // your custom handler logic
    }
}
```

Now you can pass your previously created handler instance to a `Zaphyr\Logger\Logger` and use the logger as already
described:

```php
$handlers = [
    new MyCustomHandler(),
];

$logger = new Zaphyr\Logger\Logger('app', $handlers);
```

### Usage of multiple handlers

As you may have seen in the previous examples, multiple log handlers can be triggered simultaneously as soon as a log
message is registered.

For example, if you not only want to store a log message in a file, but also send it as an email to a responsible
administrator, you can do this for the use of a [Logger](#configuring-a-logger-instance) instance, as follows:

```php
$transport = Symfony\Component\Mailer\Transport::fromDsn('smtp://localhost');
$mailer = new Symfony\Component\Mailer\Mailer($transport);
$email = (new Symfony\Component\Mime\Email())
    ->from('noreply@example.com')
    ->to('admin@example.com')
    ->subject('Log message received');

$handlers = [
    new Zaphyr\Logger\Handlers\FileHandler('/path/to/log/file.log'),
    new Zaphyr\Logger\Handlers\MailHandler($mailer, $email),
];

$logger = new Zaphyr\Logger\Logger('app', $handlers);
```

The use of multiple log handlers is also possible with the use of
a [LogManager](#configuring-a-logmanager-instance) instance:

```php
$defaultChannel ='app';
$channels = [
    'app' => [
        'handlers' => [
            'rotate' => [
                'interval' => 'day',
                'storagePath' => '/path/to/log/directory',
            ],
            'mail' => [
                'dsn' => 'smtp://user:pass@smtp.example.com:25',
                'from' => 'noreply@example.com',
                'to' => 'admin@example.com',
                'subject' => 'Log message received',
            ],
        ],
    ],
];

$logManager = new Zapyhr\Logger\LogManager($defaultChannel, $channels);
```

## Formatters

Formatters, as the name suggests, determine the format in which log messages are written to the log files.
By default, the logging service offers only one formatter, the `Zaphyr\Logger\Formatters\DefaultFormatter`.

The default formatter creates a log message in the following format:

```
[{date}] {name}.{level}: {message} [{context}] [{exception}]

// [2022-09-13 09:41:00] app.alert: Something went wrong! […] […]
```

### Create a custom formatter

It could be possible that you want to save a log message in a different format. In this case you can simply create
a custom formatter instance:

```php
class MyCustomFormatter implements Zaphyr\Logger\Contracts\FormatterInterface
{
    /**
     * {@inheritdoc}
     */
    public function interpolate(string $name, string $level, string $message, array $context = []): string
    {
        // Your custom formatter logic
    }
}
```

A good starting point for creating your custom formatter is to take a look at the
[DefaultFormatter](https://github.com/zaphyr-org/logger/blob/master/src/Formatters/DefaultFormatter.php).

#### Add your custom formatter to a handler instance

After you have created your custom formatter, you pass it to your handler instance:

```php
$formatter = new MyCustomFormatter();
$handler = new Zaphyr\Logger\Handlers\FileHandler('/path/to/file.log', $formatter);
```

Now your log messages will be written in your custom format.
