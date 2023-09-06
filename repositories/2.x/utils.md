# Utils

_A collection of useful helper classes, which make a developer's workday a little easier._

---

[TOC]

---

## Installation

To get started, install the utils repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/utils
```

---

## Array

Useful helper methods to work with arrays.

---

### accessible

Determines whether the given value is an array.

```php
Zaphyr\Utils\Arr::accessible(['foo' => ['bar']]); // true
```

----

### exists

Determines whether the given key exists in the provided array.

```php
Zaphyr\Utils\Arr::exists(['foo' => ['bar']], 'foo'); // true
```

----

### set

Sets an array item to a given value using "dot" notation.
If no key is given to the method, the entire array will be replaced.

```php
$array = ['foo' => ['bar']];

Zaphyr\Utils\Arr::set($array, 'foo', 'baz'); // ['foo' => 'baz']
Zaphyr\Utils\Arr::set($array, 'baz', 'qux'); // ['foo' => ['bar'], 'baz' => 'qux']
Zaphyr\Utils\Arr::set($array, 'bar.baz', 'QUX'); // ['foo' => ['bar'], 'bar' => ['baz' => 'QUX']]
Zaphyr\Utils\Arr::set($array, ['bar' => 'BAR', 'baz' => 'BAZ']) // ['bar' => 'BAR', 'baz' => 'BAZ']
```

----

### add

Adds an element to an array using "dot" notation if the given key doesn't exist.
The `add` method will not overwrite existing keys!

```php
$array = ['foo' => ['bar']];

Zaphyr\Utils\Arr::add($array, 'baz', 'qux'); // ['foo' => ['bar'], 'baz' => 'qux']
Zaphyr\Utils\Arr::add($array, 'foo', 'BAR'); // ['foo' => ['bar']]
```

----

### get

Returns an item from an array using "dot" notation.

```php
Zaphyr\Utils\Arr::get(['foo' => ['bar' => 'baz']], 'foo.bar'); // 'baz'
```

The `get()` method also accepts a default value, which will be returned if the specified key is not found.

```php
Zaphyr\Utils\Arr::get(['foo' => ['bar' => 'baz']], 'foo.qux', 'qux');
```

----

### first

Returns the first element in an array passing a given truth test.

```php
Zaphyr\Utils\Arr::first([100, 200, 300], fn ($value, $key) => $value <= 150 )); // 100
```

A default value may also be passed as the third parameter to the method.
This value will be returned if no value passes the truth test.


```php
Zaphyr\Utils\Arr::first([100, 200, 300], fn ($value, $key) => $value <= 50, 100); // 100
```

----

### last

Returns the last element in an array passing a given truth test.

```php
Zaphyr\Utils\Arr::last([100, 200, 300], fn ($value, $key) => $value >= 150); // 300
```

A default value may be passed as the third argument to the method.
This value will be returned if no value passes the truth test.

```php
Zaphyr\Utils\Arr::last([100, 200, 300], fn ($value, $key) => $value >= 400, 400); // 400
```

----

### has

Checks if an item or items exist in an array using "dot" notation.

```php
Zaphyr\Utils\Arr::has(['foo' => ['bar' => 'baz'], 'qux'], 'foo.bar'); // true
```

----

### where

Filters an array using the given callback.

```php
Zaphyr\Utils\Arr::where([100, '200', 300], fn ($value, $key) => is_string($value)); // ['200']
```

----

### only

Returns a subset of the items from the given array.

```php
Zaphyr\Utils\Arr::only(['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'], ['foo', 'quu']); // ['foo' => 'bar', 'quu' => 'qaa']
```

----

### forget

Removes one or many array items from a given array using "dot" notation.

```php
$array = ['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'];

Zaphyr\Utils\Arr::forget($array, ['baz', 'quu']);

$array; // ['foo' => 'bar']
```

----

### except

Returns all the given array elements except for a specified array of keys.

```php
Zaphyr\Utils\Arr::except(['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'], ['foo', 'quu']); // ['baz' => 'qux']
```

----

## ClassFinder

Useful helper methods to work with classes.

---

### getClassesFromDirectory

Returns an array with all classes of a given directory.

```php
Zaphyr\Utils\ClassFinder::getClassesFromDirectory('src'); // […]
```

---

### getClassNameFromFile

> [!WARNING]
> This method will be removed in v3.0. Use [getClassBasename](#getclassbasename) method instead!

Returns the class name of a given file.

```php
Zaphyr\Utils\ClassFinder::getClassNameFromFile('src/Foo.php'); // 'Foo'
```

---

### getClassBasename

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns the class basename of a given string or object.

```php
Zaphyr\Utils\ClassFinder::getClassBasename('src/Foo.php'); // 'Foo'
Zaphyr\Utils\ClassFinder::getClassBasename('Foo\Bar\Baz'); // 'Baz'
Zaphyr\Utils\ClassFinder::getClassBasename(new \stdClass()); // 'stdClass'
```

---

### getNamespaceFromFile

Returns the namespace from a given file.

```php
Zaphyr\Utils\ClassFinder::getNamespaceFromFile('src/Bar.php'); // 'Foo\Bar'
```

----

## Country

Useful helper class for country names and lists.

---

### getNameByIsoCode

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns a country name by its corresponding ISO code.

```php
Zaphyr\Utils\Country::getNameByIsoCode('DE'); // 'Germany'
```

---

### getAllCountries

Returns an array of all available countries with ISO code as key.

```php
Zaphyr\Utils\Country::getAllCountries(); // ['AF' => 'Afghanistan', […] 'ZW' => 'Zimbabwe']
```

> [!NOTE]
> Since v2.0.0 this method returns the ISO code of a country as array key.

---

### getAllCountriesAsJsonString

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns a JSON string of all available countries with ISO code as key.

```php
Zaphyr\Utils\Country::getAllCountriesAsJsonString(); // ‘{'AF': 'Afghanistan', […] 'ZW': 'Zimbabwe'}'
```

---

## Date

Useful helper methods to work with dates and times.

---

### timestamp

Returns the current timestamp or the timestamp of a given date.

```php
Zaphyr\Utils\Date::timestamp(); // time()
Zaphyr\Utils\Date::timestamp(new DateTime('2022-09-13 09:41:00')); // 1663918860
Zaphyr\Utils\Date::timestamp('1663918860'); // 1663918860
Zaphyr\Utils\Date::timestamp('yesterday'); // 'yesterday'
```

---

### timezone

Returns the timezone.

```php
Zaphyr\Utils\Date::timezone(new DateTimeZone('UTC')); // \DateTimezone::class
Zaphyr\Utils\Date::timezone()->getName(); // 'UTC'
Zaphyr\Utils\Date::timezone('GMT')->getName(); // 'GMT'
```

---

### factory

Returns a DateTime object.

```php
Zaphyr\Utils\Date::factory(); // \DateTime::class
Zaphyr\Utils\Date::factory('2022-09-13 09:41:00'); // \DateTime::class
```

---

### sqlFormat

Return a SQL compliant date format.

```php
Zaphyr\Utils\Date::sqlFormat(); // 'Y-m-d H:i:s'
Zaphyr\Utils\Date::sqlFormat(1663918860); // '2022-09-13 09:41:00'
Zaphyr\Utils\Date::sqlFormat('2022-09-13'); // '2022-09-13 00:00:00'
```

---

### humanReadable

Returns a human-readable date format.

```php
Zaphyr\Utils\Date::humanReadable(1663918860); // '23 Sep 2022 09:41'
Zaphyr\Utils\Date::humanReadable('2022-09-23 09:41:00', 'd F Y'); // '23 September 2022'
```

---

### isValid

Determines whether a given date is valid.

```php
Zaphyr\Utils\Date::isValid('1663918860'); // true
Zaphyr\Utils\Date::isValid('now'); // true
Zaphyr\Utils\Date::isValid('2022-09-23 09:41:00'); // true
Zaphyr\Utils\Date::isValid(''); // false
```

---

### isToday

Determines whether a given date is today.

```php
Zaphyr\Utils\Date::isToday('+0 day'); // true
```

---

### isTomorrow

Determines whether a given date is tomorrow.

```php
Zaphyr\Utils\Date::isTomorrow('+1 day'); // true
Zaphyr\Utils\Date::isTomorrow('+0 day'); // false
```

---

### isThisWeek

Determines whether a given date is this week.

```php
Zaphyr\Utils\Date::isThisWeek('+0 week'); // ture
Zaphyr\Utils\Date::isThisWeek('+1 week'); // false
```

---

### isThisMonth

Determines whether a given date is this month.

```php
Zaphyr\Utils\Date::isThisMonth('+0 month'); // true
Zaphyr\Utils\Date::isThisMonth('+1 month') // false
```

---

### isThisYear

Determines whether a given date is this year.

```php
Zaphyr\Utils\Date::isThisYear('+0 year'); // true
Zaphyr\Utils\Date::isThisYear('+1 year'); // false
```

---

## File

Useful helper methods to work with files and folders.

---

### exists

> [!WARNING]
> This method will be removed in v3.0. Use PHP built-in function "file_exists" instead!

Determines whether a file or directory exists.

```php
Zaphyr\Utils\File::exists(__FILE__); // true
Zaphyr\Utils\File::exists(__DIR__); // true
```

---

### isFile

> [!WARNING]
> This method will be removed in v3.0. Use PHP built-in function "is_file" instead!

Determines whether a file exists.

```php
Zaphyr\Utils\File::isFile(__FILE__); // true
Zaphyr\Utils\File::isFile(__DIR__); // false
```

---

### isDirectory

> [!WARNING]
> This method will be removed in v3.0. Use PHP built-in function "is_dir" instead!

Determines whether a directory exists.

```php
Zaphyr\Utils\File::isDir(__DIR__); // true
Zaphyr\Utils\File::isDir(__FILE__); // false
```

---

### info

Returns a path information of a given file or directory.

```php
Zaphyr\Utils\File::info(__FILE__);
// [
//      'dirname' => '…',
//      'basename' => '…',
//      'extension'  => '…',
//      'filename' => '…',
// ]
```
This method also accepts a second parameter to return a specific path information.
Any valid `PATHINFO_*` constant can be used:

```php
Zaphyr\Utils\File::info(__FILE__, PATHINFO_EXTENSION);
```

---

### name

Returns the name of a file or directory path.

```php
Zaphyr\Utils\File::name(__FILE__);
Zaphyr\Utils\File::name(__DIR__);
```

---

### basename

Returns the basename of a file or directory path.

```php
Zaphyr\Utils\File::basename(__FILE__);
Zaphyr\Utils\File::basename(__DIR__);
```

---

### dirname

Returns the directory name of a file or directory.

```php
Zaphyr\Utils\File::dirname(__FILE__);
Zaphyr\Utils\File::dirname(__DIR__);
```

---

### extension

Returns the file extension of a given file.

```php
Zaphyr\Utils\File::extension(__FILE__);
```

---

### type

Returns the type of file or directory.

```php
Zaphyr\Utils\File::type(__FILE__); // 'file'
Zaphyr\Utils\File::type(__DIR__); // 'dir'
```

---

### mimeType

Returns the mime type of file or directory.

```php
Zaphyr\Utils\File::mimeType(__FILE__); // 'text/x-php'
Zaphyr\Utils\File::mimeType(__DIR__); // 'directory'
```

---

### size

Returns the size of a file or directory as an integer.

```php
Zaphyr\Utils\File::size(__FILE__);
Zaphyr\Utils\File::size(__DIR__);
```

---

### hash

Calculates the md5 hash of a given file.

```php
Zaphyr\Utils\File::hash(__FILE__);
```

---

### chmod

Changes the file mode.

```php
// set chmod
Zaphyr\Utils\File::chmod(__DIR__, '0775'); // bool

// get chmod
Zaphyr\Utils\File::chmod(__DIR__); // '0775'
```

---

### lastModified

Returns the file modification time as an integer.

```php
Zaphyr\Utils\File::lastModified(__FILE__);
Zaphyr\Utils\File::lastModified(__DIR__);
```

---

### isReadable

Determines whether a file or directory is readable.

```php
Zaphyr\Utils\File::isReadable(__FILE__); // true
Zaphyr\Utils\File::isReadable(__DIR__); // true
```

---

### isWritable

Determines whether a file or directory is writable.

```php
Zaphyr\Utils\File::isWritable(__FILE__); // true
Zaphyr\Utils\File::isWritable(__DIR__); // true
```

---

### glob

Finds all pathname of a matching pattern.

```php
Zaphyr\Utils\File::glob(__DIR__ . '/*.*'); // […]
```

You can also pass a second parameter to specify the flags. Any valid `GLOB_*` constant can be used.

```php
Zaphyr\Utils\File::glob(__DIR__ . '/*.*', GLOB_ONLYDIR); // […]
```

---

### files

Returns all files in a given directory.

```php
Zaphyr\Utils\File::files(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::files('non-existing-dir'); // null

// with hidden files
Zaphyr\Utils\Files::files(__DIR__, true); // SplFileInfo[]
```

---

### allFiles

Returns all files in a given directory including all files in subdirectories.

```php
Zaphyr\Utils\File::allFiles(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::allFiles('non-existing-dir'); // null

// with hidden files
Zaphyr\Utils\File::allFiles(__DIR__, true); // SplFileInfo[]
```

---

### directories

Returns all directories inside a given directory.

```php
Zaphyr\Utils\File::directories(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::directories('non-existing-dir'); // null

// with hidden directories
Zaphyr\Utils\File::directories(__DIR__, true); // SplFileInfo[]
```

---

### getRequire

Requires a file.

```php
try {
    Zaphyr\Utils\File::getRequire(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

---

### getRequireOnce

Requires a file once.

```php
try {
    Zaphyr\Utils\File::getRequireOnce(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

---

### read

Reads the contents of a file.

```php
try {
    Zaphyr\Utils\File::read(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

---

### put

Puts content inside a file.

```php
Zaphyr\Utils\File::put(__FILE__, 'contents');
```

It is also possible to acquire a lock on the file while writing to it. Simply pass `true` as the third parameter.

```php
Zaphyr\Utils\File::put(__FILE__, 'contents', true);
```

---

### replace

Replaces content inside a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::replace(__FILE__, 'contents');
```

---

### prepend

Prepends content to a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::prepend(__FILE__, 'contents');
```

---

### append

Appends content to a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::append(__FILE__, 'contents');
```

---

### delete

Deletes a given file.

```php
Zaphyr\Utils\File::delete('file'); // true
Zaphyr\Utils\File::delete(['file1', 'file2']); // true
```

---

### move

Moves a file to a given directory.

```php
Zaphyr\Utils\File::move('source-file', 'destination-target'); // true
```

---

### copy

Copies a file to a given directory.

```php
Zaphyr\Utils\File::copy('source-file', 'destination-target'); // true
```

---

### createDirectory

Creates a new directory.

```php
$directory = 'dirname';
$mode = 0755;
$recursive = false;
$force = false;

Zaphyr\Utils\File::createDirectory($directory, $mode, $recursive, $force);
```

---

### deleteDirectory

Deletes an existing directory.

```php
$directory = 'directory'
$preserve = false;

Zaphyr\Utils\File::deleteDirectory($directory, $preserve); // true
```

---

### cleanDirectory

Removes all files and directories inside a given directory.

```php
Zaphyr\Utils\File::cleanDirectory('directory'); // true
```

---

### moveDirectory

Moves a directory to a given path.

```php
Zaphyr\Utils\File::moveDirectory('from', 'to');
```

---

### copyDirectory

Copies a directory to a given path.

```php
$options = \FilesystemIterator::SKIP_DOTS;

Zaphyr\Utils\File::copyDirectory('source-directory', 'target-destination-directory', $options);
```

---

## Filter

Useful helper methods to filter for occurrences.

---

### alpha

Returns only the alpha chars of a string.

```php
Zaphyr\Utils\Filter::alpha('fo-0-o'); // 'foo'
```

The `alpha()` method also accepts array of strings.

```php
Zaphyr\Utils\Filter::alpha(['fo-0-o', '123asd']); // ['foo', 'asd']
```

---

### alphanum

Returns only the alphanum chars of a string.

```php
Zaphyr\Utils\Filter::alphanum('foo-* 123'); // 'foo123'
```

This method also accepts array of strings.

```php
Zaphyr\Utils\Filter::alphanum(['foo-* 123']); // ['foo123']
```

---

### base64

Returns only the base64 chars of a string.

```php
Zaphyr\Utils\Filter::base64('a8O2bGph*c2do ZHZiYX () Nua2zDtmzDpMOkYQ=='); // 'a8O2bGphc2doZHZiYXNua2zDtmzDpMOkYQ=='
```

---

### digits

Returns only digits of the given string.

```php
Zaphyr\Utils\Filter::digits('f1o2o3'); // 123
```

---

### float

Smart converts any string to float with round.

```php
Zaphyr\Utils\Filter::float('1.5123', 2); // 1.51
Zaphyr\Utils\Filter::float('- 1 0'); // -10.0
```

---

### int

Smart converts any string to int.

```php
Zaphyr\Utils\Filter::int('+3'); // 3
Zaphyr\Utils\Filter::int('- 1 0'); // -10
```

---

## Form

The Form class contains useful helper to rapidly built HTML forms.

---

### open

The `open()` method generates an opening `<form>` tag. By default, the method will generate a form that submits via the
`POST` method.

```php
Zaphyr\Utils\Form::open();
// <form accept-charset="UTF-8" method="POST">
```

To change the submit method, you may pass the desired method as the first argument to the method.

```php
Zaphyr\Utils\Form::open(['method' => 'get']);
// <form accept-charset="UTF-8" method="GET">
```

You may also pass an `action`, `accept-charset`, `class` or `id` to the open method.

```php
Zaphyr\Utils\Form::open([
    'method' => 'get',
    'action' => 'https://localhost/foo',
    'accept-charset' => 'UTF-16',
    'class' => 'form',
    'id' => 'id-form'
]);
// <form accept-charset="UTF-16" method="GET" action="https://localhost/foo" class="form" id="id-form">
```

If your form is going to accept file uploads, you must set the `files` option to `true`.

```php
Zaphyr\Utils\Form::open(['files' => true]);
// <form accept-charset="UTF-8" method="POST" enctype="multipart/form-data">
```

HTML forms do not support `PUT`, `PATCH`, or `DELETE` actions. So, when defining `PUT`, `PATCH`, or `DELETE` routes the
open method will automatically change the method to `POST` and add a hidden `_method` field to the form.

```php
Zaphyr\Utils\Form::open(['method' => 'PUT']);
// <form accept-charset="UTF-8" method="POST"><input name="_method" type="hidden" value="PUT">
```

It is also possible to pass custom attributes to the opening form tag.

```php
Zaphyr\Utils\Form::open(['data-foo' => 'bar'])
// <form accept-charset="UTF-8" method="POST" data-foo="bar">
```

---

### close

To close a form, you may use the `close()` method.

```php
Zaphyr\Utils\Form::close();
// </form>
```

---

### label

Returns a label for an input field. Keep in mind that after creating a label, any form element you create with a name
matching the label name will automatically receive an `id` attribute matching the label name.

```php
Zaphyr\Utils\Form::label('foo');
// <label for="foo">Foo</label>
```

By default, the label method will generate a label with the text matching the input name. If you want to customize the
label text, you may pass the label text as the second argument to the method.

```php
Zaphyr\Utils\Form::label('foo', 'Bar');
// <label for="foo">Bar</label>
```

To customize the attributes of the label, you may pass an array of attributes as the third argument to the method.

```php
Zaphyr\Utils\Form::label('foo', 'Bar', ['id' => 'baz', 'data-id' => 'test']);
// <label for="foo" id="baz" data-id="test">Bar</label>
```

The label method will strip HTML tags from the label text. If you wish to include HTML tags in the label text, you may
pass `false` as the fourth argument to the method.

```php
Zaphyr\Utils\Form::label('foo', '<span>Bar</span>');
// <label for="foo">&lt;span&gt;Bar&lt;/span&gt;</label>

Zaphyr\Utils\Form::label('foo', '<span>Bar</span>', [], false);
// <label for="foo"><span>Bar</span></label>
```

---

### input

The `input()` method generates an `<input>` element of the given type.

```php
Zaphyr\Utils\Form::input('text', 'name');
// <input name="name" type="text">

Zaphyr\Utils\Form::input('text', 'name[givenname]');
// <input name="name[givenname]" type="text">
```

It is also possible to pass a default value as the third argument to the method.

```php
Zaphyr\Utils\Form::input('text', 'name', 'John Doe');
// <input name="name" type="text" value="John Doe">
```

You may also pass an array of custom HTML attributes as the fourth argument to the method.

```php
Zaphyr\Utils\Form::input('text', 'name', null, ['class' => 'input']);
// <input class="input" name="name" type="text">
```

---

### text

Returns a `text` input field.

```php
Zaphyr\Utils\Form::text('name');
// <input name="name" type="text">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
echo Zaphyr\Utils\Form::text('name', 'John Doe', ['class' => 'input']);
// <input class="input" name="name" type="text" value="John Doe">
```

---

### password

Returns a `password` input field.

```php
Zaphyr\Utils\Form::password('secret');
// <input name="secret" type="password" value="">
```

The method also accepts an array of custom HTML attributes as the second argument.

```php
echo Zaphyr\Utils\Form::password('secret', ['class' => 'input']);
// <input class="input" name="secret" type="password" value="">
```

---

### range

Returns a `range` input field.

```php
Zaphyr\Utils\Form::range('volume');
// <input name="volume" type="range">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::range('volume', 10, ['class' => 'input']);
// <input class="input" name="volume" type="range" value="10">
```

---

### hidden

Returns a `hidden` input field.

```php
Zaphyr\Utils\Form::hidden('user-id');
// <input name="user-id" type="hidden">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::hidden('user-id', '123', ['class' => 'input']);
// <input class="input" name="user-id" type="hidden" value="123">
```

---

### search

Returns a `search` input field.

```php
Zaphyr\Utils\Form::search('query');
// <input name="query" type="search">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::search('query', 'one', ['class' => 'input']);
// <input class="input" name="query" type="search" value="one">
```

---

### email

Returns an `email` input field.

```php
Zaphyr\Utils\Form::email('mail');
// <input name="mail" type="email">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::email('mail', 'john@doe.com', ['class' => 'input']);
// <input class="input" name="mail" type="email" value="john@doe.com">
```

---

### tel

Returns a `tel` input field.

```php
Zaphyr\Utils\Form::tel('phone-number');
// <input name="phone-number" type="tel">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::tel('phone-number', '1234', ['class' => 'input']);
// <input class="input" name="phone-number" type="tel" value="1234">
```

---

### number

Returns a `number` input field.

```php
Zaphyr\Utils\Form::number('pin');
// <input name="pin" type="number">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the
third argument.

```php
Zaphyr\Utils\Form::Number('pin', '1234', ['class' => 'input']);
// <input class="input" name="pin" type="number" value="1234">
```

---

### date

Returns a `date` input field.

```php
Zaphyr\Utils\Form::date('birthdate');
// <input name="birthdate" type="date">
```

The date method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::date('birthdate', '2022-09-23');
// <input name="birthdate" type="date" value="2022-09-23">

Zaphyr\Utils\Form::date('birthdate', new DateTime('2022-09-23'), ['class' => 'input']);
// <input class="input" name="birthdate" type="date" value="2022-09-23">
```

---

### datetime

Returns a `datetime` input field.

```php
Zaphyr\Utils\Form::datetime('birthdate');
// <input name="birthdate" type="datetime">
```

The datetime method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::datetime('birthdate', '2022-09-23T09:41:00+00:00');
// <input name="birthdate" type="datetime" value="2022-09-23T09:41:00+00:00">

Zaphyr\Utils\Form::datetime('birthdate', new DateTime('2022-09-23'), ['class' => 'input']);
// <input class="input" name="birthdate" type="datetime" value="2022-09-23T00:00:00+00:00">
```

---

### datetimelocal

Returns a `datetime-local` input field.

```php
Zaphyr\Utils\Form::datetimeLocal('birthdate');
// <input name="birthdate" type="datetime-local">
```

The datetimeLocal method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::datetimeLocal('birthdate', '2022-09-23T09:41');
// <input name="birthdate" type="datetime-local" value="2022-09-23T09:41">

Zaphyr\Utils\Form::datetimeLocal('birthdate', new DateTime('2022-09-23'), ['class' => 'input']);
// <input class="input" name="birthdate" type="datetime-local" value="2022-09-23T00:00">
```

---

### time

Returns a `time` input field.

```php
Zaphyr\Utils\Form::time('start-time');
// <input name="start-time" type="time">
```

The time method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::time('start-time', '05:04');
// <input name="start-time" type="time" value="05:04">

Zaphyr\Utils\Form::time('start-time', new DateTime('15:00'), ['class' => 'input']);
// <input class="input" name="start-time" type="time" value="15:00">
```

---

### week

Returns a `week` input field.

```php
Zaphyr\Utils\Form::week('start');
// <input name="start" type="week">
```

The week method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::week('start', '2019-W12');
// <input name="start" type="week" value="2019-W12">

Zaphyr\Utils\Form::week('start', new DateTime('2019-W32'), ['class' => 'input']);
// <input class="input" name="start" type="week" value="2019-W32">
```

---

### month

Returns a `month` input field.

```php
Zaphyr\Utils\Form::month('start');
// <input name="start" type="month">
```

The month method accepts string dates or `DateTime` objects as the default value.

```php
Zaphyr\Utils\Form::month('start', '2019-03');
// <input name="start" type="month" value="2019-03">

Zaphyr\Utils\Form::month('start', new DateTime('2019-09'), ['class' => 'input']);
// <input class="input" name="start" type="month" value="2019-09">
```

---

### url

Returns a `url` input field.

```php
Zaphyr\Utils\Form::url('website');
// <input name="website" type="url">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the third argument.

```php
Zaphyr\Utils\Form::url('website', 'https://localhost', ['class' => 'input']);
// <input class="input" name="website" type="url" value="https://localhost">
```

---

### file

Returns a `file` input field.

```php
Zaphyr\Utils\Form::file('data');
// <input name="data" type="file">
```

The method also accepts an array of custom HTML attributes as the second argument.

```php
Zaphyr\Utils\Form::file('data', ['class' => 'input']);
// <input class="input" name="data" type="file">
```

---

### image

Returns a `image` input field.

```php
Zaphyr\Utils\Form::image('avatar', 'https://localhost/foo.jpg');
// <input src="https://localhost/foo.jpg" name="avatar" type="image">
```

The method also accepts an array of custom HTML attributes as the third argument.

```php
Zaphyr\Utils\Form::image('avatar', 'https://localhost/foo.jpg', ['class' => 'input']);
// <input class="input" src="https://localhost/foo.jpg" name="avatar" type="image">
```

---

### color

Returns a `color` input field.

```php
Zaphyr\Utils\Form::color('background');
// <input name="background" type="color">
```

The method also accepts a default value as the second argument as well as an array of custom HTML attributes as the third argument.

```php
Zaphyr\Utils\Form::color('background', '#ffffff', ['class' => 'input']);
// <input class="input" name="background" type="color" value="#ffffff">
```

---

### reset

Returns a `reset` field.

```php
Zaphyr\Utils\Form::reset('Reset form');
// <input type="reset" value="Reset form">
```

The method also accepts an array of custom HTML attributes as the second argument.

```php
Zaphyr\Utils\Form::reset('Reset form', ['class' => 'input']);
// <input class="input" type="reset" value="Reset form">
```

---

### submit

Returns a `submit` field.

```php
Zaphyr\Utils\Form::submit('Submit form');
// <input type="submit" value="Submit form">
```

The method also accepts an array of custom HTML attributes as the second argument.

```php
Zaphyr\Utils\Form::submit('Submit form', ['class' => 'input']);
// <input class="input" type="submit" value="Submit form">
```

---

### button

Returns a button field.

```php
Zaphyr\Utils\Form::button('Submit');
// <button type="button">Submit</button>
```

The method also accepts an array of custom HTML attributes as the second argument.

```php
Zaphyr\Utils\Form::button('Submit', ['class' => 'input', 'type' => 'submit']);
// <button class="input" type="submit">Submit</button>
```

---

### checkbox

Returns a checkbox field.

```php
Zaphyr\Utils\Form::checkbox('terms-conditions');
// <input name="terms-conditions" type="checkbox" value="1">
```

Generate a checkbox with custom attributes, a default value, and a checked state.

```php
Zaphyr\Utils\Form::checkbox('terms-conditions', 'accept', true, ['class' => ['input']]);
// <input class="input" checked="checked" name="terms-conditions" type="checkbox" value="accept">
```

---

### radio

Returns a radio field.

```php
Zaphyr\Utils\Form::radio('terms-conditions');
// <input name="terms-conditions" type="radio" value="terms-conditions">
```

Generate a radio field with custom attributes, a default value, and a checked state.

```php
Zaphyr\Utils\Form::radio('terms-conditions', 'accepted', true, ['class' => ['input']]);
// <input class="input" checked="checked" name="terms-conditions" type="radio" value="accepted">
```

---

### textarea

Returns a textarea field.

```php
Zaphyr\Utils\Form::textarea('message');
// <textarea name="message" cols="50" rows="10"></textarea>

Zaphyr\Utils\Form::textarea('message', 'Hello world');
// <textarea name="message" cols="50" rows="10">Hello world</textarea>
```

It is also possible to pass an array of custom HTML attributes as the third argument.

```php
Zaphyr\Utils\Form::textarea('message', null, ['class' => 'input']);
// <textarea class="input" name="message" cols="50" rows="10"></textarea>
```

By default, a textarea will be 50 columns wide and 10 rows tall. You may override these defaults by passing an array
of options to the method. The `size`, or `cols` and `rows` keys will be used to set the size of the textarea.

```php
Zaphyr\Utils\Form::textarea('message', null, ['size' => '60x20']);
// <textarea name="message" cols="60" rows="20"></textarea>

Zaphyr\Utils\Form::textarea('message', null, ['rows' => 20, 'cols' => 60]);
<textarea rows="20" cols="60" name="message">
```

By default, the textarea will be escaped. To disable escaping, pass `false` as the fourth argument.

```php
Zaphyr\Utils\Form::textarea('foo', '<span>Bar</span>');
// '<textarea name="foo" cols="50" rows="10">&lt;span&gt;Bar&lt;/span&gt;</textarea>'

Zaphyr\Utils\Form::textarea('foo', '<span>Bar</span>', [], false);
// '<textarea name="foo" cols="50" rows="10"><span>Bar</span></textarea>'
```

---

### select

Returns a drop-down list.

```php
Zaphyr\Utils\Form::select('gender', ['Male', 'Female']);
// <select name="gender">
//      <option value="0">Male</option>
//      <option value="1">Female</option>
// </select>
```

Returns a drop-down list with a selected value.

```php
Zaphyr\Utils\Form::select(
    name: 'gender',
    list: ['m' => 'Male', 'f' => 'Female'],
    selected: 'm'
);
// <select name="gender">
//      <option value="m" selected="selected">Male</option>
//      <option value="f">Female</option>
// </select>
```

Returns a drop-down list that allows multiple selections.

```php
Zaphyr\Utils\Form::select(
    name: 'gender',
    list: ['m' => 'Male', 'f' => 'Female'],
    selectAttributes: ['multiple']
);
// <select multiple name="gender">
//      <option value="m">Male</option>
//      <option value="f">Female</option>
// </select>
```

Returns a dropdown list with additional HTML attributes.

```php
Zaphyr\Utils\Form::select(
    name: 'gender',
    list: ['m' => 'Male', 'f' => 'Female'],
    selectAttributes: ['class' => 'form-class', 'id' => 'form-id']
);
// <select class="form-class" id="form-id" name="gender">
//      <option value="m">Male</option>
//      <option value="f">Female</option>
// </select>
```

Returns a drop-down list with `<optgroup>` tags.

```php
Zaphyr\Utils\Form::select(
    name: 'size',
    list: [
        'Large sizes' => [
            'l' => 'Large',
            'xl' => 'Extra Large'
        ],
        's' => 'Small'
    ]
);
// <select name="size">
//      <optgroup label="Large sizes">
//          <option value="l">Large</option>
//          <option value="xl">Extra Large</option>
//      </optgroup>
//      <option value="s">Small</option>
// </select>
```
Returns a drop-down list with `<optgroup>` tags and disabled options.

```php
Zaphyr\Utils\Form::select(
    name: 'size',
    list: [
        'Large sizes' => [
            'l' => 'Large',
            'xl' => 'Extra Large'
        ],
        'm' => 'Medium',
        'Small sizes' => [
            's' => 'Small',
            'xs' => 'Extra Small'
        ],
    ],
    optionsAttributes: [
        'Large sizes' => [
            'l' => ['disabled']
        ],
        'm' => ['disabled']
    ],
    optgroupsAttributes: [
        'Small sizes' => ['disabled']
    ]
);
// <select name="size">
//      <optgroup label="Large sizes">
//      <option value="l" disabled>Large</option>
//          <option value="xl">Extra Large</option>
//      </optgroup>
//      <option value="m" disabled>Medium</option>
//      <optgroup label="Small sizes" disabled>
//          <option value="s">Small</option>
//          <option value="xs">Extra Small</option>
//      </optgroup>
// </select>
```

Returns a drop-down list with custom option attributes.

```php
Zaphyr\Utils\Form::select(
    name: 'gender',
    list: ['m' => 'Male', 'f' => 'Female'],
    optionsAttributes: ['m' => ['data-foo' => 'foo', 'disabled']]
);
// <select name="gender">
//      <option value="m" data-foo="foo" disabled>Male</option>
//      <option value="f">Female</option>
//  </select>
```

Returns a drop-down list with a placeholder.

```php
echo Zaphyr\Utils\Form::select(
    name: 'avc',
    list: [1 => 'Yes', 0 => 'No'],
    selectAttributes: ['placeholder' => 'Choose']
);
// <select name="avc">
//      <option disabled="disabled" selected="selected">Choose</option>
//      <option value="1">Yes</option>
//      <option value="0">No</option>
// </select>
```

---

### selectRange

Returns a drop-down list with a selected range.

```php
Zaphyr\Utils\Form::selectRange(
    name: 'year',
    begin: 2022,
    end: 2024
);
// <select name="year">
//      <option value="2022">2022</option>
//      <option value="2023">2023</option>
//      <option value="2024">2024</option>
//  </select>
```

Returns a drop-down list with a selected range and a selected value.

```php
Zaphyr\Utils\Form::selectRange(
    name: 'year',
    begin: 2022,
    end: 2024,
    selected: 2023
);
// <select name="year">
//      <option value="2022">2022</option>
//      <option value="2024" selected="selected">2024</option>
//      <option value="2023">2023</option>
//  </select>
```

Returns a drop-down list with a selected range and additional HTML attributes.

```php
Zaphyr\Utils\Form::selectRange(
    name: 'year',
    begin: 2022,
    end: 2024,
    selectAttributes: ['class' => 'form-class'],
    optionsAttributes: ['2022' => ['data-foo' => 'foo', 'disabled']]
);
// <select class="form-class" name="year">
//      <option value="2022" data-foo="foo" disabled>2022</option>
//      <option value="2023">2023</option>
//      <option value="2024">2024</option>
// </select>
```

---

### datalist

Returns a datalist field.

```php
Zaphyr\Utils\Form::text(name: 'gender', options: ['list' => 'genders']);
Zaphyr\Utils\Form::datalist('genders', ['male', 'female']);
// <input list="genders" name="gender" type="text">
// <datalist id="genders">
//      <option value="male">male</option>
//      <option value="female">female</option>
//  </datalist>
```

---

## HTML

Contains a few HTML attributes methods.

---

### attributes

Builds an HTML attribute string from an array.

```php
Zaphyr\Utils\HTML::attributes(['id' => 'foo', 'class' => ['bar', 'baz']]); // ' id="foo" class="bar baz"'
```

---

### attributeElements

Builds a single HTML attribute element.

```php
Zaphyr\Utils\HTML::attributeElement('id', 'foo'); // 'id="foo"'
Zaphyr\Utils\HTML::attributeElement('class', ['bar', 'baz']); // 'class="bar baz"'
```

---

## Math

Useful helper methods for calculating.

---

### round

Rounds a given number up/down by a given precision.

```php
Zaphyr\Utils\Math::round(1.614); // 1.62

$precision = 1;
$roundDown = Zaphyr\Utils\Math::ROUND_DOWN;
Zaphyr\Utils\Math::round(1.61, $precision, $roundDown); // 1.6
```

---

### average

Returns the average of a given number array.

```php
Zaphyr\Utils\Math::average([1, 3, 5]); // 3.0

$precision = 1;
$roundDown = Zaphyr\Utils\Math::ROUND_DOWN;
Zaphyr\Utils\Math::average([1.2, 3.4, 5.6], $precision, $roundDown); // 3.4
```

---

### percentage

Returns the percentage of a given number.

```php
$percentage = 40;
$total = 20;
Zaphyr\Utils\Math::percentage($percentage, $total); // 8

$precision = 1;
$roundDown = Zaphyr\Utils\Math::ROUND_DOWN;
Zaphyr\Utils\Math::percentage(40.6, 20, $precision, $roundDown); // 8.1
```

---

### ordinal

Returns the ordinal of a given number.

```php
Zaphyr\Utils\Math::ordinal(1); // '1st'
Zaphyr\Utils\Math::ordinal(2); // '2nd'
Zaphyr\Utils\Math::ordinal(223); // '223rd'
```

---

### faculty

Returns the faculty of a given number.

```php
Zaphyr\Utils\Math::faculty(10); // 3628800
```

---

### combinations

Returns all possible combinations of an array of numbers.

```php
Zaphyr\Utils\Math::combinations([1, 2]); // [1 => [1], 2 => [2], 3 => [1, 2]]
```

It is also possible to pass a second parameter to get a specific combination item.

```php
Zaphyr\Utils\Math::combinations([1, 2], 1); // [1]
Zaphyr\Utils\Math::combinations([1, 2], 2); // [2]
Zaphyr\Utils\Math::combinations([1, 2], 3); // [1, 2]
```

> [!WARNING]
> This method can be very slow and should be used with care!

---

### min

Returns the minimum required number.

```php
Zaphyr\Utils\Math::min(5, 10); // 10
Zaphyr\Utils\Math::min(20, 10); // 20
```

---

### max

Returns the maximum required number.

```php
Zaphyr\Utils\Math::max(5, 10); // 5
Zaphyr\Utils\Math::max(10, 20); // 10
```

---

### isInteger

Determines whether the given number is an integer.

```php
Zaphyr\Utils\Math::isInteger(1); // true
Zaphyr\Utils\Math::isInteger(1.2); // false
```

---

### isFloat

Determines whether the given number is a float.

```php
Zaphyr\Utils\Math::isFloat(1.2); // true
Zaphyr\Utils\Math::isFloat('1 '); // false
```

---

### isInRange

Determines whether the given number is in a given range.

```php
$number = 50;
$min = 10;
$max = 100;

Zaphyr\Utils\Math::isInRange($number, $min, $max); // true
Zaphyr\Utils\Math::isInRange($number, $min, 20); // false
```

---

### isOutOfRange

Determines whether the given number is out of a given range.

```php
$number = 50;
$min = 10;
$max = 20;

Zaphyr\Utils\Math::isOutOfRange($number, $min, $max); // true
Zaphyr\Utils\Math::isOutOfRange($number, $min, 100); // false
```

---

### isEven

Determines whether the given number is even.

```php
Zaphyr\Utils\Math::isEven(20); // true
Zaphyr\Utils\Math::isEven(0.5); // true
Zaphyr\Utils\Math::isEven(21); // false
```

---

### isOdd

Determines whether the given number is odd.

```php
Zaphyr\Utils\Math::isOdd(21); // true;
Zaphyr\Utils\Math::isOdd(21.25); // true
Zaphyr\Utils\Math::isOdd(20); // false
```

---

### isPositive

Determines whether the given number is positive.

```php
Zaphyr\Utils\Math::isPositive(1); // true
Zaphyr\Utils\Math::isPositive(0); // true
Zaphyr\Utils\Math::isPositive(0, false); // false
Zaphyr\Utils\Math::isPositive(-1); // false
```

---

### isNegative

Determines whether the given number is negative.

```php
Zaphyr\Utils\Math::isNegative(-1); // true
Zaphyr\Utils\Math::isNegative(-0.5); // true
Zaphyr\Utils\Math::isNegative(0); // false
```

---

## String

Useful string helper methods.

---

### toAscii

Returns an ASCII version of the string. A set of non-ASCII characters are replaced with their closest ASCII counterparts,
and the rest are removed by default.

```php
Zaphyr\Utils\Str::toAscii('déjà σσς iıii'); // 'deja sss iiii'
```

Since v2.0.0 it is also possible to pass a language code to the method. The language or locale of the source string can
be supplied for language-specific transliteration in any of the following formats: en, en_GB, or en-GB. For example,
passing “de” results in “äöü” mapping to “aeoeue” rather than “aou” as in other languages.

```php
Zaphyr\Utils\Str::toAscii('�Düsseldorf�', 'de'); // 'Duesseldorf'
Zaphyr\Utils\Str::toAscii('�Düsseldorf�', 'en'); // 'Dusseldorf'
```

---

### isAscii

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Checks if a string is 7 bit ASCII.

```php
Zaphyr\Utils\Str::isAscii('deja sss iiii'); // true
Zaphyr\Utils\Str::isAscii('白'); // false
```

---

### toArray

Converts a string to an array.

```php
Zaphyr\Utils\Str::toArray('Hello'); // ['H', 'e', 'l', 'l', 'o']
```

---

### toBool

Returns the boolean representation of a string. Returns true for `"1"`, `"true"`, `"on"` and `"yes"`.
Returns false otherwise.

```php
Zaphyr\Utils\Str::toBool('true'); // true
Zaphyr\Utils\Str::toBool('Off'); // false
```

### beginsWith

Determines if the given string starts with a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'C'); // true
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'c'); // false

// case insensitive
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'c', false); // true
```

---

### endsWith

Determines if a given string ends with a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::endsWith('Case sensitive', 'e'); // true
Zaphyr\Utils\Str::endsWith('Case sensitive', 'E'); // false

// case insensitive
Zaphyr\Utils\Str::endsWith('Case sensitive', 'E', false); // true
```

---

### contains

Determines if a given string contains a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::contains('Case sensitive', 'C'); // true
Zaphyr\Utils\Str::contains('Case sensitive', 'c'); // false

// case insensitive
Zaphyr\Utils\Str::contains('Case sensitive', 'c', false); // true
```

---

### containsAll

Determines if a given string contains all array substring.

```php
// case sensitive
Zaphyr\Utils\Str::containsAll('Case sensitive', ['C', 's']); // true
Zaphyr\Utils\Str::containsAll('Case sensitive', ['c', 'S']); // false

// case insensitive
Zaphyr\Utils\Str::containsAll('Case sensitive', ['c', 'S'], false); // true
```

---

### lower

Converts the given string to lower-case.

```php
Zaphyr\Utils\Str::lower('FOO'); // 'foo'
```

---

### lowerFirst

Converts the first character of a given string to lower-case.

```php
Zaphyr\Utils\Str::lowerFirst('FOO'); // 'fOO'
```

---

### upper

Converts the given string top upper-case.

```php
Zaphyr\Utils\Str::upper('foo'); // 'FOO'
```

---

### upperFirst

Converts the first character of a given string to upper-case.

```php
Zaphyr\Utils\Str::upperFirst('foo'); // 'Foo'
```

---

### limit

Limits the number of characters in a string.

```php
Zaphyr\Utils\Str::limit('Foobar', 3); // 'Foo...'
Zaphyr\Utils\Str::limit('Foobar', 3, ' ->'); // 'Foo ->'
```

---

### limitSafe

Limits the length of a string, taking into account not cutting words.

```php
Zaphyr\Utils\Str::limitSafe('Foo bar', 5); // 'Foo...'
Zaphyr\Utils\Str::limitSafe('Foo bar', 5, ' ->')); // 'Foo ->'
```

---

### firstPos

Finds position of first occurrence of string in a string.

```php
// case sensitive
Zaphyr\Utils\Str::firstPos('Hello World', 'World'); // 6
Zaphyr\Utils\Str::firstPos('Hello World', 'world'); // false

// offset
Zaphyr\Utils\Str::firstPos('Hello World, Hello You', 'Hello', 7); // 13

// case insensitive
Zaphyr\Utils\Str::firstPos('Hello World', 'world', 0, false); // 6
```

---

### lastPos

Finds position of last occurrence of a string in a string.

```php
// case sensitive
Zaphyr\Utils\Str::lastPos('Hello World, Hello You', 'Hello'); // 13
Zaphyr\Utils\Str::lastPos('Hello World, Hello You', 'hello'); // false

// offset
Zaphyr\Utils\Str::lastPos('Hello World, Hello You', 'Hello', 14); // false

// case insensitive
Zaphyr\Utils\Str::lastPos('Hello World, Hello You', 'hello', 0, false); // 13
```

---

### replace

Replaces part of a string by a matching pattern.

```php
Zaphyr\Utils\Str::replace('foo', 'f[o]+', 'bar'); // 'bar'
```

---

### stripWhitespace

Strips all whitespaces from the given string.

```php
Zaphyr\Utils\Str::stripWhitespace('Foo bar'); // 'Foobar
```

---

### insert

Inserts a string inside a string at a given position.

```php
Zaphyr\Utils\Str::insert('foo', 'bar', 3); // 'Foobar'
```

---

### equals

Determines whether two strings are equal.

```php
// case sensitive
Zaphyr\Utils\Str::equals('Case sensitive', 'Case sensitive'); // true
Zaphyr\Utils\Str::equals('Case sensitive', 'case sensitive'); // false

// case insensitive
Zaphyr\Utils\Str::equals('Case sensitive', 'case sensitive', false); // true
```

---

### length

Returns the string length.

```php
Zaphyr\Utils\Str::length('Foo'); // 3
```

---

### escape

Escapes a string.

```php
Zaphyr\Utils\Str::escape('<h1>Hello world</h1>'); // '&lt;h1&gt;Hello world&lt;/h1&gt;'
```

---

### title

Converts the given string to title case.

```php
Zaphyr\Utils\Str::title('hello world'); // 'Hello World'
```

---

### slug

Generates a URL friendly "slug" from a given string.

```php
Zaphyr\Utils\Str::slug('Hello World'); // 'hello-world'
```

Since v2.0.0 it is also possible to pass a language code to the method. The language or locale of the source string can
be supplied for language-specific transliteration in any of the following formats: en, en_GB, or en-GB. For example,
passing “de” results in “äöü” mapping to “aeoeue” rather than “aou” as in other languages.

```php
Zaphyr\Utils\Str::slug('Düsseldorf ist eine schöne Stadt', '_', 'de'); // 'duesseldorf_ist_eine_schoene_stadt'
```

---

### studly

Converts a value to studly caps case.

```php
Zaphyr\Utils\Str::studly('Sho-ebo-x'); // 'ShoEboX'
Zaphyr\Utils\Str::studly('Sho -_- ebo -_ - x')); // 'ShoEboX'
```

---

### camel

Converts a value to camel case.

```php
Zaphyr\Utils\Str::camel('Hello world-whats up'); // 'helloWorldWhatsUp'
```

---

### snake

Converts a string to snake case.

```php
Zaphyr\Utils\Str::snake('HelloWorld'); // 'hello_world'
```

---

## Template

<span class="badge rounded-pill text-bg-primary">Available since v2.1.0</span>

The `Template` class provides an easy way to enrich templates with placeholders. The class is very limited and can
only replace simple string placeholders. So this class is rather meant for small projects, simple template files or
prototyping.

---

### render

The template class provides only a `render()` function. This function takes the path to the template file and an
array of placeholders and replaces them with the corresponding values inside the provided template file.
All placeholders are automatically escaped!

```php
echo Zaphyr\Utils\Template::render('/path/to/template/file.html', [
    'name' => 'John Doe',
]);
// <p>Hello John Doe</p>
```

The template file will look like the following example. All variables which should be replaced within the template file
are wrapped with `%`.

```html
<p>Hello %name%</p>
```

---

## Timezone

Useful helper class to work with time zones

---

### getAllTimezones

Returns all available timezones.

```php
Zaphyr\Utils\Timezone::getAllTimezones();
// [
//      'Africa' => [
//          'Abidjan' => '(UTC+00:00) Abidjan',
//          […]
//      ],
//      […]
// ],
```
> [!NOTE]
> Since v2.0.0 this method returns the timezones grouped by its corresponding continent. Also, the timezones are now
> generated dynamically via `\DateTimeZone::listIdentifiers()`

---

### getAllTimezonesAsJsonString

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns all available timezones as JSON string.

```php
Zaphyr\Utils\Timezone::getAllTimezonesAsJsonString();
// {'Africa': {'Abidjan': '(UTC+00:00) Abidjan' […] 'Wallis': '(UTC+12:00) Wallis'}}
```

---

### getTimezonesByContinent

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns a timezone array of the given continent.

```php
Zaphyr\Utils\Timezone::getTimezonesByContinent('Europe');
// [
//    'Amsterdam' => '(UTC+02:00) Amsterdam'
//    […]
//    'Zurich' => '(UTC+02:00) Zurich'
// ]
```

---

### getTimezone

<span class="badge rounded-pill text-bg-primary">Available since v2.0.0</span>

Returns the timezone of the given continent and country.

```php
Zaphyr\Utils\Timezone::getTimezone('Europe', 'Amsterdam'); // '(UTC+02:00) Amsterdam'
```
