---
title: "PHP.ini settings"
weight: 4
---

There are two ways to customize `php.ini` values for your application.
The recommended method is to use the [`variables` property](../../create-apps/app-reference.md#variables)
of `.platform.app.yaml` to set `ini` values using the `php` prefix.
For example, to increase the PHP memory limit you'd put the following in `.platform.app.yaml`:

```yaml
variables:
    php:
        memory_limit: "256M"
```

It's also possible to provide a custom `php.ini` file in the repository in [your app root](../../create-apps/app-reference.md#root-directory).

```ini
; php.ini
; Increase PHP memory limit
memory_limit = 256M
```

Another example is to set the timezone of the PHP runtime (though, the timezone settings of containers/services would remain in UTC):

```yaml
variables:
    php:
        date.timezone: "Europe/Paris"
```

or

```ini
; php.ini
; Set PHP runtime timezone
date.timezone = "Europe/Paris"
```

Environment-specific `php.ini` configuration directives can be provided via environment variables separately from the application code.
See the note on [environment variables](../../development/variables/_index.md#php-specific-variables).

## Disabling functions

A common recommendation for securing a PHP installation is to disable certain built-in functions that are frequently used in remote attacks.
By default, Platform.sh doesn't disable any functions as they all do have some legitimate use in various applications.
You may wish to disable them yourself if you know they aren't needed.
For example, to disable `pcntl_exec` and `pcntl_fork` (which aren't usable in a web request anyway):

```yaml
variables:
    php:
        disable_functions: "pcntl_exec,pcntl_fork"
```

Common functions to disable include:

* `create_function` - `create_function` has no useful purpose since PHP 5.3 and shouldn't be used.
  It has been effectively replaced by anonymous functions.
* `exec,passthru,shell_exec,system,proc_open,popen` - These functions all allow a PHP script to run a bash shell command. That's rarely used by web applications, although build scripts may need them.
* `pcntl_exec,pcntl_fork,pcntl_setpriority` - The `pcntl_*` functions (including those not listed here) are responsible for process management.
  Most of them cause a fatal error if used within a web request.
  Cron tasks or workers may make use of them.
  Most are safe to disable unless you know that you are using them.
* `curl_exec,curl_multi_exec` - These functions allow a PHP script to make arbitrary HTTP requests.
  Note that they're frequently used by other HTTP libraries such as Guzzle, in which case you should *not* disable them.
* `show_source` - This function shows a syntax highlighted version of a named PHP source file.
  That's rarely useful outside of development.

If your application does make use of any of these functions, it fails if you disable them.
In that case, don't disable them.

## Default php.ini settings

The default values for some frequently modified `php.ini` settings are listed below.

* `memory_limit=128M`
* `post_max_size=64M`
* `upload_max_filesize=64M`
* `display_errors=On`

    This value is on by default to ease setting up a project on Platform.sh.
    It's strongly recommended to provide a custom error handler in your application or setting this value to `Off` before you make your site live.
* `zend.assertions=-1`

    Assertions are optimized out of existence and have no impact at runtime. You should have assertions set to `1` for your local development system.
* `opcache.memory_consumption=64`

    This is the number of megabytes available for the OPcache. Large applications with many files may want to increase this value.
* `opcache.validate_timestamps=On`

    The OPcache checks for updated files on disk.
    This is necessary to support applications that generate compiled PHP code from user configuration.
    If you are certain your application doesn't do so, then you can disable this setting for a small performance boost.

{{< note theme="warning" >}}
You aren't limited in what you can put in your `php.ini` file, but many settings can break your application. This is a facility for advanced users.
{{< /note >}}
