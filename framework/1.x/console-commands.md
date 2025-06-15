# Console Commands

The ZAPHYR framework offers a set of built-in commands to help you manage and maintain your application. You can execute
these commands from the command-line interface (CLI) by running:

```bash
php bin/zaphyr
```

## Available commands

To view a list of all available commands, run the following in your terminal:

```bash
php bin/zaphyr list
```

This will display all registered commands, grouped by category.

To get detailed information about a specific command, including its options and usage, use the `-h` (help) flag:

```bash
php bin/zaphyr <command-name> -h
```

For example, to view help for the `app:environment` command:

```bash
php bin/zaphyr app:environment -h
```

## Create commands

You can generate new console commands using the `create:command` generator. This command will scaffold a new command
class
for your application:

```bash
php bin/zaphyr create:command MyCommand
```

By default, the new class will be placed in the `app/Commands` directory. If this directory does not exist, it will be
created automatically.

#### Custom namespace

To generate the command in a custom namespace, use the `--namespace` option. For example, to place the command under the
`App\Console\Commands` namespace:

```bash
php bin/zaphyr create:command MyCommand --namespace="App\Console\Commands"
```

Once generated, you can find your new command class in the corresponding directory structure under the specified
namespace:

```php
<?php

declare(strict_types=1);

namespace App\Commands;

use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;
use Zaphyr\Framework\Console\Commands\AbstractCommand;

#[AsCommand(name: 'app:example', description: 'An example command')]
class MyCommand extends AbstractCommand
{
    /**
     * {@inheritdoc}
     */
    protected function configure(): void
    {
        $this->addArgument('name', InputOption::VALUE_REQUIRED, 'The name');
    }

    /**
     * {@inheritdoc}
     */
    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $name = $input->getArgument('name');

        $output->writeln('<info>Hello ' . $name . '.</info>');

        return self::SUCCESS;
    }
}
```

If you’ve used Symfony Console before, this will feel very familiar. That’s because ZAPHYR uses the Symfony Console
component under the hood to handle console commands. You can leverage all of Symfony Console’s features in your own
command classes, including arguments, options, and even subcommands.

For a complete overview of what’s possible, refer to the
[Symfony Console documentation](https://symfony.com/doc/current/console.html#creating-a-command).

#### Extended command features

ZAPHYR also provides a base class `Zaphyr\Framework\Console\Commands\AbstractCommand`, which adds some convenient
features to enhance your command classes. These include:

- `call`: Executes another console command.
- `callSilent`: Executes a command without outputting anything.
- `confirmToProceed`: Prompts the user for confirmation before continuing, ideal for dangerous or destructive actions.

#### Calling other commands

You can use the `call` method to execute another console command from within your command class. This is useful when you
want to delegate part of your command's functionality to another command.

The `call` method accepts the command name and an optional array of arguments and options. It also takes an
`OutputInterface instance to handle the output of the called command.

```php
protected function execute(InputInterface $input, OutputInterface $output): int
{
    // Call another command
    $this->call('my:command', $output);
    
    // Call a command with arguments
    $this->call(['command' => 'my:command', 'arg1' => 'value1'], $output);
    
    // Call a command with options (e.g., disable interaction)
    $this->call(['command' => 'my:command', '--no-interaction' => true], $output);
    
    // …
}
```

#### Calling commands silently

If you want to execute a command without displaying any output, you can use the `callSilent` method. This is
particularly useful when chaining internal commands where the output is not needed.

```php
protected function execute(InputInterface $input, OutputInterface $output): int
{
    // Call a command silently
    $this->callSilent('my:command');
 
    // Call with arguments silently
    $this->callSilent(['command' => 'my:command', 'arg1' => 'value1']);
    // …
}
```

#### Confirmation before proceeding

For critical or potentially destructive operations, it's good practice to prompt the user for confirmation before
continuing. You can achieve this using the `confirmToProceed` method.

This method displays a confirmation prompt and waits for the user to type `yes` or `no`. If the user confirms, the
command
proceeds; otherwise, it exits early.

The third parameter is a boolean that controls whether the confirmation prompt should be enforced. This is particularly
useful when tailoring behavior to different environments, for example:

- In **development**, you might want to skip the confirmation to streamline testing (`false`).
- In **production**, you likely want to enforce the confirmation to prevent accidental data loss (`true`).

```php
protected function execute(InputInterface $input, OutputInterface $output): int
{
    if ($this->confirmToProceed($input, $output, true, 'Do you want to continue?')) {
        // Proceed with the command
    }
    
    // …
}
```

By using this approach, you can ensure safety in sensitive environments while maintaining flexibility during
development.

## Configure commands

You can control which console commands are available in your ZAPHYR application by editing the `config/console.yaml`
configuration file.

By default, all command classes located in the `app/Commands` directory are automatically registered. However, if you
prefer explicit control, you can list the desired command classes manually using the commands section:

```yaml
commands:
    - App\Commands\MyCommand
    - App\Commands\AnotherCommand
```

If you want to exclude certain commands from being available, you can list them in the `commands_ignore` section. This
is useful if you have commands that should only run in specific environments or are not meant for public execution.

```yaml
commands_ignore:
    - App\Commands\MyCommand
```

Any command listed here will be ignored even if it exists in the default directory or is explicitly registered.
