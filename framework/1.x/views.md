# Views

Views are the visual components of your application, typicalliy used to return HTML responses.

## Rendering views

To render a view, you can use the `view()` helper function defined in the `app/functions.php` file.

```php
return view('welcome', ['name' => 'John Doe']);
```

This function internally uses the `Zaphyr\Utils\Template` class, which is part of the
[utils repository](/docs/repositories/latest/utils#template), to load and process the template.

The `view()`function returns an instance of `Zaphyr\Framework\Http\HtmlResponse`, which implements the
`Psr\Http\Message\ResponseInterface`. This allows it to be seamlessly used as an HTTP response within your application.

## View files location

By default, all view templates are stored in the `resources/views` directory of your application. For example, when
calling `view('welcome')`, the framework will automatically look for the corresponding template file at
`resources/views/welcome.html`.

## Template syntax

The view system is intentionally kept lightweight and is best suited for simple projects or quick prototypes. It only
supports static HTML files and basic variable replacement using string interpolation.

To include dynamic variables in your templates, wrap them with `%` signs. For example:

```html
<!-- resources/views/welcome.html -->
<h1>Hello %name%</h1>
```

When rendered with the `view()` function, the `%name%` placeholder will be replaced with the value provided in the
second argument:

```php
return view('welcome', ['name' => 'John Doe']);
```

The resulting HTML will be:

```html
<h1>Hello John Doe</h1>
```
