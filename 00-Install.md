# Установка

**Phar**

Grab Codeception phar executable http://codeception.com/thanks

~~~
wget http://codeception.com/codecept.phar .
~~~

Prepare tests directory and configs

~~~
php codecept.phar bootstrap
~~~

**Composer**

Install a Composer to your project's root http://getcomposer.org/

Run

~~~
php composer.phar require "codeception/codeception:*"
~~~

From now on Codeception (with installed PHPUnit) can be run as:

~~~
vendor/bin/codecept
~~~

Initialize your testing environment with

~~~
vendor/bin/codecept bootstrap
~~~

**Git**

Alternative installation method for bugfixing, contributions and hacking

Clone from GitHub:

~~~
git clone git@github.com:Codeception/Codeception.git
~~~

Install dependencies with Composer

~~~
cd Codeception
curl -s http://getcomposer.org/installer | php
php composer.phar install
~~~

Execute bootstrap, specifying path to your directory.

~~~
php codecept bootstrap /path/to/my/project
~~~

To run tests use -c option for specifing path.

~~~
php codecept run -c /path/to/my/project
~~~

If you want to build phar package run php bin/build_phar.php

Don't forget to send Pull Requests!