# Начнем!

Давайте посмотрим на архитектуру Codeception. Будем считать, что Codeception уже [установлен](http://codeception.com/install) а наборы тестов проинициализированны. Codeception генерирует три стандартных набора для слудующих категорий тестов: модульных (unit), функциональных (functional), и приемочных (acceptance). Все они были описаны в предыдущей статье. Внутри каталога `/tests` мы имеем три конфигурационных файла и три каталога с названиями соответствующими наборам тестов. "Наборы тестов" (suites) - это независимые группы тестов, объединенные общей целью. 

## Парни

Основной концепцией Codeception является представление тестов как действий человека. Мы будем называть этого человека "парнем" (Guy). У нас есть CodeGuy, который исполняет функии и тестирует код. Также у нас есть TestGuy, опытный тестировщик, который тестирует приложение целиком и знает кое-что о его внутреннем устройстве. Еще у нас есть WebGuy, пользователь, который работает с нашим приложением через интрерфейс, который мы предоставляем.

Каждый из этих "парней" парней представляет из себя PHP класс с набором действий, который он может совершать. Как вы можете видеть, каждый из этих "парней" емеет различные способности. Эти способности не постоянны, Вы можете расширять их. Вы даже можете создавать новых "парней", но запомните: один "парень" - один набор тестов.


Классы "парней" не пишутся, они генерятся следующей командой:

```
$ php codecept.phar build
```

## Напишем простой сценарий

По умолчанию тесты пишутся в виде последовательных сценариев. Чтобы PHP-файл стал валидным сценарием, его имя должно содержать суффикс "Cept". 

Предположим, что мы создали файл  `tests/acceptance/SigninCept.php`

Мы можем сделать это следующей командой:

```
$ php codecept.phar generate:cept acceptance Signin
```

~~~
[php]
<?php
$I = new WebGuy($scenario);
?>
~~~

Сценарий всегда начинается с инициализации класса нашего "парня". После этого написание сценария сводится к написанию `$I->` и выбору соответсвующего действия из списка авто-дополнения.

Давайте авторизуемся на нашем сайте. Мы предполагаем, что у нас есть страница "login", на которой для авторизации необходимо ввести логин и праоль. После авторизации мы редиректим пользователя на страницу, где он видит текст `Hello, %username%`. Давайте разберемся как написать этот сценраий, используя Codeception.

~~~
[php]
<?php
$I = new WebGuy($scenario);
$I->wantTo('log in as regular user');
$I->amOnPage('/login');
$I->fillField('Username','davert');
$I->fillField('Password','qwerty');
$I->click('Login');
$I->see('Hello, davert');
?>
~~~

Прежде чем запустить тест, мы должны убедиться, что сайт работает на локальном веб-сервере. Откройте `tests/acceptance.suite.yml` и замените URL на URL Вашего приложения:

``` yaml
config:
    PhpBrowser:
        url: 'http://myappurl.local'
```

Если у Вас нет запущенного веб-сервера Вы можете использолвать [PHP Built-in Web Server](http://php.net/manual/en/features.commandline.webserver.php) который доступен, начиная с версии PHP 5.4 

После того как Вы установили корректный URL, Вы можете запустить тест, используя следующую команду:

``` bash
$ php codecept.phar run
```

Вот, что Вы должны увидеть на выходе: 

``` bash
Suite acceptance started
Trying log in as regular user (SigninCept.php) - Ok

Suite functional started

Suite unit started


Time: 1 second, Memory: 21.00Mb

OK (1 test, 1 assertions)
```

Давайте посмотрим более детальный вывод:

```bash
$ php codecept.phar run acceptance --steps
```

Мы должны увидеть шаг за шагом последовательность выполняемых действий.

```bash
Suite acceptance started
Trying to log in as regular user (SigninCept.php)
Scenario:
* I am on page "/login"
* I fill field "Username" "davert"
* I fill field "Password" "qwerty"
* I click "Login"
* I see "Hello, davert"
  OK


Time: 0 seconds, Memory: 21.00Mb

OK (1 test, 1 assertions)
```

Это очень простой, который Вы можете воспроизвести для своего собственного сайта. Эмулируя действия пользователя, Вы можете протестировать весь Ваш сайт подобным образом.

Просто попробуйте!

## Модули и хелперы

Все действия в Guy-классе берутся из модулей. Команда "build", описанная ваше, позволяет Codeception эмулировать множественное наследование. Modules are designed to have one action performed with one method. According to the [DRY principle](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself), if you use the same scenario components in different modules, you can combine them and move them to a custom module. By default each suite has an empty module, which can extend Guy classes. They are stored in the `helpers` directory.

## Bootstrap

Each suite has its own bootstrap file. It's located in the suite directory and is named `_bootstrap.php`. It will be executed before each test.

Write any setup preparations for the suite there.

## Tests
Codeception supports three test formats. Besides the previously described scenario-based Cept format, Codeception can also execute PHPUnit test files for unit testing, and a hybrid scenario-unit Cest format. They are covered in later chapters. There is no difference in the way the tests of either format will be run in the suite.

## Important Notes

Codeception is pretty smart when executing test scenarios. But you should keep in mind that Codeception executes each scenario two times: one for analysis and one for execution. So, any custom PHP code put in the file will be executed two times! Probably, you don't need that. Before injecting any custom PHP code (which is not using the `$I` object) into your test, please specify the stage when it should be executed.

__To say it again: each test file is run two times: for analyses and execution.__

For instance, this code will be run when the scenario is analyzed.

```php
<?php
if ($scenario->preload()) {
    Logger::log('analyzing');
}
?>
```

and this is executed in the test runtime:

```php
<?php
if ($scenario->running()) {
    Logger::log('running');
}
?>
```

But whenever you can try to keep you tests simple and avoid using any custom PHP code. Use only the `$I` object whenever it is possible. Keep the test clean and readable.

## Configuration

Codeception has a global configuration in `codeception.yml` and a config for each suite. We also support `.dist` configuration files. If you have several developers in a project, put shared settings into `codeception.dist.yml` and personal settings into `codeception.yml`. Same goes for suite configs. For example, the `unit.suite.yml` will be merged with `unit.suite.dist.yml`. 

By default your global configuration file will be this:

```yaml
paths:
    # where the modules stored
    tests: tests

    # logs and debug 
    # outputs will be written there
    log: tests/_log

    # directory for fixture data    
    data: tests/_data

    # directory for custom modules (helpers)
    helpers: tests/_helpers

settings:

    # name of bootstrap that will be used
    # each bootstrap file should be 
    # inside a suite directory.

    bootstrap: _bootstrap.php

    # You can extend the suite class if you need to.
    suite_class: \PHPUnit_Framework_TestSuite

    # by default it's false on Windows
    # use [ANSICON](http://adoxa.110mb.com/ansicon/) to colorize output.
    colors: true

    # Tests (especially functional) can take a lot of memory
    # We set a high limit for them by default.
    memory_limit: 1024M

    # If a log should be written.
    # Every action in test is logged.
    # Logs are kept for 3 days.
    log: true

# Global modules configuration.    
modules:
    config:
        Db:
            dsn: ''
            user: ''
            password: ''
            dump: tests/_data/dump.sql
```

Suite configuration acceptance.yml

```yaml
class_name: WebGuy
modules:
    # enabled modules and helpers
    enabled:
        - PhpBrowser
        - WebHelper
        - Db

    # local module configuration. Overrides the global.        
    config:
    	Db:
            dsn:
```


## Running Tests

Tests can be started with the `run` command.

```bash
$ php codecept.phar run
```

With the first argument you can run tests from one suite.

```bash
$ php codecept.phar run acceptance
```

To run exactly one test, add a second argument. Provide a local path to the test, from the suite directory.

```bash
$ php codecept.phar run acceptance SigninCept.php
```

There are plenty of options you can use.

* `steps` - all performed actions will be printed to console.
* `debug` - additional debug output will be printed.
* `config` - specify different config file for current run.
* `colors` - turn on colors (if disabled)
* `silent` - don't show the progress output.
* `report` - format results in report mode.
* `coverage` - collect code coverage report
* `no-exit` - don't provide exit codes on finish. This option may be useful for using Codeception with some CI servers like Bamboo.

With the following options you can set the output in the most suitable format.

* `html` - generate HTML file with results. It will be stored as 'report.html' in tests/_log.
* `xml` - generate report in JUnit format for CI services. It will be stored as 'report.xml' in tests/_log.
* `tap` - generate report in TAP format. It will be stored as 'report.tap.log' in tests/_log.
* `json` - generate report in JSON format. It will be stored as 'report.json' in tests/_log.

Example:

```bash
$ php codecept.phar run --steps --xml --html
```

This command will run all tests for all suites, displaying the steps, and building HTML and XML reports.

### Generators

There are plenty of useful Codeception commands.

* `generate:cept` *suite* *filename* - Generates a sample Cept scenario.
* `generate:cest` *suite* *filename* - Generates a sample Cest test.
* `generate:test` *suite* *filename* - Generates a sample PHPUnit Test with Codeception hooks.
* `generate:phpunit` *suite* *filename* - Generates a classic PHPUnit Test.
* `generate:suite` *suite* *guy* - Generates a new suite with the given Guy class name.
* `generate:scenarios` *suite* - Generates text files containing scenarios from tests.


## Conclusion

We took a look into the Codeception structure. Most of the things you need were already generated by the `bootstrap` command. After you have reviewed the basic concepts and configurations, you can start writing your first scenarios. 




Трудились и переводили ребята из [amyLabs](http://amylabs.ru/)
