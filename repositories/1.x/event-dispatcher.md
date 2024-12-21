# Event Dispatcher

An efficient [PSR-14](https://www.php-fig.org/psr/psr-14/) event dispatcher.

## Installation

To get started, install the event-dispatcher repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/event-dispatcher
```

## Basic usage

Events enable your code to respond to specific occurrences or actions within your application, promoting modularity and
flexibility by allowing different components to react independently to the same event.

In the following example, we will create a `UserEvent` class that will be dispatched each time a user logs in to your
application. We will then create a listener that will be called each time the `UserEvent` is dispatched:

```php
class UserEvent
{
    public function isLoggedIn(): bool
    {
        return true;
    }
}

$listenerProvider = new Zaphyr\EventDispatcher\ListenerProvider();
$listenerProvider->addListener(UserEvent::class, function (UserEvent $event) {
    if ($event->isLoggedIn()) {
        // …
    }
});

$eventDispatcher = new Zaphyr\EventDispatcher\EventDispatcher($listenerProvider);
$eventDispatcher->dispatch(new UserEvent());
```

In the above example we have created a new `Zaphyr\EventDispatcher\ListenerProvider` instance and added a listener to
the `addListener` method. The `addListener` method accepts two arguments: the event name and a callable. The callable
will be called each time the event is dispatched. We then created a new `Zaphyr\EventDispatcher\EventDispatcher` instance
and passed the `ListenerProvider` instance to the constructor. Finally, we dispatched our example event by calling
the `dispatch` method on the `EventDispatcher` instance. We will now take a closer look at the `ListenerProvider` and
`EventDispatcher` classes.

## Subscribe to events

The `Zaphyr\EventDispatcher\ListenerProvider` class is responsible for subscribing to events and registering listeners
for these events. The `ListenerProvider` class implements the `Psr\EventDispatcher\ListenerProviderInterface`:

```php
$listenerProvider = new Zaphyr\EventDispatcher\ListenerProvider();
```

### Add listeners

The `addListener` method allows you to add a listener to a specific event. This method accepts two arguments: the event
name and a callable. The event name must be the fully qualified class name of the event object. The callable will be
called each time the event is dispatched. The callable must accept one argument: the event object. The callable can be
any PHP callable, including a closure or an invokable object:

```php
$listenerProvider->addListener($eventName, $listenerCallable);
```

In the following example, we created a `UserListener` class with the `__invoke` method. The `__invoke` method will be
called each time the `UserEvent` is dispatched:

```php
class UserListener
{
    public function __invoke(UserEvent $event)
    {
        if ($event->isLoggedIn()) {
            // …
        }
    }
}

$listenerProvider->addListener(UserEvent::class, new UserListener());
```

> [!NOTE]
> The `addListener` method is not part of the PSR-14 `Psr\EventDispatcher\ListenerProviderInterface`.

### Prioritize listeners

The `addListener` method allows to influence the caller order of the event listeners by providing a priority. The higher
the priority number of a listener is, the earlier the listener is called:

```php
$listenerProvider->addListener($eventName, $listenerCallable2, -100);
$listenerProvider->addListener($eventName, $listenerCallable1, 100);
```

If listeners have the same priority, they are called in the order they were added:

```php
$listenerProvider->addListener($eventName, $listenerCallable1, 100);
$listenerProvider->addListener($eventName, $listenerCallable2, 100);
```

The `Zaphyr\EventDispatcher\ListenerProvider` exposes a set of constants that can be used to prioritize listeners:

- `Zaphyr\EventDispatcher\ListenerProvider::PRIORITY_LOW` (-100)
- `Zaphyr\EventDispatcher\ListenerProvider::PRIORITY_NORMAL` (0)
- `Zaphyr\EventDispatcher\ListenerProvider::PRIORITY_HIGH` (100)

```php
$listenerProvider->addListener(
    $eventName,
    $listenerCallable,
    Zaphyr\EventDispatcher\ListenerProvider::PRIORITY_HIGH
);
```

## Dispatch events

The `Zaphyr\EventDispatcher\EventDispatcher` class is the central point of the event dispatcher repository. It is
responsible for dispatching events to all subscribed listeners. The `EventDispatcher` class implements the
`Psr\EventDispatcher\EventDispatcherInterface`. After you have created all your desired listeners, you can create a new
`EventDispatcher` instance and pass the `ListenerProvider` instance to the constructor:

```php
$eventDispatcher = new Zaphyr\EventDispatcher\EventDispatcher($listenerProvider);
```

Now that you have created a new `EventDispatcher` instance, you can dispatch events by calling the `dispatch` method on
the `EventDispatcher` instance. The `dispatch` method accepts one argument: the event object.

```php
$eventDispatcher->dispatch($eventObject);
```

## Stop event propagation

Sometimes you may want to stop the propagation of an event to other listeners. The event-dispatcher repository provides
a `Zaphyr\EventDispatcher\AbstractStoppableEvent` class that you can extend to create stoppable events. The
`AbstractStoppableEvent` class implements the `Psr\EventDispatcher\StoppableEventInterface`:

```php
class StoppableEvent extends Zaphyr\EventDispatcher\AbstractStoppableEvent
{
   // …
}
```

The `AbstractStoppableEvent` class provides a `stopPropagation` method that you can call to stop the propagation of the
event to other listeners:

```php
$listenerProvider->addListener(StoppableEvent::class, function (StoppableEvent $event) {
    $event->stopPropagation();
});
```
