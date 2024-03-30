# Contribute

Contributions via pull requests on [GitHub](https://github.com/zaphyr-org) are welcome and will be fully credited.

## Coding style

The ZAPHYR project is committed to maintaining a high standard of code quality and readability. To achieve this, the
project strictly adheres to the [PSR-12](https://www.php-fig.org/psr/psr-12/) coding style guideline. By following
PSR-12, ZAPHYR ensures consistency and uniformity in its codebase, making it easier for developers to collaborate and
maintain the project. The easiest way to apply the conventions is to install
[PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer).

## Running tests

Your pull request won't be accepted if it doesn't have tests. Please use the composer test command to run tests. This
command will run the [phpunit tests](https://phpunit.de/index.html), the [phpstan analyse](https://phpstan.org/) and
also checks for [PSR12 coding standard](https://www.php-fig.org/psr/psr-12/):

```bash
composer test
```

## Documentation

Document any change in behaviour. Make sure the `README.md` and any other relevant
[documentation](https://github.com/zaphyr-org/documentation) are kept up-to-date.

## Release cycle

Consider the release cycle. ZAPHYR tries to follow [SemVer v2.0.0](https://semver.org/). Randomly breaking public APIs
is not an option.

## Pull requests

One pull request per feature. If you want to do more than one thing, send multiple pull requests. Send coherent history.
Make sure each individual commit in your pull request is meaningful. If you had to make multiple intermediate commits
while developing, please [squash](https://www.git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Changing-Multiple-Commit-Messages)
them before submitting.

### Commit message rules

There are some rules for what a commit message should look like. Here is an example of a final commit message:

```bash
New: A super new feature

Most importantly, describe what is changed with the commit and not
what is has not been working (that is part of the bug report already).

* Bullet points are okay, too
* An asterisk is used for the bullet, it can be preceded by a single space.
```

### Summary line (first line)

A summary line starts with a keyword (e.g. `New`) and a brief summary of what the change does. Make sure to describe how
the behavior is now, not how it used to be - in the end, telling someone what was broken doesn't help anyone, you want
to tell what is working now.

Prefix the line with a keyword. Possible keywords are:

| Keyword    | 	Description                                                |
|------------|-------------------------------------------------------------|
| New        | 	A new feature or element. Also small additions             |
| Changed    | 	A change (which is not a bugfix) to an existing component  |
| Deprecated | 	A new deprecation of a feature or function                 |
| Removed    | 	Removal of a feature or function                           |
| Fixed      | 	A fix for a bug                                            |
| Security   | 	A security fix                                             |

### Description (Message body)

Here you can go into detail about the how and why of the change. It should be brief, but yet descriptive so people
reviewing your change get an idea what they need to look out for.
