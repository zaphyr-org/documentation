# Session

The framework provides a robust and flexible session management system that allows you to store user data across
requests. Sessions are useful for maintaining state, such as user authentication, preferences, and other temporary data.
The framework's session implementation is built on top of the [Session Repository](/docs/repositories/latest/session)
and provides a simple and intuitive API for managing session data.

## Configuration

You can configure session behavior in your application by editing the `config/session.yaml` file and `.env` file. These
settings allow you to define how sessions are handled, stored, and secured.

### Default Session Handler

The session handler determines where and how session data is stored. By default sessions are stored in files, but you
can configure other supported handlers such as database drivers if needed.

Set the default session handler using the `SESSION_DEFAULT_HANDLER` environment variable in your `.env` file:

```bash
SESSION_DEFAULT_HANDLER=file
```

Availabale handlers include:

- `file`: Stores session data in files on the server.
- `database`: Stores session data in a database table (if configured).

Read more about the available session handlers in the
[Session Handlers](/docs/repositories/latest/session#session-handlers) documentation.

### Session Lifetime

You can control how long a session remains active by setting the session expiration time (in seconds). After this time,
the session will be considered expired and cleared automatically.

```bash
SESSION_EXPIRE=120
```

In the example above, session data will expire after 120 seconds of inactivity.

### Session Encryption

For security, all session data is encrypted by default before being written to storage. This ensures sensitive data is
protected even if the storage is compromised. You can disable session encryption by setting the `SESSION_ENCRYPT`
environment variable to `false` in your `.env` file:

```bash
SESSION_ENCRYPT=true
```

For more details on how encryption works and how to manage encryption keys, refer to the
[Encryption](/docs/framework/latest/encryption) documentation.

## Usage

Each HTTP request contains a session object that can be accessed through the `Request`object. This
session object allows you to store, retrieve, and manage session data throughout the application.

You can learn more in the [Session Repository](/docs/repositories/latest/session#retrieve-session-data)
documentation, which provides detailed information on how to use the session repository effectively.

### Access the Session

To access the session data in a controller, use the `getSession()` method of the `Zaphyr\Framework\Http\Request` object.
This method returns an instance of the session repository, which provides methods for managing session data:

```php
namespace App\Controllers;

use Zaphyr\Framework\Http\Request;
use Zaphyr\Framework\Http\Response;

Class UserController
{
    public function indexAction(Request $request): Response
    {
        $value = $request->getSession()->get('key');
        
        // …
    }
}
```

#### Retrieve All Session Data

To retrieve all session data currently stored, use the `all()` method. This method returns an associative array of all
session key-value pairs:

```php
$request->getSession()->all('key', 'value');
```

#### Store Session Data

To store a value in the session, use the `set()` method. This will save the value under the specified key and make it
available across subsequent requests:

```php
$request->getSession()->set('key', 'value');
```

#### Check for Session Data

To check if a particular key exists in the session, use the `has()` method. This method returns `true` if the key
exists:

```php
$request->getSession()->has('key');
```

#### Remove Session Data

To remove a specific key and its value from the session, use the `remove()` method:

```php
$request->getSession()->remove('key');
```

### Flash

Flash messages are temporary session messages that are available for the next HTTP request only. They are
typically used to provide user feedback after an action, such as submitting a form or completing a process.

Once retrieved in the next request, the flash mesage is automatically removed from the session.

For more details, refer to the [Flash Messages](/docs/repositories/latest/session#flash-session-data) section
in the session documentation.

#### Flash Session Data

To store a flash message in the session, you can use the `flash()` method. This method accepts a key and a value,
which will be available in the next request:

```php
$request->getSession()->flash('success', 'Your profile has been updated.');
```

#### Retrieve Flash Session Data

To retrieve a flash message, you can use the `get()` method with the key you used to store the message in the `flash()`
method:

```php
if ($request->getSession->has('success')) {
    $message = $request->getSession()->get('success');
}
```

### Flash Input

Flash input allows you to temporarily store user input data in the session so that it can be accessed on the next HTTP
request. This is particularly useful after form validation errors. When a user submits a from with invalid data, the
input can be "flashed" to the session, allowing it to be repopulated in the form on the next request.

You can read more about flash input handling in
the [Session Flash Input](/docs/repositories/latest/session#flash-input-data) documentation.

#### Flash Input Data

To store or "flash" input data to the session, you can use the `flashInput()` method. This method is typically used
after form validation errors to retain the user's input for the next request:

```php
$request->getSession()->flashInput(['name', 'John Doe']);
```

#### Retrieve Flashed Input Data

To retrieve previously flashed input data, use the `getOldInput()` method. This method retrieves the old input data
for a specific key:

```php
$request->getSession()->getOldInput('name');
```

If the key does not exist, it will return `null` by default. You may also provide a default value as the second
argument:

```php
$request->getSession()->getOldInput('name', 'Default Name');
```

#### Check for Old Input Data

To determine whether a given key has been flashed to the session, you can use the `hasOldInput()` method. This method
checks if there is any old input data for the specified key:

```php
$request->getSession()->hasOldInput('name');
```

This is useful for conditionally displaying pre-filled form fields or alerting the user that some values were preserved.
