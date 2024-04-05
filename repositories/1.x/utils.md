# Utils

> [!WARNING]
> You're browsing the documentation for an old version running php 7.x.
> Consider upgrading to [v2.x](/docs/repositories/2.x/utils).

A collection of useful helper classes, which make a developer's workday a little easier.

## Installation

To get started, install the utils repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/utils
```

## Array

Useful helper methods to work with arrays.

#### accessible

Determines whether the given value is an array.

```php
Zaphyr\Utils\Arr::accessible(['foo' => ['bar']]); // true
```

#### exists

Determines whether the given key exists in the provided array.

```php
Zaphyr\Utils\Arr::exists(['foo' => ['bar']], 'foo'); // true
```

#### set

Sets an array item to a given value using "dot" notation.
If no key is given to the method, the entire array will be replaced.

```php
$array = ['foo' => ['bar']];

Zaphyr\Utils\Arr::set($array, 'foo', 'baz'); // ['foo' => 'baz']
Zaphyr\Utils\Arr::set($array, 'baz', 'qux'); // ['foo' => ['bar'], 'baz' => 'qux']
Zaphyr\Utils\Arr::set($array, 'bar.baz', 'QUX'); // ['foo' => ['bar'], 'bar' => ['baz' => 'QUX']]
Zaphyr\Utils\Arr::set($array, ['bar' => 'BAR', 'baz' => 'BAZ']) // ['bar' => 'BAR', 'baz' => 'BAZ']
```

#### add

Adds an element to an array using "dot" notation if the given key doesn't exist.
The `add` method will not overwrite existing keys!

```php
$array = ['foo' => ['bar']];

Zaphyr\Utils\Arr::add($array, 'baz', 'qux'); // ['foo' => ['bar'], 'baz' => 'qux']
Zaphyr\Utils\Arr::add($array, 'foo', 'BAR'); // ['foo' => ['bar']]
```

#### get

Returns an item from an array using "dot" notation.

```php
Zaphyr\Utils\Arr::get(['foo' => ['bar' => 'baz']], 'foo.bar'); // 'baz'
```

The `get()` method also accepts a default value, which will be returned if the specified key is not found.

```php
Zaphyr\Utils\Arr::get(['foo' => ['bar' => 'baz']], 'foo.qux', 'qux');
```

#### first

Returns the first element in an array passing a given truth test.

```php
Zaphyr\Utils\ Arr::first([100, 200, 300], function ($value, $key) {
    return $value <= 150;
})) // 100
```

A default value may also be passed as the third parameter to the method.
This value will be returned if no value passes the truth test.


```php
Zaphyr\Utils\Arr::first([100, 200, 300], function ($value, $key) {
    return $value <= 50;
}, 100) // 100
```

#### last

Returns the last element in an array passing a given truth test.

```php
Zaphyr\Utils\Arr::last([100, 200, 300], function ($value, $key) {
    return $value >= 150;
}); // 300
```

A default value may be passed as the third argument to the method.
This value will be returned if no value passes the truth test.

```php
Zaphyr\Utils\Arr::last([100, 200, 300], function ($value, $key) {
    return $value >= 400;
}, 400); // 400
```

#### has

Checks if an item or items exist in an array using "dot" notation.

```php
Zaphyr\Utils\Arr::has(['foo' => ['bar' => 'baz'], 'qux'], 'foo.bar'); // true
```

#### where

Filters an array using the given callback.

```php
Zaphyr\Utils\Arr::where([100, '200', 300], function ($value, $key) {
    return is_string($value);
}); // ['200']
```

#### only

Returns a subset of the items from the given array.

```php
Zaphyr\Utils\Arr::only(['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'], ['foo', 'quu']); // ['foo' => 'bar', 'quu' => 'qaa']
```

#### forget

Removes one or many array items from a given array using "dot" notation.

```php
$array = ['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'];

Zaphyr\Utils\Arr::forget($array, ['baz', 'quu']);

$array; // ['foo' => 'bar']
```

#### except

Returns all the given array elements except for a specified array of keys.

```php
Zaphyr\Utils\Arr::except(['foo' => 'bar', 'baz' => 'qux', 'quu' => 'qaa'], ['foo', 'quu']); // ['baz' => 'qux']
```

## ClassFinder

Useful helper methods to work with classes.

#### getClassesFromDirectory

Returns an array with all classes of a given directory.

```php
Zaphyr\Utils\ClassFinder::getClassesFromDirectory('src'); // [...]
```

#### getClassNameFromFile

Returns the class name of a given file.

```php
Zaphyr\Utils\ClassFinder::getClassNameFromFile('src/Arr.php'); // 'Arr'
```

#### getNamespaceFromFile

Returns the namespace from a given file.

```php
Zaphyr\Utils\ClassFinder::getNamespaceFromFile('src/Arr.php'); // 'Zaphyr\Utils'
```

## Country

Simply returns an array of all 242 countries.

#### getAllCountries

Returns an array of all available countries.

```php
Zaphyr\Utils\Country::getAllCountries();
```

## Date

Useful helper methods to work with dates and times.

#### timestamp

Returns the current timestamp or the timestamp of a given date.

```php
Zaphyr\Utils\Date::timestamp(); // time()
Zaphyr\Utils\Date::timestamp(new DateTime('2022-09-13 09:41:00')); // 1663918860
Zaphyr\Utils\Date::timestamp('1663918860'); // 1663918860
Zaphyr\Utils\Date::timestamp('yesterday'); // 'yesterday'
```

#### timezone

Returns the timezone.

```php
Zaphyr\Utils\Date::timezone(new DateTimeZone('UTC')); // \DateTimezone::class
Zaphyr\Utils\Date::timezone()->getName(); // 'UTC'
Zaphyr\Utils\Date::timezone('GMT')->getName(); // 'GMT'
```

#### factory

Returns a DateTime object.

```php
Zaphyr\Utils\Date::factory(); // \DateTime::class
Zaphyr\Utils\Date::factory('2022-09-13 09:41:00'); // \DateTime::class
```

#### sqlFormat

Return a SQL compliant date format.

```php
Zaphyr\Utils\Date::sqlFormat(); // 'Y-m-d H:i:s'
Zaphyr\Utils\Date::sqlFormat(1663918860); // '2022-09-13 09:41:00'
Zaphyr\Utils\Date::sqlFormat('2022-09-13'); // '2022-09-13 00:00:00'
```

#### humanReadable

Returns a human-readable date format.

```php
Zaphyr\Utils\Date::humanReadable(1663918860); // '23 Sep 2022 09:41'
Zaphyr\Utils\Date::humanReadable('2022-09-23 09:41:00', 'd F Y'); // '23 September 2022'
```

#### isValid

Determines whether a given date is valid.

```php
Zaphyr\Utils\Date::isValid('1663918860'); // true
Zaphyr\Utils\Date::isValid('now'); // true
Zaphyr\Utils\Date::isValid('2022-09-23 09:41:00'); // true
Zaphyr\Utils\Date::isValid(''); // false
```

#### isToday

Determines whether a given date is today.

```php
Zaphyr\Utils\Date::isToday('+0 day'); // true
```

#### isTomorrow

Determines whether a given date is tomorrow.

```php
Zaphyr\Utils\Date::isTomorrow('+1 day'); // true
Zaphyr\Utils\Date::isTomorrow('+0 day'); // false
```

#### isThisWeek

Determines whether a given date is this week.

```php
Zaphyr\Utils\Date::isThisWeek('+0 week'); // ture
Zaphyr\Utils\Date::isThisWeek('+1 week'); // false
```

#### isThisMonth

Determines whether a given date is this month.

```php
Zaphyr\Utils\Date::isThisMonth('+0 month'); // true
Zaphyr\Utils\Date::isThisMonth('+1 month') // false
```

#### isThisYear

Determines whether a given date is this year.

```php
Zaphyr\Utils\Date::isThisYear('+0 year'); // true
Zaphyr\Utils\Date::isThisYear('+1 year'); // false
```

## File

Useful helper methods to work with files and folders.

#### exists

Determines whether a file or directory exists.

```php
Zaphyr\Utils\File::exists(__FILE__); // true
Zaphyr\Utils\File::exists(__DIR__); // true
```

#### isFile

Determines whether a file exists.

```php
Zaphyr\Utils\File::isFile(__FILE__); // true
Zaphyr\Utils\File::isFile(__DIR__); // false
```

#### isDirectory

Determines whether a directory exists.

```php
Zaphyr\Utils\File::isDir(__DIR__); // true
Zaphyr\Utils\File::isDir(__FILE__); // false
```

#### info

Returns a path information of a given file or directory.

```php
Zaphyr\Utils\File::info(__FILE__, PATHINFO_EXTENSION);
Zaphyr\Utils\File::info(__DIR__, PATHINFO_DIRNAME);
```

#### name

Returns the name of a file or directory path.

```php
Zaphyr\Utils\File::name(__FILE__);
Zaphyr\Utils\File::name(__DIR__);
```

#### basename

Returns the basename of a file or directory path.

```php
Zaphyr\Utils\File::basename(__FILE__);
Zaphyr\Utils\File::basename(__DIR__);
```

#### dirname

Returns the directory name of a file or directory.

```php
Zaphyr\Utils\File::dirname(__FILE__);
Zaphyr\Utils\File::dirname(__DIR__);
```

#### extension

Returns the file extension of a given file.

```php
Zaphyr\Utils\File::extension(__FILE__);
Zaphyr\Utils\File::extension(__DIR__); // null
```

#### type

Returns the type of file or directory.

```php
Zaphyr\Utils\File::type(__FILE__); // 'file'
Zaphyr\Utils\File::type(__DIR__); // 'dir'
```

#### mimeType

Returns the mime type of file or directory.

```php
Zaphyr\Utils\File::mimeType(__FILE__); // 'text/x-php'
Zaphyr\Utils\File::mimeType(__DIR__); // 'directory'
```

#### size

Returns the size of a file or directory.

```php
Zaphyr\Utils\File::type(__FILE__);
Zaphyr\Utils\File::type(__DIR__);
```

#### hash

Calculates the md5 hash of a given file.

```php
Zaphyr\Utils\File::hash(__FILE__);
Zaphyr\Utils\File::hash(__DIR__); // null
```

#### chmod

Changes the file mode.

```php
// set chmod
Zaphyr\Utils\File::chmod(__DIR__, '0775'); // bool

// get chmod
Zaphyr\Utils\File::chmod(__DIR__); // '0775'
```

#### lastModified

Returns the file modification time.

```php
Zaphyr\Utils\File::lastModified(__FILE__);
Zaphyr\Utils\File::lastModified(__DIR__);
```

#### isReadable

Determines whether a file or directory is readable.

```php
Zaphyr\Utils\File::isReadable(__FILE__); // true
Zaphyr\Utils\File::isReadable(__DIR__); // true
```

#### isWritable

Determines whether a file or directory is writable.

```php
Zaphyr\Utils\File::isWritable(__FILE__); // true
Zaphyr\Utils\File::isWritable(__DIR__); // true
```

#### glob

Finds all pathname of a matching pattern.

```php
Zaphyr\Utils\File::glob(__DIR__ . '/*.*'); // [...]
```

#### files

Returns all files in a given directory.

```php
Zaphyr\Utils\File::files(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::files('non-existing-dir'); // null

// with hidden files
Zaphyr\Utils\Files::files(__DIR__, true); // SplFileInfo[]
```

#### allFiles

Returns all files in a given directory including all files in subdirectories.

```php
Zaphyr\Utils\File::allFiles(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::allFiles('non-existing-dir'); // null

// with hidden files
Zaphyr\Utils\File::allFiles(__DIR__, true); // SplFileInfo[]
```

#### directories

Returns all directories inside a given directory.

```php
Zaphyr\Utils\File::directories(__DIR__); // SplFileInfo[]
Zaphyr\Utils\File::directories('non-existing-dir'); // null

// with hidden directories
Zaphyr\Utils\File::directories(__DIR__, true); // SplFileInfo[]
```

#### getRequire

Requires a file.

```php
try {
    Zaphyr\Utils\File::getRequire(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

#### getRequireOnce

Requires a file once.

```php
try {
    Zaphyr\Utils\File::getRequireOnce(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

#### read

Reads the contents of a file.

```php
try {
    Zaphyr\Utils\File::read(__FILE__);
} catch (Zaphyr\Utils\Exceptions\FileNotFoundException $exception) {
    //
}
```

#### put

Puts content inside a file.

```php
Zaphyr\Utils\File::put(__FILE__, 'contents');
```

#### replace

Replaces content inside a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::replace(__FILE__, 'contents');
```

#### prepend

Prepends content to a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::prepend(__FILE__, 'contents');
```

#### append

Appends content to a given file. If the file does not exist it will be created.

```php
Zaphyr\Utils\File::append(__FILE__, 'contents');
```

#### delete

Deletes a given file.

```php
Zaphyr\Utils\File::delete('file'); // true
Zaphyr\Utils\File::delete(['file1', 'file2']); // true
```

#### move

Moves a file to a given directory.

```php
Zaphyr\Utils\File::move('source-file', 'destination-target'); // true
```

#### copy

Copies a file to a given directory.

```php
Zaphyr\Utils\File::copy('source-file', 'destination-target'); // true
```

#### createDirectory

Creates a new directory.

```php
$directory = 'dirname';
$mode = 0755;
$recursive = false;
$force = false;

Zaphyr\Utils\File::createDirectory($directory, $mode, $recursive, $force);
```

#### deleteDirectory

Deletes an existing directory.

```php
$directory = 'directory'
$preserve = false;

Zaphyr\Utils\File::deleteDirectory($directory, $preserve); // true
Zaphyr\Utils\File::deleteDirectory('none-existing-directory'); // false
```

#### cleanDirectory

Removes all files and directories inside a given directory.

```php
Zaphyr\Utils\File::cleanDirectory('directory'); // true
Zaphyr\Utils\File::cleanDirectory('none-existing-directory'); // false
```

#### moveDirectory

Moves a directory to a given path.

```php
Zaphyr\Utils\File::moveDirectory('from', 'to');
```

#### copyDirectory

Copies a directory to a given path.

```php
$options = \FilesystemIterator::SKIP_DOTS;

Zaphyr\Utils\File::copyDirectory('source-directory', 'target-destination-directory', $options);
```

## Filter

Useful helper methods to filter for occurrences.

#### alpha

Returns only the alpha chars of a string.

```php
Zaphyr\Utils\Filter::alpha('fo-0-o'); // 'foo'
```

#### alphanum

Returns only the alphanum chars of a string.

```php
Zaphyr\Utils\Filter::alphanum('foo-* 123'); // 'foo123'
```

#### base64

Returns only the base64 chars of a string.

```php
Zaphyr\Utils\Filter::base64('a8O2bGph*c2do ZHZiYX () Nua2zDtmzDpMOkYQ=='); // 'a8O2bGphc2doZHZiYXNua2zDtmzDpMOkYQ=='
```

#### digits

Returns only digits of the given string.

```php
Zaphyr\Utils\Filter::digits('f1o2o3'); // 123
```

#### float

Smart converts any string to float with round.

```php
Zaphyr\Utils\Filter::float('1.5123', 2); // 1.51
Zaphyr\Utils\Filter::float('- 1 0'); // -10.0
```

#### int

Smart converts any string to int.

```php
Zaphyr\Utils\Filter::int('+3'); // 3
Zaphyr\Utils\Filter::int('- 1 0'); // -10
```

## Form

Useful helper methods for HTML forms.

#### open

Opens a new form.

```php
Zaphyr\Utils\Form::open(); // '<form accept-charset="UTF-8" method="POST">'
Zaphyr\Utils\Form::open(['method' => 'get', 'action' => 'https://localhost/foo']); // '<form accept-charset="UTF-8" method="GET" action="https://localhost/foo">'
Zaphyr\Utils\Form::open(['method' => 'POST', 'class' => 'form', 'id' => 'id-form']); // '<form accept-charset="UTF-8" method="POST" class="form" id="id-form">'
Zaphyr\Utils\Form::open(['method' => 'GET', 'accept-charset' => 'UTF-16']); // '<form accept-charset="UTF-16" method="GET">'
Zaphyr\Utils\Form::open(['accept-charset' => 'UTF-16', 'files' => true]); // '<form accept-charset="UTF-16" method="POST" enctype="multipart/form-data">'
Zaphyr\Utils\Form::open(['method' => 'PUT']); // '<form accept-charset="UTF-8" method="POST"><input name="_method" type="hidden" value="PUT">'
```

#### getMethod

Returns the method of the form.

```php
Zaphyr\Utils\Form::getMethod('get'); // 'GET'
Zaphyr\Utils\Form::getMethod('DELETE'); // 'POST'
```

#### getAction

Returns the action of the form.

```php
Zaphyr\Utils\Form::getAction(['action' => '/foo/bar']); // 'foo/bar'
Zaphyr\Utils\Form::getAction([]); // null
```

#### close

Closes a form.

```php
Zaphyr\Utils\Form::close(); // '</form>'
```

#### label

Returns a label for an input field.

```php
Zaphyr\Utils\Form::label('foo-bar'); // <label for="foo-bar">Foo Bar</label>
Zaphyr\Utils\Form::label('foo', 'Bar'); // '<label for="foo">Bar</label>'
Zaphyr\Utils\Form::label('foo', 'Bar', ['id' => 'baz']); // '<label for="foo" id="baz">Bar</label>'
Zaphyr\Utils\Form::label('foo', '<span>Bar</span>'); // '<label for="foo">&lt;span&gt;Bar&lt;/span&gt;</label>'
Zaphyr\Utils\Form::label('foo', '<span>Bar</span>', [], false); // '<label for="foo"><span>Bar</span></label>'
```

#### input

Returns an input field.

```php
Zaphyr\Utils\Form::input('text', 'foo'); // '<input name="foo" type="text">'
Zaphyr\Utils\Form::input('text', 'foo', 'bar'); // '<input name="foo" type="text" value="bar">'
Zaphyr\Utils\Form::input('date', 'foo', null, ['class' => 'bar']); // '<input class="bar" name="foo" type="date">'
Zaphyr\Utils\Form::input('checkbox', 'foo', true); // '<input id="foo" name="foo" type="text">'
Zaphyr\Utils\Form::input('text', 'foo[bar]'); // '<input name="foo[bar]" type="text">'
```

#### text

Returns a `text` input field.

```php
Zaphyr\Utils\Form::text('foo'); // '<input name="foo" type="text">'
Zaphyr\Utils\Form::text('foo', 'bar'); // '<input name="foo" type="text" value="bar">'
Zaphyr\Utils\Form::text('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="text">'
Zaphyr\Utils\Form::text('foo[bar]'); // '<input name="foo[bar]" type="text">'

```

#### password

Returns a `password` input field.

```php
Zaphyr\Utils\Form::password('foo'); // <input name="foo" type="password" value="">
Zaphyr\Utils\Form::password('foo', ['class' => 'baz']); // '<input class="baz" name="foo" type="password" value="">'
```

#### range

Returns a `range` input field.

```php
Zaphyr\Utils\Form::range('foo'); // <input name="foo" type="range">
Zaphyr\Utils\Form::range('foo', '1'); // <input name="foo" type="range" value="1">
Zaphyr\Utils\Form::range('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="range">'
```

#### hidden

Returns a `hidden` input field.

```php
Zaphyr\Utils\Form::hidden('foo'); // '<input name="foo" type="hidden">'
Zaphyr\Utils\Form::hidden('foo', 'bar'); // '<input name="foo" type="hidden" value="bar">'
Zaphyr\Utils\Form::hidden('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="hidden">'
```

#### search

Returns a `search` input field.

```php
Zaphyr\Utils\Form::search('foo')); // '<input name="foo" type="search">'
Zaphyr\Utils\Form::search('foo', 'bar'); // '<input name="foo" type="search" value="bar">'
Zaphyr\Utils\Form::search('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="search">'
```

#### email

Returns an `email` input field.

```php
Zaphyr\Utils\Form::email('foo'); // '<input name="foo" type="email">'
Zaphyr\Utils\Form::email('foo', 'foo@bar.com'); // '<input name="foo" type="email" value="foo@bar.com">'
Zaphyr\Utils\Form::email('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="email">'
```

#### tel

Returns a `tel` input field.

```php
Zaphyr\Utils\Form::tel('foo')); // '<input name="foo" type="tel">'
Zaphyr\Utils\Form::tel('foo', '1234'); // '<input name="foo" type="tel" value="1234">'
Zaphyr\Utils\Form::tel('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="tel">'
```

#### number

Returns a `number` input field.

```php
Zaphyr\Utils\Form::number('foo'); // '<input name="foo" type="number" value="1234">'
Zaphyr\Utils\Form::number('foo', '1234'); // '<input name="foo" type="number" value="1234">'
Zaphyr\Utils\Form::Number('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="number">'
```

#### date

Returns a `date` input field.

```php
Zaphyr\Utils\Form::date('foo'); // '<input name="foo" type="date">'
Zaphyr\Utils\Form::date('foo', '2022-09-23'); // '<input name="foo" type="date" value="2022-09-23">'
Zaphyr\Utils\Form::date('foo', new DateTime('2022-09-23')); // '<input name="foo" type="date" value="2022-09-23">'
```

#### datetime

Returns a `datetime` input field.

```php
Zaphyr\Utils\Form::datetime('foo'); // '<input name="foo" type="datetime">'
Zaphyr\Utils\Form::datetime('foo', '2022-09-23T09:41:00+00:00'); // '<input name="foo" type="datetime" value="2022-09-23T09:41:00+00:00">'
Zaphyr\Utils\Form::datetime('foo', new DateTime('2022-09-23')); // '<input name="foo" type="datetime" value="2022-09-23T00:00:00+01:00">'
Zaphyr\Utils\Form::datetime('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="datetime">'
```

#### datetimelocal

Returns a `datetime-local` input field.

```php
Zaphyr\Utils\Form::datetimeLocal('foo'); '<input name="foo" type="datetime-local">'
Zaphyr\Utils\Form::datetimeLocal('foo', '2022-09-23T09:41'); // '<input name="foo" type="datetime-local" value="2022-09-23T09:41">'
Zaphyr\Utils\Form::datetimeLocal('foo', new \DateTime('2022-09-23')); // '<input name="foo" type="datetime-local" value="2022-09-23T00:00">'
Zaphyr\Utils\Form::datetimeLocal('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="datetime-local">'
```

#### time

Returns a `time` input field.

```php
Zaphyr\Utils\Form::time('foo')); // '<input name="foo" type="time">'
Zaphyr\Utils\Form::time('foo', '05:04'); // '<input name="foo" type="time" value="05:04">'
Zaphyr\Utils\Form::time('foo', new DateTime('15:00')); // '<input name="foo" type="time" value="15:00">'
Zaphyr\Utils\Form::time('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="time">'
```

#### week

Returns a `week` input field.

```php
Zaphyr\Utils\Form::week('foo'); // '<input name="foo" type="week">'
Zaphyr\Utils\Form::week('foo', '2019-W12'); // '<input name="foo" type="week" value="2019-W12">'
Zaphyr\Utils\Form::week('foo', new DateTime('2019-W32')); // '<input name="foo" type="week" value="2019-W32">'
Zaphyr\Utils\Form::week('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="week">'
```

#### month

Returns a `month` input field.

```php
Zaphyr\Utils\Form::month('foo'); // '<input name="foo" type="month">'
Zaphyr\Utils\Form::month('foo', '2019-03'); // '<input name="foo" type="month" value="2019-03">'
Zaphyr\Utils\Form::month('foo', new DateTime('2019-09')); // '<input name="foo" type="month" value="2019-09">'
Zaphyr\Utils\Form::month('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="month">'
```

#### url

Returns a `url` input field.

```php
Zaphyr\Utils\Form::url('foo'); // '<input name="foo" type="url">'
Zaphyr\Utils\Form::url('foo', 'https://localhost'); // '<input name="foo" type="url" value="https://localhost">'
Zaphyr\Utils\Form::url('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="url">'
```

#### file

Returns a `file` input field.

```php
Zaphyr\Utils\Form::file('foo'); // '<input name="foo" type="file">'
Zaphyr\Utils\Form::file('foo', ['class' => 'baz']); // '<input class="baz" name="foo" type="file">'
```

#### image

Returns a `image` input field.

```php
Zaphyr\Utils\Form::image('foo', 'https://localhost/foo.jpg'); // '<input src="https://localhost/foo.jpg" name="foo" type="image">'
Zaphyr\Utils\Form::image('foo', 'https://localhost/foo.jpg', ['class' => 'baz']); // '<input class="baz" src="https://localhost/foo.jpg" name="foo" type="image">'
```

#### color

Returns a `color` input field.

```php
Zaphyr\Utils\Form::color('foo'); // '<input name="foo" type="color">'
Zaphyr\Utils\Form::color('foo', '#ffffff'); // '<input name="foo" type="color" value="#ffffff">'
Zaphyr\Utils\Form::color('foo', null, ['class' => 'baz']); // '<input class="baz" name="foo" type="color">'
```

#### reset

Returns a `reset` field.

```php
Zaphyr\Utils\Form::reset('Reset'); // '<input type="reset" value="Reset">'
Zaphyr\Utils\Form::reset('Reset', ['class' => 'baz']); // '<input class="baz" type="reset" value="Reset">'
```

#### submit

Returns a `submit` field.

```php
Zaphyr\Utils\Form::submit('Submit'); // '<input type="submit" value="Submit">'
Zaphyr\Utils\Form::submit('Submit', ['class' => 'baz']); // '<input class="baz" type="submit" value="Submit">',
```

#### button

Returns a button field.

```php
Zaphyr\Utils\Form::button('Submit'); // '<button type="button">Submit</button>'
Zaphyr\Utils\Form::button('Submit', ['class' => 'baz', 'type' => 'submit']); // '<button class="baz" type="submit">Submit</button>'
```

#### checkbox

Returns a checkbox field.

```php
Zaphyr\Utils\Form::checkbox('foo'); // '<input name="foo" type="checkbox" value="1">'
Zaphyr\Utils\Form::checkbox('foo', 'bar', true); // '<input checked="checked" name="foo" type="checkbox" value="bar">',
Zaphyr\Utils\Form::checkbox('foo', 'bar', false, ['class' => ['baz']]); // '<input class="baz" name="foo" type="checkbox" value="bar">'
```

#### radio

Returns a radio field.

```php
Zaphyr\Utils\Form::radio('foo'); // '<input name="foo" type="radio" value="foo">'
Zaphyr\Utils\Form::radio('foo', 'bar', true); // '<input checked="checked" name="foo" type="radio" value="bar">'
Zaphyr\Utils\Form::radio('foo', 'bar', false, ['class' => ['baz']]); // '<input class="baz" name="foo" type="radio" value="bar">'
```

#### textarea

Returns a textarea field.

```php
Zaphyr\Utils\Form::textarea('foo'); // '<textarea name="foo" cols="50" rows="10"></textarea>'
Zaphyr\Utils\Form::textarea('foo', 'Bar'); // '<textarea name="foo" cols="50" rows="10">Bar</textarea>'
Zaphyr\Utils\Form::textarea('foo', '<span>Bar</span>'); // '<textarea name="foo" cols="50" rows="10">&lt;span&gt;Bar&lt;/span&gt;</textarea>'
Zaphyr\Utils\Form::textarea('foo', '<span>Bar</span>', [], false); // '<textarea name="foo" cols="50" rows="10"><span>Bar</span></textarea>'
Zaphyr\Utils\Form::textarea('foo', null, ['class' => 'baz']); // '<textarea class="baz" name="foo" cols="50" rows="10"></textarea>'
Zaphyr\Utils\Form::textarea('foo', null, ['size' => '60x20']); // '<textarea name="foo" cols="60" rows="20"></textarea>'
Zaphyr\Utils\Form::textarea('foo', null, ['rows' => 20, 'cols' => 60]); // '<textarea rows="20" cols="60" name="foo"></textarea>'
```

#### select

Returns a select field.

```php
Zaphyr\Utils\Form::select('foo'); // '<select name="foo"></select>'

Zaphyr\Utils\Form::select('gender', ['Male', 'Female']);
// <select name="gender">
//      <option value="0">Male</option>
//      <option value="1">Female</option>
//  </select>

Zaphyr\Utils\Form::select('gender', ['m' => 'Male', 'f' => 'Female']);
// <select name="gender">
//      <option value="m">Male</option>
//      <option value="f">Female</option>
//  </select>

Zaphyr\Utils\Form::select('gender', ['m' => 'Male', 'f' => 'Female'], 'm');
// <select name="gender">
//      <option value="m" selected="selected">Male</option>
//      <option value="f">Female</option>
//  </select>

Zaphyr\Utils\Form::select('gender', ['m' => 'Male', 'f' => 'Female'], ['m', 'f'], ['multiple']);
// <select multiple name="gender">
//      <option value="m" selected="selected">Male</option>
//      <option value="f" selected="selected">Female</option>
//  </select>

Zaphyr\Utils\Form::select('gender', ['m' => 'Male', 'f' => 'Female'], null, ['class' => 'form-class', 'id' => 'form-id']);
// <select class="form-class" id="form-id" name="gender">
//      <option value="m">Male</option>
//      <option value="f">Female</option>
//  </select>

Zaphyr\Utils\Form::select('size', ['Large sizes' => ['l' => 'Large', 'xl' => 'Extra Large'], 's' => 'Small']);
// <select name="size">
//      <optgroup label="Large sizes">
//          <option value="l">Large</option>
//          <option value="xl">Extra Large</option>
//      </optgroup>
//      <option value="s">Small</option>
//  </select>

Zaphyr\Utils\Form::select('size', [
    'Large sizes' => ['l' => 'Large', 'xl' => 'Extra Large'],
    'm' => 'Medium',
    'Small sizes' => ['s' => 'Small', 'xs' => 'Extra Small'],
], null, [], ['Large sizes' => ['l' => ['disabled']], 'm' => ['disabled']], ['Small sizes' => ['disabled']]);
//  <select name="size">
//		<optgroup label="Large sizes">
//			<option value="l" disabled>Large</option>
//			<option value="xl">Extra Large</option>
//		</optgroup>
//		<option value="m" disabled>Medium</option>
//		<optgroup label="Small sizes" disabled>
//			<option value="s">Small</option>
//			<option value="xs">Extra Small</option>
//		</optgroup>
//	</select>

Zaphyr\Utils\Form::select('foo', ['<span>Bar</span>']);
// <select name="foo">
//      <option value="0">&lt;span&gt;Bar&lt;/span&gt;</option>
//  </select>

Zaphyr\Utils\Form::select(
    'gender',
    ['m' => 'Male', 'f' => 'Female'],
    null,
    [],
    ['m' => ['data-foo' => 'foo', 'disabled']]
);
// <select name="gender">
//      <option value="m" data-foo="foo" disabled>Male</option>
//      <option value="f">Female</option>
//  </select>

Zaphyr\Utils\Form::select('avc', [1 => 'Yes', 0 => 'No'], true, ['placeholder' => 'Choose']);
// <select name="avc">
//      <option value="">Choose</option>
//      <option value="1" selected>Yes</option>
//      <option value="0" >No</option>
//  </select>

Zaphyr\Utils\Form::select('size[multi][]', ['m' => 'Medium', 'l' => 'Large'], 'l', ['multiple' => 'multiple']);
// <select multiple="multiple" name="size[multi][]">
//      <option value="m">Medium</option>
//      <option value="l" selected="selected">Large</option>
//  </select>

Zaphyr\Utils\Form::select('size[key]', ['m' => 'Medium', 'l' => 'Large']);
// <select name="size[key]">
//      <option value="m">Medium</option>
//      <option value="l">Large</option>
//  </select>
```

#### selectRange

Returns a select range field.

```php
Zaphyr\Utils\Form::selectRange('year', 2022, 2024);
// <select name="year">
//      <option value="2022">2022</option>
//      <option value="2023">2023</option>
//      <option value="2024">2024</option>
//  </select>

Zaphyr\Utils\Form::selectRange('year', 2022, 2024, 2023);
// <select name="year">
//      <option value="2022">2022</option>
//      <option value="2024" selected="selected">2024</option>
//      <option value="2023">2023</option>
//  </select>

Zaphyr\Utils\Form::selectRange('year', 2022, 2024, '2023', ['class' => 'form-class']);
// <select class="form-class" name="year">
//      <option value="2022">2022</option>
//      <option value="2023" selected="selected">2023</option>
//      <option value="2024">2024</option>
//  </select>

Zaphyr\Utils\Form::selectRange('year', 2022, 2024, null, [], ['2022' => ['data-foo' => 'foo', 'disabled']]);
// <select name="year">
//		<option value="2022" data-foo="foo" disabled>2022</option>
//		<option value="2023">2023</option>
//		<option value="2024">2024</option>
//	</select>
```

#### datalist

Returns a datalist field.

```php
Zaphyr\Utils\Form::datalist('genders', ['male', 'female']);
// <datalist id="genders">
//      <option value="male">male</option>
//      <option value="female">female</option>
//  </datalist>

Zaphyr\Utils\Form::datalist('genders', ['m' => 'male', 'f' => 'female']);
// <datalist id="genders">
//      <option value="m">male</option>
//      <option value="f">female</option>
// </datalist>

Zaphyr\Utils\Form::datalist('genders', [1 => 'male', 2 => 'female']);
// <datalist id="genders">
//      <option value="1">male</option>
//      <option value="2">female</option>
// </datalist>
```

## HTML

Contains a few HTML attributes methods.

#### attributes

Builds an HTML attribute string from an array.

```php
Zaphyr\Utils\HTML::attributes(['id' => 'foo', 'class' => ['bar', 'baz']]); // ' id="foo" class="bar baz"'
```

#### attributeElements

Builds a single HTML attribute element.

```php
Zaphyr\Utils\HTML::attributeElement('id', 'foo'); // 'id="foo"'
Zaphyr\Utils\HTML::attributeElement('class', ['bar', 'baz']); // 'class="bar baz"'
```

## Math

Useful helper methods for calculating.

#### round

Rounds a given number up/down by a given precision.

```php
Zaphyr\Utils\Math::round(1.61, 1); // 1.7
Zaphyr\Utils\Math::round(1.61, 1, Math::ROUND_DOWN); // 1.6
```

#### average

Returns the average of a given number array.

```php
Zaphyr\Utils\Math::average([1, 3, 5]); // 3.0
```

#### percentage

Returns the percentage of a given number.

```php
$percentage = 40;
$total = 20;

Zaphyr\Utils\Math::percentage($percentage, $total); // 8
```

#### ordinal

Returns the ordinal of a given number.

```php
Zaphyr\Utils\Math::ordinal(1); // '1st'
Zaphyr\Utils\Math::ordinal(2); // '2nd'
Zaphyr\Utils\Math::ordinal(223); // '223rd'
```

#### faculty

Returns the faculty of a given number.

```php
Zaphyr\Utils\Math::faculty(10); // 3628800
```

#### combinations

Returns all possible combinations of an array of numbers.

```php
Zaphyr\Utils\Math::combinations([1, 2]); // [1 => [1], 2 => [2], 3 => [1, 2]]
```

> [!WARNING]
> This method can be very slow and should be used with care!

#### min

Returns the minimum required number.

```php
Zaphyr\Utils\Math::min(5, 10); // 10
Zaphyr\Utils\Math::min(20, 10); // 20
```

#### max

Returns the maximum required number.

```php
Zaphyr\Utils\Math::max(5, 10); // 5
Zaphyr\Utils\Math::max(10, 20); // 10
```

#### isInteger

Determines whether the given number is an integer.

```php
Zaphyr\Utils\Math::isInteger(1); // true
Zaphyr\Utils\Math::isInteger(1.2); // false
```

#### isFloat

Determines whether the given number is a float.

```php
Zaphyr\Utils\Math::isFloat(1.2); // true
Zaphyr\Utils\Math::isFloat('1 '); // false
```

#### isInRange

Determines whether the given number is in a given range.

```php
$number = 50;
$min = 10;
$max = 100;

Zaphyr\Utils\Math::isInRange($number, $min, $max); // true
Zaphyr\Utils\Math::isInRange($number, $min, 20); // false
```

#### isOutOfRange

Determines whether the given number is out of a given range.

```php
$number = 50;
$min = 10;
$max = 20;

Zaphyr\Utils\Math::isOutOfRange($number, $min, $max); // true
Zaphyr\Utils\Math::isOutOfRange($number, $min, 100); // false
```

#### isEven

Determines whether the given number is even.

```php
Zaphyr\Utils\Math::isEven(20); // true
Zaphyr\Utils\Math::isEven(0.5); // true
Zaphyr\Utils\Math::isEven(21); // false
```

#### isOdd

Determines whether the given number is odd.

```php
Zaphyr\Utils\Math::isOdd(21); // true;
Zaphyr\Utils\Math::isOdd(21.25); // true
Zaphyr\Utils\Math::isOdd(20); // false
```

#### isPositive

Determines whether the given number is positive.

```php
Zaphyr\Utils\Math::isPositive(1); // true
Zaphyr\Utils\Math::isPositive(0); // true
Zaphyr\Utils\Math::isPositive(0, false); // false
Zaphyr\Utils\Math::isPositive(-1); // false
```

#### isNegative

Determines whether the given number is negative.

```php
Zaphyr\Utils\Math::isNegative(-1); // true
Zaphyr\Utils\Math::isNegative(-0.5); // true
Zaphyr\Utils\Math::isNegative(0); // false
```

## String

Useful string helper methods.

#### toAscii

Transliterates a UTF-8 value to ASCII.

```php
Zaphyr\Utils\Str::toAscii('Τάχιστη αλώπηξ'); // 'Taxisth alwphx'
```

#### toArray

Converts a string to an array.

```php
Zaphyr\Utils\Str::toArray('Hello'); // ['H', 'e', 'l', 'l', 'o']
```

#### toBool

Returns the boolean representation of a string.

```php
Zaphyr\Utils\Str::toBool('true'); // true
Zaphyr\Utils\Str::toBool('Off'); // false
```

The following strings are converted to a boolean value:

| true      | false     |
|-----------|-----------|
| `"true"`  | `"false"` |
| `1`       | `0`       |
| `"on"`    | `"off"`   |
| `"On"`    | `"Off"`   |
| `"ON"`    | `"OFF"`   |
|           | `"no"`    |
|           | `""`      |

#### beginsWith

Determines if the given string starts with a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'C'); // true
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'c'); // false

// case insensitive
Zaphyr\Utils\Str::beginsWith('Case sensitive', 'c', false); // true
```

#### endsWith

Determines if a given string ends with a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::endsWith('Case sensitive', 'e'); // true
Zaphyr\Utils\Str::endsWith('Case sensitive', 'E'); // false

// case insensitive
Zaphyr\Utils\Str::endsWith('Case sensitive', 'E', false); // true
```

#### contains

Determines if a given string contains a given substring.

```php
// case sensitive
Zaphyr\Utils\Str::contains('Case sensitive', 'C'); // true
Zaphyr\Utils\Str::contains('Case sensitive', 'c'); // false

// case insensitive
Zaphyr\Utils\Str::contains('Case sensitive', 'c', false); // true
```

#### containsAll

Determines if a given string contains all array substring.

```php
// case sensitive
Zaphyr\Utils\Str::containsAll('Case sensitive', ['C', 's']); // true
Zaphyr\Utils\Str::containsAll('Case sensitive', ['c', 'S']); // false

// case insensitive
Zaphyr\Utils\Str::containsAll('Case sensitive', ['c', 'S'], false); // true
```

#### lower

Converts the given string to lower-case.

```php
Zaphyr\Utils\Str::lower('FOO'); // 'foo'
```

#### lowerFirst

Converts the first character of a given string to lower-case.

```php
Zaphyr\Utils\Str::lowerFirst('FOO'); // 'fOO'
```

#### upper

Converts the given string top upper-case.

```php
Zaphyr\Utils\Str::upper('foo'); // 'FOO'
```

#### upperFirst

Converts the first character of a given string to upper-case.

```php
Zaphyr\Utils\Str::upperFirst('foo'); // 'Foo'
```

#### limit

Limits the number of characters in a string.

```php
Zaphyr\Utils\Str::limit('Foobar', 3); // 'Foo...'
Zaphyr\Utils\Str::limit('Foobar', 3, ' ->'); // 'Foo ->'
```

#### limitSafe

Limits the length of a string, taking into account not cutting words.

```php
Zaphyr\Utils\Str::limitSafe('Foo bar', 5); // 'Foo...'
Zaphyr\Utils\Str::limitSafe('Foo bar', 5, ' ->')); // 'Foo ->'
```

#### firstPos

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

#### lastPos

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

#### replace

Replaces part of a string by a matching pattern.

```php
Zaphyr\Utils\Str::replace('foo', 'f[o]+', 'bar'); // 'bar'
```

#### stripWhitespace

Strips all whitespaces from the given string.

```php
Zaphyr\Utils\Str::stripWhitespace('Foo bar'); // 'Foobar
```

#### insert

Inserts a string inside a string at a given position.

```php
Zaphyr\Utils\Str::insert('foo', 'bar', 3); // 'Foobar'
```

#### equals

Determines whether two strings are equal.

```php
// case sensitive
Zaphyr\Utils\Str::equals('Case sensitive', 'Case sensitive'); // true
Zaphyr\Utils\Str::equals('Case sensitive', 'case sensitive'); // false

// case insensitive
Zaphyr\Utils\Str::equals('Case sensitive', 'case sensitive', false); // true
```

#### length

Returns the string length.

```php
Zaphyr\Utils\Str::length('Foo'); // 3
```

#### escape

Escapes a string.

```php
Zaphyr\Utils\Str::escape('<h1>Hello world</h1>'); // '&lt;h1&gt;Hello world&lt;/h1&gt;'
```

#### title

Converts the given string to title case.

```php
Zaphyr\Utils\Str::title('hello world'); // 'Hello World'
```

#### slug

Generates a URL friendly "slug" from a given string.

```php
Zaphyr\Utils\Str::slug('Hello World'); // 'hello-world'
```

#### studly

Converts a value to studly caps case.

```php
Zaphyr\Utils\Str::studly('Sho-ebo-x'); // 'ShoEboX'
Zaphyr\Utils\Str::studly('Sho -_- ebo -_ - x')); // 'ShoEboX'
```

#### camel

Converts a value to camel case.

```php
Zaphyr\Utils\Str::camel('Hello world-whats up'); // 'helloWorldWhatsUp'
```

#### snake

Converts a string to snake case.

```php
Zaphyr\Utils\Str::snake('HelloWorld'); // 'hello_world'
```

## Timezone

Returns all available time zones with UTC time.

#### getAllTimezones

Returns all time zones with UTC time as the key.

```php
Zaphyr\Utils\Timezone::getAllTimezones();
// [
//    '(UTC-11:00) Midway Island' => 'Pacific/Midway',
//    '(UTC-11:00) Samoa' => 'Pacific/Samoa',
//    …
//    '(UTC+12:00) Wellington' => 'Pacific/Auckland',
//    '(UTC+13:00) Nuku\'alofa' => 'Pacific/Tongatapu',
// ]
```
