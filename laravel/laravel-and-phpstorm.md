# Make PhpStorm Laravel friendly

Both PhpStorm and Laravel are robust and complex systems that attempt to make developer's lives easier by providing an abundance of tools and features for writing software.

Most of the time, these systems work well together and you get the advantage of Laravel's advanced yet expressive featureset with the support of PhpStorm's rich functionality (code completion, inspections, etc.)

However, sometimes the two systems are not 100% compatible and extra configuration is needed to make them work together more seamlessly. The following notes explain steps to make this happen.


## PhpStorm Laravel Plugin
Install a plugin called *Laravel Plugin* in PhpStorm (via *Preferences* : *Plugins*).

Once installed, it should prompt you to enable the plugin whenever it detects you've open a Laravel project.

If it doesn't, you can manually enable it via *Preferences* : *Languages & Frameworks* : *PHP* : *Laravel* : *Enable plugin for this project*.

<img src='https://s3.amazonaws.com/making-the-internet/laravel-enable-plugin@2x.png' style='max-width:1000px;' alt='Enable Laravel Plugin'>


For a list of features this plugin adds, [see its info page](https://plugins.jetbrains.com/plugin/7532-laravel-plugin).

## laravel-ide-helper Package

>> "This package generates a file that your IDE understands, so it can provide accurate autocompletion. Generation is done based on the files in your project, so they are always up-to-date." -[ref](https://github.com/barryvdh/laravel-ide-helper)

Install the package [barryvdh/laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper):

```
$ composer require --dev barryvdh/laravel-ide-helper
```

In `app/Providers/AppServiceProvider.php` within the `register` method, add this code to register the ide-helper (on non-production environments only):

```php
if ($this->app->environment() !== 'production') {
    $this->app->register(\Barryvdh\LaravelIdeHelper\IdeHelperServiceProvider::class);
}
```

In `composer.json` in the `post-update-command` section, add the php artisan commands `ide-helper:generate`, `ide-helper:meta`, and `ide-helper:models --nowrite` like so:

```php
"post-update-cmd": [
    "Illuminate\\Foundation\\ComposerScripts::postUpdate",
    "php artisan ide-helper:generate",
    "php artisan ide-helper:meta",
    "php artisan ide-helper:models --nowrite"
]
```

When these commands are invoked (as a result of running `composer update`), they will generate 2 new files in your application: `_ide_helper.php` and `ide_helper_models.php`. Both these files can be added/committed in version control.

These files provide meta information to PhpStorm about the class structure of your app, which prevents PhpStorm from flagging certain methods as unavailable.

## Resolve *Validate method not found in Illuminate\Http\Request* flag

Even with the above changes, PhpStorm will still flag the `validate` method as unavailable when invoked on the `$request` object:


<img src='https://s3.amazonaws.com/making-the-internet/laravel-validate-method-phpstorm@2x.png' style='max-width:563px;' alt='Validate method not found'>


You can ignore this flag, or a [hack to fix it](https://github.com/barryvdh/laravel-ide-helper/issues/608) is add a new file `app/IDEAutoCompleteHelp.php` with this code:

```php
<?php
namespace Illuminate\Http;

/**
 * @method bool validate(array $rules, ...$params) Validate the given request with the given rules.
 */
class Request
{
}
```

## Resolve *Cannot resolve file* flag in views

When referencing a route in your views, PhpStorm might report that that path/file can not be resolved:

<img src='https://s3.amazonaws.com/making-the-internet/laravel-can-not-resolve-file@2x.png' style='max-width:787px;' alt=''>

Because of how URLs/directories typically work in a static site, it's looking or a file at `/books/search-process` - but because we're using routing that obviously does not exist.

I'm not aware of any ways to resolve this flag, but you can disable it on a project-by-project basis. To do this, go to *Preferences* : *Editor* : *Inspections* : *HTML* and uncheck *File reference problems*.

<img src='https://s3.amazonaws.com/making-the-internet/laravel-file-reference-problems@2x.png' style='max-width:1143px;' alt='File reference problems'>

Alternatively, you could work around the flag by using Laravel's [`url` helper method](https://laravel.com/docs/helpers#method-url):

```php
<form method='GET' action='{{ url('/books/search-process') }}'>
```

Or, you could use [named routes](https://laravel.com/docs/routing#named-routes):

```php
<form method='GET' action='{{ route('books.searchProcess') }}'>
```
