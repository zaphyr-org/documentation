# Mail

A mail API over the popular Symfony Mailer.

## Installation

To get started, install the mail repository via the [Composer](https://getcomposer.org/) package manager:

```console
composer require zaphyr-org/mail
```

## Configuration

The mail service is a wrapper over the popular [Symfony Mailer](https://symfony.com/doc/6.2/mailer.html) which makes
sending emails a breeze.

Before we can use the mailer service, we need to pass a Symfony `Mailer` instance to it. How to configure the Symfony
Mailer instance can be seen [here](https://symfony.com/doc/6.2/mailer.html#transport-setup).

```php
$symfonyTransport = Symfony\Component\Mailer\Transport::fromDsn('smtp://user:pass@smtp.example.com:port');
$symfonyMailer = new Symfony\Component\Mailer\Mailer($symfonyTransport);

$mailer = new Zaphyr\Mail\Mailer($symfonyMailer);
```

## Send emails with closure

After we have configured our mailer service, we can start sending emails. One way to send emails is to pass a closure
to the `send()` method.

```php
$mailer->send(
    [
        Zaphyr\Mail\Mailer::VIEW_HTML => '/path/to/email/templates/welcome.html'
    ],
    [
        'name' => 'John Doe'
    ],
    function(Zaphyr\Mail\EmailBuilder $message) {
        $message
            ->from('noreply@example.com')
            ->to('john@doe.com')
            ->subject('Welcome John Doe');
    }
);
```

The first argument (line 2-4) we pass to the method is an array in which we define the email template to use. In our
example this is a `html` template.

As second argument (line 5-7) we pass the placeholder variables to be replaced in the html template. How template files
have to be structured can be read [here](#templates).

As a third argument (line 8-13) we pass a closure to the `send()` method, in which we define the message methods
(e.g. sender, subject and receiver).

Both `html` and `plain text` email templates can be passed to the `send()` method as the first argument. If the client
is not able to receive html emails, a plain text fallback can be sent with the email.

```php
$mailer->send(
    [
        Zaphyr\Mail\Mailer::VIEW_HTML => '/path/to/email/templates/welcome.html',
        Zaphyr\Mail\Mailer::VIEW_TEXT => '/path/to/email/templates/welcome.txt'
    ],
    [
        'name' => 'John Doe'
    ],
    function(Zaphyr\Mail\EmailBuilder $message) {
        $message
            ->from('noreply@example.com')
            ->to('john@doe.com')
            ->subject('Welcome John Doe');
    }
);
```

## Send emails with mailable object

Another way to send emails is to use mailable objects. Mailable objects are classes that extend the
`Zaphyr\Mail\AbstractMailable`. The advantage of using mailable objects is that the email logic is encapsulated in a
mailable object and can be reused at any time.

So the first thing we do is create our mailable object:

```php
class WelcomeMail extends Zaphyr\Mail\AbstractMailable
{
    /**
     * @param string $email
     * @param string $name
     */
    public function __construct(protected string $email, protected string $name)
    {
    }

    /**
     * {@inheritdoc}
     */
    public function build(): void
    {
        $this
            ->from('noreply@example.com')
            ->to($this->email)
            ->subject('Welcome ' . $this->name)
            ->view(
                [
                    Zaphyr\Mail\Mailer::VIEW_HTML => '/path/to/email/templates/welcome.html',
                ],
                [
                    'name' => $this->name,
                ]
            );
    }
}
```

As you can see, all send logic of the email takes place within the `build()` method.

To send the email, we simply pass the mailable object to the `send()` method:

```php
$mailer->send(new WelcomeMail('john@doe.com', 'John Doe'));
```

## Templates

The mail service uses a lightweight, easy-to-use template engine by default. All the template engine can do is read
template files and replace the placeholders in them. You can read more about the template engine
[here](/docs/repositories/latest/utils#template).

A simple html email template looks like this:

```html
// welcome.html
<h1>Welcome %name%!</h1>
<p>Nice to have you aboard.</p>
```

A text email template thus looks like this:

```text
// welcome.txt
Welcome %name%!

Nice to have you aboard.
```

### Integrate third party template engines

If the provided template engine of this mail service is not sufficient, any template engine can be used for rendering
the email templates. How a third-party template engine is integrated into the mail service is shown in the following
example using [twig](https://twig.symfony.com/):

```php
class TwigView implements Zaphyr\Mail\Contracts\ViewInterface {
    /**
     * @param Twig\Environment $twig
     */
    public function __construct(protected Twig\Environment $twig)
    {
    }

    /**
     * {@inheritdoc}
     */
    public function render(string $path, array $data = []): string
    {
        try {
            return $this->twig->render($path, $data);
        } catch (Throwable $exception) {
            throw new Zaphyr\Mail\Exceptions\MailerException($exception->getMessage());
        }
    }
}

$symfonyTransport = Symfony\Component\Mailer\Transport::fromDsn('smtp://localhost');
$symfonyMailer = new Symfony\Component\Mailer\Mailer($symfonyTransport);

$twigFilesystemLoader = new Twig\Loader\FilesystemLoader('/path/to/email/templates');
$twigEnvironment = new Twig\Environment($twigFilesystemLoader);
$twigView = new TwigView($twigEnvironment);

$mailer = new Zaphyr\Mail\Mailer($symfonyMailer, $twigView);
```

After twig has been passed as template engine to the mail service the `send()` method will automatically render all
email templates with twig:

```php
$mailer->send(

    [
        Zaphyr\Mail\Mailer::VIEW_HTML => 'welcome.html.twig',
        Zaphyr\Mail\Mailer::VIEW_TEXT => 'welcome.txt.twig',
    ],
    //…
);
```

## Always methods

The mail service comes with a few always methods that can be used to set default values for the email. These methods
are called before the closure or the `build()` method of the mailable object is called.

### Always from

Sets the sender of the email for all emails sent with the mail service:

```php
$mailer->alwaysFrom('alwaysFrom@example.com');
```

### Always reply to

Sets the reply to address for all emails sent with the mail service:

```php
$mailer->alwaysReplyTo('alwaysReplyTo@example.com');
```

### Always return path

Sets the return path for all emails sent with the mail service:

```php
$mailer->alwaysReturnPath('alwaysReturnPath@example.com')
```

### Always to

Sets the receiver of the email for all emails sent with the mail service:

```php
$mailer->alwaysTo('alwaysTo@example.com');
```

### Overwrite always methods

The always methods can be overwritten within a closure or mailable object. For example, if you want to define a different
sender for a particular email, you can overwrite this within the closure or mailable object:

```php
$mailer->alwaysFrom('alwaysFrom@example.com');

$mailer->send(
    //…
    function(Zaphyr\Mail\EmailBuilder $message) {
        $message
          ->from('overwriteFrom@example.com')   // overwrites alwaysFrom
          //…
    }
);
```

In the above example, the email sender will be `overwriteFrom@example.com` instead of `alwaysFrom@example.com`.
