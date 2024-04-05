# Logger

You want to know what's happening "under the hood" of your application? ZAPHYR provides a robust
[PSR-3](https://www.php-fig.org/psr/psr-3/) logging service.

## Installation

To get started, install the logger repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/logger
```

## Configuration

There are basically two ways to start logging. Once by directly calling a `Zaphyr\Logger\Logger` instance and by using a
`Zaphyr\Logger\LogManager` instance. We will take a closer look at both options.

### Configuring a Logger instance

A `Zaphyr\Logger\Logger` instance does not require much configuration effort. You just need to create a logger instance,
set a name (e.g. `app`) for the logger and define at least one log handler (e.g. `FileHandler`).
You can read more about the available log handlers [here](#handlers).

```php
$logger = new Zaphyr\Logger\Logger('app', [
    new Zaphyr\Logger\Handlers\FileHandler('/path/to/log/file.log')
]);

$logger->debug('This is a debug message');
```

### Configuring a LogManager instance

If you want to use multiple logger instances in your application, it is recommended to use the `Zaphyr\Logger\LogManager`.
Via the LogManager multiple logger instances can be configured, and you can decide later which log stack to use.
For example, you can create different log stacks for your development application and your production application.

First we pass a default stack to the LogManager. In our example this is the `development` stack.  Whenever we make a log
entry via the LogManager and do not pass a logger name to the `logger()` method, the default stack is used.

Next, we define the desired logger stacks and specify which log handlers each stack should use. In our example we have a
`development` stack and a `production` stack:

```php
$logManager = new Zaphyr\Logger\LogManager(
    'development',
    [
        'development' => [
            new Zaphyr\Logger\Handlers\FileHandler('/path/to/log/file.log'),
        ],
        'production' => [
            new Zaphyr\Logger\Handlers\RotateHandler(
                '/path/to/log/directory',
                Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_DAY
            ),
        ],
    ]
);

$logManager->logger()->debug('This is a debug message'); // Uses the default "development" logger stack
$logManager->logger('production')->debug('This is a debug message'); // Uses the defined "production" logger stack
```

## Logging

The logging service, offers all the log levels defined in the
[RFC 5424 specification](https://www.rfc-editor.org/rfc/rfc5424#page-11). In descending order of severity, these log
levels are:

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

### Additional context data

You can also pass an array as context data to log methods. This is meant to hold extra information that does not fit
well in a string:

```php
$logger->info('User {username} created', ['username' => 'John Doe']);
```

### Exception data

It is also possible to pass `Exception` instances to the log message. Just create a context array with `exception`
as key and the corresponding Exception instance as value:

```php
$exception  = new Exception('Something went wrong');

$logger->alert('Something went wrong', ['exception' => $exception]);
```

In the log message there is now also a detailed representation of the passed Exception.

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

The `MailHandler` does exactly what the name says – sending log messages as emails. This handler uses the
`symfony\mail` package to send messages. You can read more about the symfony mailer package
[here](https://symfony.com/doc/6.2/mailer.html).

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
$rotateHandler = new Zaphyr\Logger\Handlers\RotateHandler('/path/to/log/directory', Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_DAY);
```

#### Intervals

The following intervals can be used to create log files:

| Interval                                               | Description                         |
|:-------------------------------------------------------|:------------------------------------|
| `Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_HOUR`  | Creates a new log file every hour   |
| `Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_DAY`   | Creates a new log file every day    |
| `Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_WEEK`  | Creates a new log file every week   |
| `Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_MONTH` | Creates a new log file every month  |
| `Zaphyr\Logger\Handlers\RotateHandler::INTERVAL_YEAR`  | Creates a new log file every year   |

### Usage of multiple handlers

As you may have seen in the previous examples, multiple log handlers can be triggered simultaneously as soon as a log
message is registered.

For example, if you not only want to store a log message in a file, but also send it as an email to a responsible
administrator, you can do this as follows:

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

$logger = new Zaphyr\Logger\Logger('production', $handlers);
```

The handler stack is then processed from top to bottom. In the above example, the log message is first written to the
log file and then sent as an e-mail.

### Create a custom handler

It is also possible to create a custom log handler. To do this, you first create your own handler class which
implements the ` Zaphyr\Logger\Contracts\HandlerInterface`:

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

Now you can pass your previously created handler instance to a logger instance and use the logger as already described:

```php
$handlers = [
    new MyCustomHandler(),
];

$logger = new Zaphyr\Logger\Logger('development', $handlers);
```

## Formatters

Formatters, as the name suggests, determine the format in which log messages are written to the log files.
All log handlers provided by the logging service use the `Zaphyr\Logger\Formatters\LineFormatter` by default.

The `LineFormatter` creates a log message in the following format:

```
[{date}] {name}.{level}: {message} [{context}] [{exception}]

// [2022-09-13 09:41:00] production.ALERT: Something went wrong! [] []
```

### Available formatters

The logging service comes with a handful of useful log formats:

| Formatters                               | Description                                                                                                           |
|:-----------------------------------------|:----------------------------------------------------------------------------------------------------------------------|
| `Zaphyr\Logger\Formatters\LineFormatter` | Creates single log line messages in the format shown above. This is the default formatter                             |
| `Zaphyr\Logger\Formatters\HtmlFormatter` | Creates log messages in HTML format. This formatter is useful e.g. if log messages are sent with the `MailFormatter`. |
| `Zaphyr\Logger\Formatters\JsonFormatter` | Creates log messages in JSON format.                                                                                  |


### Change the formatter of a handler

If you want to use a different formatter then the default formatter for a log handler, you can pass it as the last
argument to the desired log handler instance. In the following example we exchange the`Zaphyr\Logger\Formatters\LogFormatter`
of the `Zaphyr\Logger\Handlers\MailHandler` for the `Zaphyr\Logger\Handlers\HtmlFormatter`:

```php
$transport = Symfony\Component\Mailer\Transport::fromDsn('smtp://localhost');
$mailer = new Symfony\Component\Mailer\Mailer($transport);
$email = (new Symfony\Component\Mime\Email())
    ->from('noreply@example.com')
    ->to('admin@example.com')
    ->subject('Log message received');

new Zaphyr\Logger\Handlers\MailHandler($mailer, $email, new Zaphyr\Logger\Formatters\HtmlFormatter());
```

### Create a custom formatter

It could be possible that you want to save a log message in a different format. In this case you can simply create
a custom formatter instance:

```php
class MyCustomFormatter extends Zaphyr\Logger\Formatters\AbstractFormatter
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
[LineFormatter](https://github.com/zaphyr-org/logger/blob/master/src/Formatters/LineFormatter.php).

#### Add your custom formatter to a handler instance

After you have created your custom formatter, simply pass it to your handler instance:

```php
$formatter = new MyCustomFormatter();
$handler = new Zaphyr\Logger\Handlers\FileHandler('/path/to/file.log', $formatter);
```

Now your log messages will be written in your custom format.
