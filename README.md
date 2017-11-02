LucyUnit
=============

Containerized unit testing across any platform and programming language.

It is a fork of https://github.com/dockunit/dockunit . It has plus features of that version:

- Removed backup-cleanup process of files, which backed up files, then later deleted all of them (from the root of project) and overwrote them from backup. 
It was not just useless, but could cause data loss if there are some errors or container was stopped while the process was running. 
Also could be very slow...
- Default shell is not /bin/bash, but /bin/sh
- The shell can be specified in the `Lucyunit.json`
- Color output of Docker container (very useful e.g. with Mocha)

## Purpose

**Note:** This will be purposed towoards wordpress development cycle, a lot of things will be opiononated on that.

We all want to test our applications on as many relevant platforms as possible. Sometimes this is easy.
Sometimes it's not. Lucyunit let's you define a set of Docker containers to run your tests against. You can run your
test framework of choice in your language of choice on any type of environment. In the past many developers, myself
included, have relied on Travis CI to run tests in environments that aren't setup locally (i.e. PHP 5.2). With
Lucyunit you don't need to do this anymore.

## Requirements

* OSX or a Linux Distribution (Windows not yet tested or officially supported)
* [Node.js](http://nodejs.org/)
* [npm](https://www.npmjs.com/)
* [Docker](https://www.docker.com/)

## Installation

1. Make sure you have [Node.js](http://nodejs.org/), [Docker](https://www.docker.com/), and [npm](https://www.npmjs.com/) install
1. Install via npm:

  ```bash
  npm install -g lucyunit
  ```

## Usage

Lucyunit relies on `Lucyunit.json` files. Each of your projects should have their own `Lucyunit.json` file.
`Lucyunit.json` defines what test commands should be run on what type of containers for any given project. Here is an
example `Lucyunit.json`:

```json
{
  "containers": [
    {
      "prettyName": "PHP 5.2 on Ubuntu",
      "image": "user/my-php-image",
      "beforeScripts": [],
      "testCommand": "phpunit",
      "shell": "/bin/bash"
    },
    {
      "prettyName": "PHP 5.6 FPM on Ubuntu",
      "image": "user/my-php-image2",
      "beforeScripts": [],
      "testCommand": "phpunit"
    }
  ]
}
```

`containers` contains an array of container objects. Each container object can contain the following properties:

* `prettyName` (required) - This is used in output to help you identify your container.
* `image` (required) - This is a valid Docker container image located in the [Docker registry](https://registry.hub.docker.com/). We have a number of handy [prebuilt Docker images](https://github.com/dockunit/docker-prebuilt) for use in your `Lucyunit.json` files.
* `beforeScripts` (optional) - This is a string array of bash scripts to be run in order.
* `testCommand` (required) - This is the actual test command to be run on each container i.e. phpunit or qunit.
* `shell`: (optional) - The shell of the container (defaul is /bin/sh)

The Lucyunit command is:

```bash
lucyunit <path-to-project-directory> [--lcy-verbose] [--lcy-container] [--help] [--version] ...
```

_Note:_ `sudo` is usually required when run within a Linux distribution since Lucyunit runs Docker commands which require special permissions.

* `<path-to-project-directory>` (optional) - If you run `dockunit` in a folder with a `Lucyunit.json` folder, it will detect it
automatically.
* `[--lcy-verbose]` (optional) - This will print out light verbose Lucyunit output. `[--lcy-verbose=2]` will output even more verbose Lucyunit output.
* `[--lcy-container]` (optional) - Run only one container in your `Lucyunit.json` file by specifying the index of that container in the `containers` array .i.e `--lcy-container=1`.
* `[--help]` (optional) - This will display usage information for the `dockunit` command.
* `[--version]` (optional) - This will display the current installed version of Lucyunit.
* `...` - Any additional arguments and options passed to the command will be passed to your test command. For example,
if you wanted to pass a few extra options to PHPUnit, you could append them to the end of your `lucyunit` command.

__*You can simply run `lucyunit` in any folder with a `Lucyunit.json` to run Lucyunit.*__

## Lucyunit.json Examples

Each of your projects should have a `Lucyunit.json` file in the project root. You should define your containers to fit
your application's unique needs. Here's a few example `Lucyunit.json` files for a variety of different programming languages and
environments. Feel free to use any of our [prebuilt Docker images](https://hub.docker.com/r/dockunit/prebuilt-images/) in your `Lucyunit.json` files or create your own.

### PHP and WordPress

Lucyunit and WordPress work well together. WordPress is backwards compatible with PHP 5.2. It's very difficult to test
applications on PHP 5.2 without some sort of containerized workflow. Here is an example `Lucyunit.json` file that you
can use to test your WordPress plugins in PHP 5.2, 5.6, and PHP 7.0 RC 1 (make sure to replace `PLUGIN-FILE.php` with your plugins main file):

```javascript
{
  "containers": [
    {
      "prettyName": "PHP-FPM 5.2 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-5.2-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp-install latest"
      ],
      "testCommand": "wp-activate-plugin PLUGIN-FILE.php"
    },
    {
      "prettyName": "PHP-FPM 5.6 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-5.6-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp core download --path=/temp/wp --allow-root",
        "wp core config --path=/temp/wp --dbname=test --dbuser=root --allow-root",
        "wp core install --url=http://localhost --title=Test --admin_user=admin --admin_password=12345 --admin_email=test@test.com --path=/temp/wp --allow-root",
        "mkdir /temp/wp/wp-content/plugins/test",
        "cp -r . /temp/wp/wp-content/plugins/test"
      ],
      "testCommand": "wp plugin activate test --allow-root --path=/temp/wp"
    },
    {
      "prettyName": "PHP-FPM 7.0 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-7.0-rc-1-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp core download --path=/temp/wp --allow-root",
        "wp core config --path=/temp/wp --dbname=test --dbuser=root --allow-root",
        "wp core install --url=http://localhost --title=Test --admin_user=admin --admin_password=12345 --admin_email=test@test.com --path=/temp/wp --allow-root",
        "mkdir /temp/wp/wp-content/plugins/test",
        "cp -r . /temp/wp/wp-content/plugins/test"
      ],
      "testCommand": "wp plugin activate test --allow-root --path=/temp/wp"
    }
  ]
}
```

Here is an example `Lucyunit.json` file that you can use to test your WordPress themes in PHP 5.2, 5.6, and PHP 7.0 RC 1:

```javascript
{
  "containers": [
    {
      "prettyName": "PHP-FPM 5.2 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-5.2-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp-install latest"
      ],
      "testCommand": "wp-activate-theme test"
    },
    {
      "prettyName": "PHP-FPM 5.6 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-5.6-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp core download --path=/temp/wp --allow-root",
        "wp core config --path=/temp/wp --dbname=test --dbuser=root --allow-root",
        "wp core install --url=http://localhost --title=Test --admin_user=admin --admin_password=12345 --admin_email=test@test.com --path=/temp/wp --allow-root",
        "mkdir /temp/wp/wp-content/themes/test",
        "cp -r . /temp/wp/wp-content/themes/test"
      ],
      "testCommand": "wp theme activate test --allow-root --path=/temp/wp"
    },
    {
      "prettyName": "PHP-FPM 7.0 WordPress Latest",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-wordpress-7.0-rc-1-fpm",
      "beforeScripts": [
        "service mysql start",
        "wp core download --path=/temp/wp --allow-root",
        "wp core config --path=/temp/wp --dbname=test --dbuser=root --allow-root",
        "wp core install --url=http://localhost --title=Test --admin_user=admin --admin_password=12345 --admin_email=test@test.com --path=/temp/wp --allow-root",
        "mkdir /temp/wp/wp-content/themes/test",
        "cp -r . /temp/wp/wp-content/themes/test"
      ],
      "testCommand": "wp theme activate test --allow-root --path=/temp/wp"
    }
  ]
}
```

### PHP and WordPress Unit Tests

Here are some more advanced WordPress examples. That assume you have unit tests setup via [WP-CLI](https://github.com/wp-cli/wp-cli/wiki/Plugin-Unit-Tests).

```javascript
{
  "containers": [
    {
      "prettyName": "PHP 5.2 FPM WordPress 4.1",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-5.2-fpm",
      "beforeScripts": [
        "service mysql start",
        "bash bin/install-wp-tests.sh wordpress_test root '' localhost 4.1"
      ],
      "testCommand": "phpunit"
    },
    {
      "prettyName": "PHP 5.6 FPM WordPress 4.0",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-5.6-fpm",
      "beforeScripts": [
        "service mysql start",
        "bash bin/install-wp-tests.sh wordpress_test2 root '' localhost 4.0"
      ],
      "testCommand": "phpunit"
    },
    {
      "prettyName": "PHP 7.0 RC-1",
      "image": "dockunit/prebuilt-images:php-mysql-phpunit-7.0-rc-1-fpm",
      "beforeScripts": [
        "service mysql start",
        "bash bin/install-wp-tests.sh wordpress_test3 root '' localhost 3.9"
      ],
      "testCommand": "phpunit"
    }
  ]
}
```

[dockunit/prebuilt-images:php-mysql-phpunit-5.6-fpm](https://hub.docker.com/r/dockunit/prebuilt-images/), [dockunit/prebuilt-images:php-mysql-phpunit-5.6-fpm](https://hub.docker.com/r/dockunit/prebuilt-images), and [dockunit/prebuilt-images:php-mysql-phpunit-7.0-rc-1-fpm](https://hub.docker.com/r/dockunit/prebuilt-images) are valid Docker images available for use in any `Dockerfile.json`.

## License

Lucyunit is free software; you can redistribute it and/or modify it under the terms of the [GNU General
Public License](http://www.gnu.org/licenses/gpl-2.0.html) as published by the Free Software Foundation; either version
2 of the License, or (at your option) any later version.
