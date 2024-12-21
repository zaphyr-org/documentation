# Validate

Easy to use, highly customizable validator.

## Installation

To get started, install the validate repository via the [Composer](https://getcomposer.org/) package manager:

```bash
composer require zaphyr-org/validate
```

## Basic validation

Validating input data in a web application is a common task. Data in forms needs to be validated before it is written
to a database, API or web service. ZAPHYR offers a robust validation service with a set of validation rules for these
purposes.

So let's dive right into the code and create a basic validation:

```php
$validator = new Zaphyr\Validate\Validator();

$inputs = [
    'name' => 'John Doe',
    'email' => 'jonh@doe.com',
],

$rules = [
    'name' => 'required',
    'email' => 'required|email'
];

$validator->validate($inputs, $rules);

$validator->isValid();
```

In line 1 we first create our `Zaphyr\Validate\Validator` object. This is responsible for the validation of our input
data in the further process.

In line 3-6 we have our input data. In a real web application this will most likely be `$_POST` or `$_GET` data of
a form.

in line 8-11 we then define the validation rules against which our input data should be validated. In this case we
check if the name and a valid email address was passed. As you can see in line 10 you can concatenate validation rules
for a single input by using a "pipe" character. But for now we leave it like this. You will learn more about
[validation rules in a later section](#validation-rules).

In line 13 we finally check our input data against the validation rules.

Last but not least, we call the `isValid()` method on line 15. This method returns `true` if all inputs are valid and
`false` otherwise.

## Validation error messages

What would be a decent validator if it would not return helpful validation error messages in case of a failed validation?

So if your validation fails, you can use the `errors()` method to display all validation errors. This method returns a
`Zaphyr\Validate\MessageBag` instance containing all error messages:

```php
/** @var Zaphyr\Validate\MessageBag $errors **/
$errors = $validator->errors();
```

### Get all validation error messages

If you want to get all potential validation errors, simply use the `all()` method. This method returns an array with
all existing validation errors. If no validation errors were detected, this method returns an empty array:

```php
$validator->errors()->all();
```

### Get the first validation error message

The `first()` method returns the first validation error as a string. If a parameter is specified, the first
validation error for the given parameter is displayed:

```php
$validator->errors()->first();
$validator->errors()->first('name');
```

### Get a specific validation error message

If you want to get the validation errors for a specific input, simply use the `get()` method. This method returns an
array of all validation errors for the input field passed as parameter:

```php
$validator->errors()->get('name');
```

### Determine whether a validation error message exists

Use the `has()` method to check if a given input has validation errors. This method returns `true` if there are
validation errors for the given parameter and false otherwise:

```php
$validator->errors()->has('name');
```

### Determine whether the validation error messages are empty

If you want to check if there are any error messages at all, you can simply call the `isEmpty()` method. This method
returns `true` if there are no validation errors and `false` otherwise:

```php
$errors->isEmpty();
```

### Create custom validation error messages

Sometimes you may want to output custom validation error messages for a validation rule, you can do this as follows:

```php
$inputs = [
    'name' => 'John Doe'
];

$rules = [
    'name' => 'required'
];

$customMessages = [
    'required' => 'Please provide a valid name'
];

$validator->validate($inputs, $rules, $customMessages);
```

Now, whenever a `required` validation fails, the message "Please provide a valid name" is displayed instead of the
default message "This name field is required".

### Define custom validation error messages field names

Maybe you just want to name a field name individually. This is also possible with simple steps:

```php
$inputs = [
    'password_repeat' => 'Top secret',
];

$rules = [
    'password_repeat' => 'required',
];

$customFields = [
    'password_repeat' => 'password confirmation'
];

$validator->validate($inputs, $rules, [], $customFields);
```

Congrats, you have renamed the `password_repeat` field e.g. Now the validation error message always shows
"The password confirmation field is required" instead of "The password repeat field is required".

## Validation rules

The ZAPHYR validation service comes already shipped with a huge number of validation rules. All validation rules are
stored in separate classes and can be found under the `Zaphyr\Validate\Rules` namespace.

| Rule                                                                                                                                       | Description                                                                                                                                                                                                                                                                                                                                                 |
|--------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `'active_url'`                                                                                                                             | The field under validation must have a valid A or AAAA record according to the `dns_get_record` php function.                                                                                                                                                                                                                                               |
| `'alpha_chars'`                                                                                                                            | The field under validation must be entirely alphabetic characters.                                                                                                                                                                                                                                                                                          |
| `'alpha_dash'`                                                                                                                             | The field under validation may have alpha-numeric characters, as well as dashes and underscores.                                                                                                                                                                                                                                                            |
| `'alpha_num'`                                                                                                                              | The field under validation must be entirely alpha-numeric characters.                                                                                                                                                                                                                                                                                       |
| `'array'`                                                                                                                                  | The field under validation must be a php `array`.                                                                                                                                                                                                                                                                                                           |
| `'ascii'`                                                                                                                                  | The field under validation must be entirely ASCII characters.                                                                                                                                                                                                                                                                                               |
| `'bail'`                                                                                                                                   | Sometimes you may wish to stop running validation rules on an attribute after the first validation failure. To do so, assign the bail rule to the attribute                                                                                                                                                                                                 |
| `'between_array:1,3'`                                                                                                                      | The field under validation must be an array with the size between the given min (e.g. `1`) and max (e.g. `3`).                                                                                                                                                                                                                                              |
| `'between_number:100,500'`                                                                                                                 | The field under validation must be a number with the size between the given min (e.g. `100`) and max (e.g. `500`).                                                                                                                                                                                                                                          |
| `'between_string:1,7'`                                                                                                                     | The field under validation must be a string with the length between the given min (e.g. `1`) and max (e.g. `7`).                                                                                                                                                                                                                                            |
| `'bool'`                                                                                                                                   | The field under validation must be able to be cast as a boolean. Accepted input are `true`, `false`, `1`, `0`, `"1"`, and `"0"`.                                                                                                                                                                                                                            |
| `'checked'`                                                                                                                                | The field under validation must be checked.                                                                                                                                                                                                                                                                                                                 |
| `'date_after:yesterday'`, `'date_after:2022-09-13'`, `'date_after:start_date'`, `'date_after:09:41:00'`                                    | The field under validation must be a value after a given date. Instead of passing a date, you may specify another field name (e.g. `start_date`) to compare against the date of the specified field. This validation rule supports all formats supported by php's [DateTimeInterface](https://www.php.net/manual/en/class.datetimeinterface.php) interface. |
| `'date_before:tomorrow'`, `'date_before:2022-09-13'`, `'date_before:end_date'`, `'date_before:09:41:00'`                                   | The field under validation must be a value before a given date. Instead of passing a date, you may specify another field name (e.g. `end_date`) to compare against the date of the specified field. This validation rule supports all formats supported by php's [DateTimeInterface](https://www.php.net/manual/en/class.datetimeinterface.php) interface.  |
| `'date_equals:today'`, `'date_equals:2022-09-13'`, `'date_equals:start_date'`, `'date_equals:09:41:00'`                                    | The field under validation must be equal to the given date. Instead of passing a date, you may specify another field name (e.g. `start_date`) to compare against the date of the specified field. This validation rule supports all formats supported by php's [DateTimeInterface](https://www.php.net/manual/en/class.datetimeinterface.php) interface.    |
| `'date_format:d.m.Y'`, `'date_format:Y-m-d H:i:s'`, `'date_format:Y-m-d\TH:i:se'`, `'date_format:H:i:s'`                                   | The field under validation must be a valid, non-relative date. This validation rule supports all formats supported by php's [DateTimeInterface](https://www.php.net/manual/en/class.datetimeinterface.php) interface.                                                                                                                                       |
| `'date_time'`                                                                                                                              | The field under validation must match the given format. You should use either `date_time` or `date_format` when validating a field, not both. This validation rule supports all formats supported by php's [DateTimeInterface](https://www.php.net/manual/en/class.datetimeinterface.php) interface.                                                        |
| `'different:old_password'`                                                                                                                 | The field under validation must have a different value than the specified field (e.g. `old_password`).                                                                                                                                                                                                                                                      |
| `'digits:4'`                                                                                                                               | The field under validation must be numeric and must have an exact length of value (e.g. `4`).                                                                                                                                                                                                                                                               |
| `'email'`, `'email:dns'`, `'email:filter'`, `'email:filter_unicode'`, `'email:spoof'`, `'email:strict'`, `'email::dns,filter,spoof,check'` | The field under validation must be formatted as an e-mail address. The `email` rule is based on the [egulias/email-validator](https://github.com/egulias/EmailValidator) package.                                                                                                                                                                           |
| `'ends_without:foo'`, `'ends_without:foo,bar'`                                                                                             | The field under validation must be a string that does not end with the given value(s) (e.g. `foo`, `bar`).                                                                                                                                                                                                                                                  |
| `'ends_with:foo'`, `'ends_with:foo,bar'`                                                                                                   | The field under validation must be a string that does end with the given value(s) (e.g. `foo`, `bar`).                                                                                                                                                                                                                                                      |
| `'integer'`                                                                                                                                | The field under validation must be an integer.                                                                                                                                                                                                                                                                                                              |
| `'ip'`                                                                                                                                     | The field under validation must be an IP address.                                                                                                                                                                                                                                                                                                           |
| `'ipv4'`                                                                                                                                   | The field under validation must be an IPv4 address.                                                                                                                                                                                                                                                                                                         |
| `'ipv6'`                                                                                                                                   | The field under validation must be an IPv6 address.                                                                                                                                                                                                                                                                                                         |
| `'json'`                                                                                                                                   | The field under validation must be a valid JSON string.                                                                                                                                                                                                                                                                                                     |
| `'mac'` <span class="badge__available">Available since v2.0.0</span>                                                                       | The field under validation must be a valid MAC address.                                                                                                                                                                                                                                                                                                     |
| `'max_array:2'`                                                                                                                            | The field under validation must be an array less than or equal to a maximum value (e.g. `2`).                                                                                                                                                                                                                                                               |
| `'max_number:100'`                                                                                                                         | The field under validation must be a number less than or equal to a maximum value (e.g. `100`).                                                                                                                                                                                                                                                             |
| `'max_string:7'`                                                                                                                           | The field under validation must be a string less than or equal to a maximum length (e.g. `7`).                                                                                                                                                                                                                                                              |
| `'min_array:2'`                                                                                                                            | The field under validation must be an array with a minimum value (e.g. `2`).                                                                                                                                                                                                                                                                                |
| `'min_number:5'`                                                                                                                           | The field under validation must be a number with a minimum value (e.g. `5`).                                                                                                                                                                                                                                                                                |
| `'min_string:8'`                                                                                                                           | The field under validation must be a string with a minimum length (e.g. `8`).                                                                                                                                                                                                                                                                               |
| `'not_regex:/[yz]/i'`                                                                                                                      | The field under validation must not match the given regular expression (e.g. `/[yz]/i`).                                                                                                                                                                                                                                                                    |
| `'nullable'`                                                                                                                               | The field under validation may be `null`. This is particularly useful when validating primitive such as strings and integers that can contain `null` values.                                                                                                                                                                                                |
| `'number'`                                                                                                                                 | The field under validation must be a number.                                                                                                                                                                                                                                                                                                                |
| `'regex:/^[a-z]+$/i'`                                                                                                                      | The field under validation must match the given regular expression (e.g `/^[a-z]+$/i`).                                                                                                                                                                                                                                                                     |
| `'required'`                                                                                                                               | The field under validation must be present in the input data and not empty.                                                                                                                                                                                                                                                                                 |
| `'same:password'`, `'same:password_confirm'`                                                                                               | The given field must match the field under validation (e.g. `password`, `password_confirm`).                                                                                                                                                                                                                                                                |
| `'size_array:2'`                                                                                                                           | The field under validation must be an array whose size matching the given value (e.g. `2`).                                                                                                                                                                                                                                                                 |
| `'size_number:5'`                                                                                                                          | The field under validation must be a number whose size matching the given value (e.g. `5`).                                                                                                                                                                                                                                                                 |
| `'size_string:7'`                                                                                                                          | The field under validation must be a string whose length matching the given value (e.g. `7`).                                                                                                                                                                                                                                                               |
| `'starts_without:foo'`, `'starts_without:foo,bar'`                                                                                         | The field under validation must be a string that does not start with the given value(s) (e.g. `foo`,`bar`).                                                                                                                                                                                                                                                 |
| `'starts_with:foo'`, `'starts_with:foo,bar'`                                                                                               | The field under validation must be a string that does start with the given value(s) (e.g. `foo`,`bar`).                                                                                                                                                                                                                                                     |
| `'string'`                                                                                                                                 | The field under validation must be a string. If you would like to allow the field to also be null, you should assign the `nullable` rule to the field.                                                                                                                                                                                                      |
| `'timezone'`                                                                                                                               | The field under validation must be a valid timezone identifier.                                                                                                                                                                                                                                                                                             |
| `'url'`                                                                                                                                    | The field under validation must be a valid URL.                                                                                                                                                                                                                                                                                                             |

As described in the example above, you can concatenate multiple validation rules with the "pipe" character and apply
them to a single input. The validation rules are then processed in the defined order from left to right:

```php
$rules = [
    'name' => 'required|between_string:3,255',
    'email' => 'required|email:dns,spoof',
];
```

## Custom validation rules

If the supplied validation rules are not sufficient, you can create your custom validation rules. First of all you need
to create a new validation class, which extends the `Zaphyr\Validate\Rules\AbstractRule`:

```php
class MyCustomRule extends Zaphyr\Validate\Rules\AbstractRule
{
    /**
     * {@inheritdoc}
     */
    public function validate(string $field, mixed $value, array $parameters, array $inputs): bool
    {
        // add your custom validation rule logic here
    }
}
```

Now add your custom rule to the validator:

```php
$validator->addRule('customRule', new MyCustomRule());
```

After your custom validation rule has been added to the `Validator` instance, you can use it:

```php
$inputs = ['name' => 'John Doe'];
$rules = ['name' => 'customRule'];

$validator->validate($inputs, $rules);
```

> [!NOTE]
> Have a look at the [included rule classes](https://github.com/zaphyr-org/validate/blob/master/src/Rules).
> They offer perfect examples for developing your custom validation rules.

> [!NOTE]
> Of course, custom rules are not known to the default error messages and are therefore not taken into account in the
> translation. For custom rules you have to define custom error messages as well. How you can store your own error
> messages in translation files can be found in [here](#validation-error-messages-translations).

## Hooks

Sometimes it can be helpful to perform a certain action immediately before or after validation. This validator service
provides two useful hook methods for this case: `addBeforeValidationHook()` and `addAfterValidationHook()`.

### Add before validation hook

Execute an action immediately **before** validation:

```php
$validator->addBeforeValidationHook(function () {
    // add your before validation hook logic here
});
```

### Add after validation hook

Execute an action immediately **after** validation:

```php
$validator->addAfterValidationHook(function () {
    // add your after validation hook logic here
});
```

## Validation error messages translations

As you may have noticed, the validate service can output error messages in different languages. By default, the
`Validator` instance returns error messages in American English.

You can also instruct the validate service to output the error messages in other languages. Currently, this validation
service can return the error messages in American English as well as in German. If you want to output error messages in
German, you have to configure the `Validator` instance as follows:


```php
$locale = 'de';
$validator = new \Zaphyr\Validate\Validator($locale);
```

Now the `$validator->errors()` method returns all error messages in german language.

### Create custom translated error messages

However, you can also create your own translation files and instruct your `Validator` instance to use this translation
file for the error messages.

In our example we want to output the error messages in polish. So we first create a directory where you can store your
custom translation files. As a starting point just copy the `resources/translations/en/validation.json` file from the
vendor directory of the validation service repository.

So you should now have the following `validation.json` file:

```json
// path/to/your/translations/pl/validation.json:
{
  "required": "To pole jest obowiązkowe"
}
```

Now create a new `Validator` instance, pass the locale you want to use for the error messages and also the path where
your custom translation files are located as a second parameter:

```php
$validator = new Zaphyr\Validate\Validator('pl', '/path/to/your/translations');
```

Now the `$validator->errors()` method returns all error messages in polish language. If not all error messages have been
translated in your custom translation file, the `Validator` instance uses the default American English translations as
a fallback.

By default, your custom translation file must be named `validation.json`. But you can rename it, for example to
`messages.json`. But then you have to pass a third parameter to your `Validator` instance, in this case the name of the
translation file:

```php
// Name of your translation file: path/to/your/translations/pl/messages.json
$validator = new Zaphyr\Validate\Validator('pl', './resources/translations', 'messages');
```

### Create custom translated input error messages

The validate service also offers you the possibility to translate error messages for a specific input field.
For example, if you want to customize the required error message for the input field which is named `password`,
extend your `validation.json` file as follows:

```json
// path/to/your/translations/pl/validation.json:
{
  "_custom": {
    "password": {
      "required": "Wprowadź hasło"
    }
  }
}
```

If the input field named `password` is now empty and the required validation fails, your `error()` method for the
input field named `password` will now return "Wprowadź hasło" as an error message:

```php
$inputs = [
    'password' => '',
];

$rules = [
    'password' => 'required',
];

$validator->validate($inputs, $rules);
$validator->errors()->first('password'); // Wprowadź hasło
```

### Create custom translated input error field names

Maybe you just want to translate the name of a certain input field. The validate service offers you the possibility
for this as well. If you have e.g. an input field with the name `name` and you want to display it in the error message
as `username`, you can simply define your own names in your `valdation.json` under the key `_fields` (line 3):

```json
// path/to/your/translations/pl/validation.json:
{
  "_fields": {
    "name": "nazwa użytkownika"
  },
  "required": "Pole %field% jest wymagane"
}
```

If the `name` field is now empty and the validator throws an error message for this field, you will get the message
"Pole **nazwa użytkownika** jest wymagane" instead of "Pole **name** jest wymagane":

```php
$inputs = [
    'name' => '',
];

$rules = [
    'name' => 'required',
];

$validator->validate($inputs, $rules);
$validator->errors()->first('name'); // Pole nazwa użytkownika jest wymagane
```
