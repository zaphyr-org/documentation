# Events

This document provides an overview of event handling in ZAPHYR, including how to create events and listeners,
and how to configure them.

## Create Events

Events are a powerful way to decouple different parts of your application. They signal that something has occurred, and
listeners can respond accordingly, making your application more flexible and easier to maintain.

You can create a custom event using the command-line tool:

```bash
php bin/zaphyr create:event MyEvent
```

By default, this will generate a new event class in the `app/Events` directory with the following structure:

```php
namespace App\Events;

class MyEvent
{
    public function __construct()
    {
    }
}
```

If you want to generate the event in a different namespace, use the `--namespace` option:

```bash
php bin/zaphyr create:event MyEvent --namespace="App\Custom\Events"
```

### Create Stoppable Events

In some cases, you may want to prevent further listeners from being called after a specific condition is met. ZAPHYR
supports this via stoppable events.

To create a stoppable event, you can use the `--stoppable` option when generating the event:

```bash
php bin/zaphyr create:event MyStoppableEvent --stoppable
```

This creates a class that extends the `AbstractStoppableEvent`:

```php
namespace App\Events;

use Zaphyr\EventDispatcher\AbstractStoppableEvent;

class MyStoppableEvent extends AbstractStoppableEvent
{
    public function __construct()
    {
    }
}
```

To stop propagation within a listener, simply call:

```php
$event->stopPropagation();
```

This will prevent any subsequent listeners from being triggered for that event.

## Dispatch Events

ZAPHYR's service container includes a powerful [event dispatcher](/docs/repositories/latest/event-dispatcher) that
adheres to the [PSR-14 Event Dispatcher](https://www.php-fig.org/psr/psr-14/) standard.

Instead of directly invoking listeners, your application simply dispatches an event. The dispatcher will automatically
call all relevant listeners in the order of their configured priority. This design allows different parts of your
application to respond to an event without being tightly coupled to the code that triggers it.

You can retrieve the event dispatcher from the service container like so:

```php
use Psr\EventDispatcher\EventDispatcherInterface;

$dispatcher = $container->get(EventDispatcherInterface::class);
```

Once you have access to the dispatcher, dispatching an event is straightforward. Here's a simple example:

```php
use App\Events\MyEvent;

$dispatcher->dispatch(new MyEvent());
```

If you're using dependency injection in a service or controller, the dispatcher can also be injected directly:

```php
use App\Events\MyEvent;
use Psr\EventDispatcher\EventDispatcherInterface;

class SomeService
{
    public function __construct(private EventDispatcherInterface $dispatcher)
    {
    }

    public function handle(): void
    {
        $this->dispatcher->dispatch(new MyEvent());
    }
}
```

## Create Listeners

Listeners are responsible for handling the logic that should occur in response to dispatched events. Each listener is a
callable class, typically defined as an `__invoke` method, that reacts when its associated event is triggered.

You can create a new listener using the CLI:

```bash
php bin/zaphyr create:listener MyListener
```

By default, this will generate a listener in the `app/Listeners` directory:

```php
namespace App\Listeners;

class MyListener
{
    /**
     * @param \object $event
     */
    public function __invoke(\object $event): void
    {
    }
}
```

To generate the listener in a different namespace, use the `--namespace` option:

```bash
php bin/zaphyr create:listener MyListener --namespace="App\Custom\Listeners"
```

By default, the listener accepts a generic `object` parameter. However, you can bind it to a specific event class using
the `--event` option:

```bash
php bin/zaphyr create:listener MyListener --event="App\Events\MyEvent"
```

This will generate a listener with a type-hinted event parameter and makes your event handling more explicit:

```php
namespace App\Listeners;

class MyListener
{
    /**
     * @param \App\Events\MyEvent $event
     */
    public function __invoke(\App\Events\MyEvent $event): void
    {
    }
}
```

## Framework Events

The framework provides a set of built-in events that allow you to hook into key moments in the application's lifecycle.
These events enable you to extend or customize the behavior of the application by responding to specific system actions.

Below is an overview of the core framework events you can listen to:

### Console Events

These events are related to the execution of console commands in your application. They allow you to track or respond to
specific moments during a command's lifecycle, such as startup, successful execution, or failure.

| Event Name                                                      | Description                                            |
|-----------------------------------------------------------------|--------------------------------------------------------|
| `Zaphyr\Framework\Events\Console\Commands\CommandStartingEvent` | Dispatched when a console command begins execution.    |
| `Zaphyr\Framework\Events\Console\Commands\CommandFinishedEvent` | Dispatched after a console command completes.          |
| `Zaphyr\Framework\Events\Console\Commands\CommandFailedEvent`   | Dispatched when a console command encounters an error. |

### HTTP Events

These events are triggered during the lifecycle of HTTP requests. They are useful for monitoring request handling,
logging, analytics, or implementing custom logic based on the request state.

| Event Name                                          | Description                                              |
|-----------------------------------------------------|----------------------------------------------------------|
| `Zaphyr\Framework\Events\Http\RequestStartingEvent` | Dispatched when an HTTP request starts processing.       |
| `Zaphyr\Framework\Events\Http\RequestFinishedEvent` | Dispatched after an HTTP request has been processed.     |
| `Zaphyr\Framework\Events\Http\RequestFailedEvent`   | Dispatched when an HTTP request fails during processing. |

### Maintenance Events

Maintenance events allow you to react when the application enters or exits maintenance mode. This is especially useful
for informing external services, handling queued jobs, or preventing scheduled tasks during downtime.

| Event Name                                                     | Description                                              |
|----------------------------------------------------------------|----------------------------------------------------------|
| `Zaphyr\Framework\Events\Maintenance\MaintenanceEnabledEvent`  | Dispatched when the application enters maintenance mode. |
| `Zaphyr\Framework\Events\Maintenance\MaintenanceDisabledEvent` | Dispatched when the application exits maintenance mode.  |

## Config Listeners

You can configure event listeners declaratively through the `config/events.yaml` file. This approach keeps your event
system clean and centralized, making it easier to manage and scale your application's event handling.

To register listeners for a specific event, define the event class as a key and list the fully qualified class names of
the corresponding listeners:

```yaml
listeners:
    Zaphyr\Framework\Events\Http\RequestStartingEvent:
        - App\Listeners\RequestStartingListener
```

Each time the specified event is dispatched, the listed listeners will be invoked in the order they appear, unless a
priority is defined.

### Listeners Priority

Listeners can be assigned a priority value to control their execution order. A higher priority means the listener will
be executed earlier. This is useful when the behavior of one listener depends on the execution of another.

```yaml
listeners:
    Zaphyr\Framework\Events\Http\RequestFinishedEvent:
        - { listener: App\Listeners\LogRequestListener, priority: 200 }
        - { listener: App\Listeners\CloseConnectionListener, priority: 50 }
```

In this example, `LogRequestListener` will run before `CloseConnectionListener`.

### Ignore Listeners

In some cases, you may want to prevent certain listeners from being registered for a specific event. You can exclude
listeners using the `listeners_ignore` section:

```yaml
listeners_ignore:
    Zaphyr\Framework\Events\Http\RequestStartingEvent:
        - App\Listeners\RequestStartingListener
```

This is especially useful for overriding or disabling default listeners provided by packages or the framework.

## Cache Events

ZAPHYR allows you to cache your application's event listeners to improve performance, especially in production
environments. By compiling all event-to-listener mappings into a single file, the framework can skip scanning and
resolving listeners dynamically at runtime.

To generate the event listener cache, run the following command:

```bash
php bin/zaphyr events:cache
```

This command will scan your application for all registered events and listeners, compile them into a cache file, and
store it for a quick lookup on subsequent requests.

If you've added, removed, or modified any event listeners, you’ll need to clear the existing cache to reflect your
changes. You can do this with:

```bash
php bin/zaphyr events:clear
```
