# Logger

The framework includes a simple yet powerful [PSR-3](https://www.php-fig.org/psr/psr-3/) compliant logging system that
enables you to log messages with various levels of severity. It is built on top of
[logger repository](/docs/repositories/latest/logger), offering a consistent, extendable, and modular way to manage log
output across your application.

## Configuration

Logger behavior can be configured in your application’s `config/logging.yaml` file and `.env` file. The system supports
multiple channels, allowing logs to be directed to different files, outputs, or services based on severity levels,
environments, or custom criteria.

### Default Log Channel

You can set the default logging channel via the `.env` file:

```bash
LOG_DEFAULT_CHANNEL=daily
```

This channel will be used whenever logging is performed without explicitly specifying a channel.

### Log Channels

Multiple log channels can be defined in the `config/logging.yaml` file. Each channel can have one or more handlers,
making it easy to direct logs to files, email, external services, or other storage formats:

```yaml
channels:
    daily:
        handlers:
            rotate:
                directory: '%path:storage%/logs'
                interval: day
                level: '%env:LOG_LEVEL%'
                formatter: Zaphyr\Logger\Formatters\LineFormatter
    single:
        handlers:
            file:
                filename: '%path:storage%/logs/zaphyr.log'
                level: '%env:LOG_LEVEL%'
                formatter: Zaphyr\Logger\Formatters\LineFormatter
```

In this example, two log channels are defined: `daily` and `single`. The `daily` channel rotates logs daily, while the
`single` channel writes logs to a single file. You can specify the
[log level](/docs/repositories/latest/logger#set-log-level) and
[formatter](/docs/repositories/latest/logger#formatters) for each channel.

You can also define multiple handlers within a single channel, enabling advanced use cases such as sending critical logs
by email while still writing all logs to a file:

```yaml
channels:
    prod:
        handlers:
            file:
                filename: '%path:storage%/logs/combined.log'
                level: '%env:LOG_LEVEL%'
                formatter: Zaphyr\Logger\Formatters\LineFormatter
            mail:
                dsn: '%env:LOG_MAIL_CONNECTION%'
                from: '%env:LOG_MAIL_FROM%'
                to: '%env:LOG_MAIL_TO%'
                subject: '%env:LOG_MAIL_SUBJECT%'
                level: '%env:LOG_LEVEL%'
                formatter: Zaphyr\Logger\Formatters\HtmlFormatter
```

## Usage

To log messages in your application, you can either:

- Inject the `Psr\Log\LoggerInterface` to use the default log channel.
- Inject the `Zaphyr\Logger\Contracts\LogManagerInterface` to use a specific log channel.

### Default Log Channel

The `LoggerInterface` lets you log messages at standard PSR-3 severity levels:
`debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert`, and `emergency`.

```php
use Psr\Log\LoggerInterface;

class MyService
{
    public function __construct(private LoggerInterface $logger)
    {
    }

    public function doSomething()
    {
        $this->logger->error('An error occurred while doing something.');
    }
}
```

### Specific Log Channel

To log to a specific channel defined in your `config/logging.yaml` file, use the `LogManagerInterface`:

```php
use Zaphyr\Logger\Contracts\LogManagerInterface;

class MyService
{
    public function __construct(private LogManagerInterface $logManager)
    {
    }

    public function doSomething()
    {
        $logger = $this->logManager->channel('daily');
        $logger->info('This is an informational message.');
    }
}
```

## Ignore Errors

Some exceptions are expected in web applications and don’t need to be logged every time they occur. To reduce noise in
your logs, you can tell the logger to ignore specific exceptions.

Add the fully qualified class names of these exceptions to the `ignore` section in your `config/logging.yaml` file:

```yaml
ignore:
    - Zaphyr\Framework\Contracts\Http\Exceptions\HttpExceptionInterface
    - Zaphyr\Router\Exceptions\MethodNotAllowedException
    - Zaphyr\Router\Exceptions\NotFoundException
```

Any exception matching an entry in this list will be excluded from the log output.

## Remove Log Files

To remove all log files from your application, use the following command:

```bash
php bin/zaphyr logs:clear
```

This command deletes all files in the `storage/logs` directory.

> [!WARNING]
> Use this in production only when necessary, as it will permanently erase all log data.
