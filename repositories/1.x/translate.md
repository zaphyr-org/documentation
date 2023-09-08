> [!WARNING]
> You're browsing the documentation for an old version running php 7.x.
> Consider upgrading to [v2.x](/docs/2.x/repositories/translate).

# Translate

_A simple translator to serve your applications in multiple languages._

---

[TOC]

---

## Installation

To get started, install the translate repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/translate
```

---

## Configuration and usage

Sooner or later you might reach the point where you want to translate your web application into different languages
to present it to a wider audience. This translation service offers you the perfect starting point. It supports string
translation, pluralization, fallback languages and various translation file formats (`.ini`, `.json`, `.xml` and `.yaml`).
However, before you start translating your application, you need to configure your `Translator` instance a bit.

Let's say you put your translation strings in a `resources/lang` directory. Then each language needs its own subdirectory:

```bash
+-- resources
|   +-- lang
|   |   +-- en
|   |   |   +-- app.php
|   |   +-- pl
|   |   |   +-- app.php
```

Within your translation files you define an array of keyed translation strings:

```php
// resources/lang/en/app.php
return [
    'welcome' => 'How do you do?',
];

// resources/lang/pl/app.php
return [
    'welcome' => 'Jak się masz?',
];
```

Now we can create a `Zaphyr\Translate\Translator` instance and return our translation strings using the `get()` method:

```php
$directory = 'resources/lang/';
$locale = 'en';

$translator = new Zaphyr\Translate\Translator($directory, $locale);

$translator->get('app.welcome'); // How do you do?
```

> [!NOTE]
> The file name of the translation resources always acts as a "namespace". In our example, the translation files are
> called `app.php`. So if we want to access a translation string from `app.php`, we pass the "namespace" `app` to the
> `get()` method.

### Adding additional translation directories

You can also add additional translation directories to an existing translator instance using the `addDirectory()` method:

```php
$translator->addDirectory('localization/translations');
$translator->hasDirectory('localization/translations'); // true
```

> [!NOTE]
> If you want to add additional translation directories to your `Translator` instance, it is best to do this before
> calling the `get()` method. Otherwise, the `get()` method doesn't even know the translation
> directories that were added in the `addDirectory()` method.

### Set and get locale

It is also possible to define the language using the `setLocale()` method:

```php
$translator->setLocale('pl');
$translator->getLocale(); // pl
```

> [!NOTE]
> If you want to change the locale of your `Translator` instance by using the `setLocale()` method, it is best to do
> this before calling the `get()` method. Otherwise, the `get()` method doesn't even know the changed locale that were
> added in the `setLocale()` method and continues to use the locale set in the constructor of the translator instance.

---

## Fallback locale

Sometimes you may not have fully translated your application in all languages. It makes sense to define a fallback
language, which is used if a language does not yet contain a corresponding translation string:

```php
$directory = 'resources/lang/';
$locale = 'de';
$fallbackLocale = 'en';

$translator = new Zaphyr\Translate\Translator($directory, $locale, $fallbackLocale);
```

### Set and get fallback locale

You can also define the fallback language with the `setFallbackLocale()` method:

```php
$translator->setFallbackLocale('en');
$translator->getFallbackLocale(); // en
```

> [!NOTE]
> If you have set the fallback language using the `setFallbackLocale()` method and want your `get()` methods to consider
> your defined fallback language, you must call the `setFallbackLocale()` method before using the `get()` method.
> Otherwise, the `get()` method does not yet know your defined fallback language and continues to use the fallback
> language set in the constructor of the translator instance.

---

## Translation file reader

Different file formats can be used as translation files. By default, the translator service loads `PHP` files.
Other supported file formats are `INI`, `JSON`, `XML` and `YAML`.

```php
$directory = 'resources/lang/';
$locale = 'de';
$fallbackLocale = 'en';
$reader = Zaphyr\Translate\Translator::READER_JSON;

$translator = new Zaphyr\Translate\Translator($directory, $locale, $fallbackLocale, $reader);
```

### Set and get file reader

You can also use a setter method to define the desired file reader:

```php
$translator->setReader(Zaphyr\Translate\Translator::READER_YAML);
$translator->getReader(); // yaml
```

> [!NOTE]
> If you have set the file reader using the `setReader()` method and want your `get()` methods to use your new defined
> file reader, you must call the `setReader()` method before using the `get()` method. Otherwise, the `get()` method
> does not yet know your desired reader and continues to use the file reader set in the constructor of the
> translator instance.

---

## Replace translation parameters

You can also use placeholders in your translation strings. All placeholders are surrounded by `%`. For example, you
might want to greet visitors of your application:

```php
return [
    'greet' => 'Hello %name%'
];
```

To replace the placeholders in your translation strings, pass an array of replacements as the second parameter to the
`get()` method:

```php
$translator->get('app.greet', ['name' => 'john']); // Hello john
```

If your placeholder consists of capitalized letters, or only the first letter is capitalized,
the translator will capitalize your translation value accordingly:

```php
return [
    'hello' => 'Hello %Name%', // Hello John
    'goodbye' => 'Goodbye %NAME%' // Goodbye JOHN
];
```

---

## Pluralization

Pluralization is a science in itself, as different languages have a wide variety of complex pluralization rules. This
translation service offers you the possibility to map pluralization's for any language. By using the "pipe" character
you can represent singular and plural forms for translation strings:

```php
return [
    'newMessage' => 'You have a new message|You have new messages',
];
```

You also have the possibility to define more complex pluralization rules for several number ranges:

```php
return [
    'tasks' => '{0} No tasks. Chill|[1,19] You have some work to do|[20,*] Now you have to get busy',
];
```

After you have defined a translation string for your desired pluralization options, simple call the `choice()` method
to get the line for a desired plural form:

```php
$translator->choice('app.newMessage', 1); // You have a new message
$translator->choice('app.newMessage', 2); // You have new messages
$translator->choice('app.tasks', 0); // No tasks. Chill
$translator->choice('app.tasks', 5); // You have some work to do
$translator->choice('app.tasks', 25); // Now you have to get busy
```

### Pluralization placeholders

You may also define placeholder attributes in pluralization strings. Pass an array as third parameter to the `choice()`
method and the placeholders will be replaced:

```php
return [
    'minutes' => '{1} %value% minute ago|[2,*] %value% minutes ago',
];

$translator->choice('app.minutes', 1, ['value' => 1]); // 1 minute ago
$translator->choice('app.minutes', 2, ['value' => 2]); // 2 minutes ago
```

### Pluralization count

If you want to represent the integer value passed to the `choice()` method, you can use the `%count%` placeholder.
The `%count%` placeholder can handle integer, arrays and objects implementing the
[Countable interface](https://www.php.net/manual/en/class.countable.php):

```php
return [
    'count' => '{0} There are none|{1} There is one|[2,*] There are %count%'
];
```

#### Count with an integer

If you pass an integer as second parameter to the `choice()` method, the translator will replace the `%count%`
placeholder in your translation string:

```php
$translator->choice('app.count', 2); // There are 2
```

#### Count with an array

If you pass an array as second parameter to the `choice()` method, the translator replaces the `%count%` placeholder
with the amount of the existing array elements:

```php
$translator->choice('app.count', [1, 2, 3]); // There are 3
```

#### Count with an object implementing Countable interface

If you have an object that implements the Countable interface, the translator replaces the `%count%` placeholder with
the value returned by the `count()` method of your object:

```php
$values = new class implements Countable
{
    public function count(): int
    {
        return 5;
    }
};

$translator->choice('app.count', $value); // There are 5
```
