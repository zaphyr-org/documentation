# Session

A session handler repository for maintaining user state across multiple requests.

## Installation

To get started, install the session repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/session
```

## Session

HTTP is a stateless protocol, which means that the server does not keep any information (state) between two requests.
This is a problem when you want to keep track of a user's activity on your website. For example, if you want to check
if a user is logged in, you need to keep track of that somehow. This is, where the session repository comes into play.
The session repository does not store session data in the `$_SESSION` super global. Instead, the session repository uses
custom [session handlers](#session-handlers) to store the session data.

### Create a session

To start a session, you need to create a new `Zaphyr\Session\Session` instance and pass a session name and session
handler to it. The session name is used to identify the session cookie. The session handler is responsible for storing
the session data. The session handler must implement
[PHP's built in `SessionHandlerInterface`](https://www.php.net/manual/en/class.sessionhandlerinterface.php). In our
example we will use the `Zaphyr\Session\Handler\FileSessionHandler` which stores the session data in files. We will
learn about session handlers in more detail in the [session handlers](#session-handlers) section.

```php
$handler = new Zaphyr\Session\Handler\FileSessionHandler('/path/to/storage');

$session = new Zaphyr\Session\Session('session_name', $handler);
```

### Start session

To start a new session, you need to call the `start` method:

```php
$session->start();
```

If you want to check if a session is already started, you can use the `isStarted` method:

```php
$session->isStarted();
```

### Session token

The session token is used to prevent XSS attacks. The session token is generated automatically when the session is
started. You can retrieve the session token with the `getToken` method:

```php
$session->getToken();
```

You can also set a custom session token with the `setToken` method. The provided token must be a string of 40
alphanumeric
characters. If the provided string does not fit the criteria, the `setToken` method will create a new random token:

```php
$session->setToken('oRILZgj1i94DNaAUotusSOCR7WvymbaLSMOxYhNF');
```

> [!NOTE]
> The session token is stored in the session data under the `_token` key. The `_token` key is reserved and should not be
> used for other session data!

### Session ID

The session ID is used to identify the session. The session ID is generated automatically when the session is
instantiated.
You can retrieve the session ID with the `getId` method:

```php
$session->getId();
```

You can also set a custom session ID with the `setId` method. The provided ID must be a string of 40 alphanumeric
characters. If the provided string does not fit the criteria, the `setId` method will create a new random ID:

```php
$session->setId('oRILZgj1i94DNaAUotusSOCR7WvymbaLSMOxYhNF');
```

### Session name

The session name is used to identify the session cookie. You can retrieve the session name with the `getName` method:

```php
$session->getName();
```

You can also set a custom session name with the `setName` method:

```php
$session->setName('oRILZgj1i94DNaAUotusSOCR7WvymbaLSMOxYhNF');
```

### Retrieve session data

To get a single session value, you may use the `get` method. This method will return the value of the session data
element, or `null` by default if the value is not present:

```php
$session->get('key');
```

You may pass a default value as the second argument to the `get` method. This value will be returned if the session
value does not exist:

```php
$session->get('key', 'default');
```

If you would like to get all the session data, you can use the `all` method:

```php
$session->all();
```

### Store session data

To store a session value, you may use the `set` method:

```php
$session->set('key', 'value');
```

If you want to push a value onto an array session value, you may use the `add` method:

```php
$session->add('data.key', 'value');
```

### Determine if session data exists

To determine if a session value is present, you may use the `has` method. The `has` method will return `true` if the
session value exists:

```php
$session->has('key');
```

### Remove session data

If you would like to remove a session value, you may use the `remove` method. The `remove` method will remove the value
of the session data element if it exists:

```php
$session->remove('key');
```

You may also delete multiple session values at once by passing an array of keys to the `removeMultiple` method:

```php
$session->removeMultiple(['key', 'anotherKey']);
```

To remove all session values, you may use the `flush` method:

```php
$session->flush();
```

### Migrate session

If you want to generate a new session ID, you may use the `migrate` method:

```php
$session->migrate();
```

If you also want to destroy the existing session data, pass `true` as an argument to the `migrate` method:

```php
$session->migrate(true);
```

### Regenerate session

To regenerate the session token, you may use the `regenerate` method:

```php
$session->regenerate();
```

If you also want to destroy the existing session data, pass `true` as an argument to the `regenerate` method:

```php
$session->regenerate(true);
```

### Invalidate session

The `invalidate` method will flush all session data and regenerates the session ID:

```php
$session->invalidate();
```

### Save session

The `save` method will clear all flash data, serialize the session data, and write them to the session storage:

```php
$session->save();
```

### Flash session data

The session repository comes with a flash data feature. Flash data are session values that will be available for the
next request only. After the next request, the flash data will be deleted automatically. Flash data are useful for
short-lived messages like "Item has been successfully deleted".

To store a flash value, you may use the `flash` method:

```php
$session->flash('success', 'Item has been successfully deleted');
```

If you want to persist the flash data only for the current request, you may use the `now` method:

```php
$session->now('success', 'Item has been successfully deleted');
```

To store your flash data for an additional request, you may use the `reflash` method:

```php
$session->reflash();
```

To keep specific flash data for an additional request, you may use the `keep` method:

```php
$session->keep(['success']);
```

To clear all flash data, you may use the `clearFlashData` method. This method will also be called when you are using
the `save` method:

```php
$session->clearFlashData();
```

> [!NOTE]
> The flash session data are stored in the session data under the `_flash` key. The `_flash` key is reserved and should
> not be used for other session data!

### Flash input data

The session repository also provides some handy methods to store input data. Input data are session values that will be
available for the next request only. After the next request, the input data will be deleted automatically. Input data
are useful for short-lived input data like form data.

To store the input data for the next request, you may use the `flashInput` method:

```php
$session->flashInput(['username' => 'JohnDoe']);
```

If you want to get the old input data for a specific key, you may use the `getOldInput` method. If the old input data
for the given key does not exist, the `getOldInput` method will return `null` as the default value:

```php
$session->getOldInput('username');
```

You may pass a default value as the second argument to the `getOldInput` method. This value will be returned if the old
input data for the given key does not exist:

```php
$session->getOldInput('username', 'default');
```

To determine if the session has old input data, you may use the `hasOldInput` method:

```php
$session->hasOldInput('username');
```

> [!NOTE]
> The flash input data are stored in the session data under the `_input` key. The `_input` key is reserved and should
> not be used for other session data!

### Session handler

To get the instance of the provided session handler, you may use the `getHandler` method:

```php
$session->getHandler();
```

### Encrypt session data

The session repository also allows you to encrypt the session data. To encrypt the session data, you have to pass an
instance of the `Zaphyr\Encrypt\Encrypt` class to the constructor of the `Zaphyr\Session\EncryptedSession` instance.
The constructor of the `Zaphyr\Encrypt\Encrypt` class expects the encryption key and the encryption algorithm as
arguments.
You can read more about the encryption in the [encryption](/docs/repositories/latest/encrypt) repository. After you
have configured the encryption, you can use the `Zaphyr\Session\EncryptedSession` instance as usual:

```php
$encryptor = new Zaphyr\Encrypt\Encrypt('OOQPAgC4tA7NanCiVCa1QN5BiRDpdQZR', 'AES-256-CBC');

$handler = new Zaphyr\Session\Handler\FileHandler('/path/to/storage');

$session = new Zaphyr\Session\EncryptedSession('encrypted_session', $handler, $encryptor);
```

## Session handlers

The session handlers define how the session data is stored and retrieved. The session repository comes shipped with a
file and database session handler. You can also create your [custom session handler](#session-handler)
if you want to store the session data in a custom way.

### File session handler

The `Zaphyr\Session\Handler\FileHandler` stores the session data in a file on the server. Pass the path to the storage,
where the session data should be stored to the constructor, and you're good to go:

```php
$handler = new Zaphyr\Session\Handler\FileHandler('/path/to/storage');
```

> [!NOTE]
> The file handler does not create the storage directory automatically. You have to create the directory manually.
> Also, the storage directory must be writable by the web server!

#### Configuring the file handler session lifetime

By default, the file handler will set the lifetime of the session to **60 minutes**. However, you can set the lifetime
to a different value by passing the `minutes` parameter to the constructor:

```php
$handler = new Zaphyr\Session\Handler\FileHandler('/path/to/storage', minutes: 120);
```

### Database session handler

The `Zaphyr\Session\Handler\DatabaseHandler` stores the session data in a database. The database handler fits perfect,
if you serve your application on multiple servers, and want to share the session data between the servers.

When using the database handler, you will need to create a table to store the session data. Below is an example of a
table that can be used to store the session data in a MySQL database:

```sql
CREATE TABLE `sessions`
(
    `id`   VARBINARY(128) NOT NULL PRIMARY KEY,
    `data` TEXT NOT NULL,
    `time` INTEGER UNSIGNED NOT NULL
) COLLATE utf8mb4_bin, ENGINE = InnoDB;
```

The database handler requires a `Doctrine\DBAL\Connection` instance, which is used to store and retrieve the session
data. You can create a connection instance by using the `Doctrine\DBAL\DriverManager` class:

```php
$connectionParams = [
    'dbname' => 'mydb',
    'user' => 'user',
    'password' => 'secret',
    'host' => 'localhost',
    'driver' => 'pdo_mysql',
];

$connection = Doctrine\DBAL\DriverManager::getConnection($connectionParams);

$handler = new Zaphyr\Session\Handler\DatabaseHandler($connection);
```

You can read more about doctrine database connections in the
[Doctrine documentation](https://www.doctrine-project.org/projects/doctrine-dbal/en/3.7/reference/configuration.html).

#### Configuring the database table and column names

The table used to store the session data is called `sessions` by default and defines certain column names. However, you
can change the table and column names by passing an `dbOptions` array parameter to the constructor:

```php
$dbOptions =  [
    'timeColumn' => 'lifetime',
];

$handler = new Zaphyr\Session\Handler\DatabaseHandler($connection, dbOptions: $dbOptions);
```

These are the parameters that you can configure:

- `table`: The name of the table where to store the session data (default is `sessions`).
- `idColumn`: The name of the field where to store the session ID (default is `id`).
- `dataColumn`: The name of the field where to store the session data (default is `data`).
- `timeColumn`: The name of the field where to store the session lifetime (default is `time`).

#### Configuring the database handler session lifetime

By default, the database handler will set the lifetime of the session to **60 minutes**. However, you can set the
lifetime to a different value by passing the `minutes` parameter to the constructor:

```php
$handler = new Zaphyr\Session\Handler\DatabaseHandler($connection, minutes: 120);
```

### Array session handler

<span class="badge__available">Available since v1.1.0</span>

The session repository also comes shipped with an array session handler, which stores the session data in an array and
prevent the session data from being persisted. The array session handler is useful for testing purposes:

```php
$handler = new Zaphyr\Session\Handler\ArrayHandler(minutes: 120);
```

### Custom session handler

If none of the provided session handlers fits your needs, you can create your own custom session handler. To create a
custom session handler, you will need to implement PHP's built in `SessionHandlerInterface`. You do not need to do any
serialization when retrieving or storing session data, as the session repository will perform the serialization for you.

```php
class MyCustomHandler implements SessionHandlerInterface
{
    /**
     * {@inheritdoc}
     */
    public function open(string $path, string $name): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function close(): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function read(string $id): string|false
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function write(string $id, string $data): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function destroy(string $id): bool
    {
        // …
    }

    /**
     * {@inheritdoc}
     */
    public function gc(int $max_lifetime): int|false
    {
        // …
    }
}
```

## Session manager

The session repository comes shipped with a session manager, which makes it easy to create and manage sessions. Instead
of creating a session instance manually, you can use the session manager to create sessions. The session manager also
makes it easy to change the session handler, and to set the session lifetime. The session manager requires a session
name and an array of session handler options. The session name is used to identify the session, and the session handler
options are used to create the session handler. Below is an example of how to create a session manager:

```php
$sessionName = 'my_session';
$handlerOptions = [
    Zaphyr\Session\SessionManager::DATABASE_HANDLER => [
        'connection' => [
            'dbname' => 'mydb',
            'user' => 'user',
            'password' => 'secret',
            'host' => 'localhost',
            'driver' => 'pdo_mysql',
        ],
        //'options' => [
        //    'table' => 'sessions',
        //    'idColumn' => 'id',
        //    'dataColumn' => 'data',
        //    'timeColumn' => 'time',
        //],
    ],
    Zaphyr\Session\SessionManager::FILE_HANDLER => [
        'path' => __DIR__ . '/storage',
    ],
]

$sessionManager = new Zaphyr\Session\SessionManager($sessionName, $handlerOptions);
```

The [database handler](#database-session-handler) requires a `connection` configuration, which is used to create
a database connection. Optionally, you can pass an `options` array to the database handler configuration, which is used
to configure the database table and column names.

The [file handler](#file-session-handler) requires a `path` configuration, which is used to store the session
data. You can read more about the session handler options in the [session handler section](#session-handlers).

### Start a session

After you have created the session manager, you can start a session by calling the `session()` method. The `session()`
method accepts an optional parameter, which specifies the session handler to use. If no session handler is specified,
the session manager will use the default session handler (`Zaphyr\Session\Handler\FileHandler` by default).
The `session()` method will return an instance of `Zaphyr\Session\Contracts\SessionInterface`.

```php
$session = $sessionManager->session();
```

To get a specific session handler, you can pass the session handler name to the `session()` method:

```php
$session = $sessionManager->session(Zaphyr\Session\SessionManager::DATABASE_HANDLER);
```

### Change the default session expiration time

By default, the session manager will set the expiration time to **60 minutes**. If you want to set a different expiration
time, you can pass the `sessionExpireMinutes` parameter to the constructor:

```php
$sessionManager = new Zaphyr\Session\SessionManager($sessionName, $handlerOptions, sessionExpireMinutes: 120);
```

### Change the default session handler

The default session handler is the `Zaphyr\Session\Handler\FileHandler`. However, you can change the default session
handler by passing the `defaultHandler` parameter to the constructor:

```php
$sessionManager = new Zaphyr\Session\SessionManager(
    $sessionName,
    $handlerOptions,
    defaultHandler: Zaphyr\Session\SessionManager::DATABASE_HANDLER
);
```

### Encrypt the session data

To encrypt the session data, you will need to pass an instance of the `Zaphyr\Encrypt\Encrypt` class to the constructor.
Now the session data will be encrypted before it is stored in the session, and decrypted when it is retrieved:

```php
$encryptor = new Zaphyr\Encrypt\Encrypt('OOQPAgC4tA7NanCiVCa1QN5BiRDpdQZR', 'AES-256-CBC');

$sessionManager = new Zaphyr\Session\SessionManager($sessionName, $handlerOptions, encryptor: $encryptor);
```

### Add custom session handler

We already learned how to create a [custom session handler](#custom-session-handler). To use the custom session
handler on the session manager, you will need to add it to the session manager. You can add a custom session handler by
using the `addHandler`. The first parameter is the name of the handler, and the second parameter is a callback that
returns an instance of the custom session handler:

```php
$sessionManager->addHandler('custom_handler', fn () => new MyCustomHandler());
```

After you have added the custom session handler, you can use it by passing the name of the handler to the `session`
method:

```php
$sessionManager->session('custom_handler');
```
