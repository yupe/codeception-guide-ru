# Настройка

В данном разделе, мы объясним, как вы можете расширить и настроить структуру файлов тестов, а так же их порядок выполнения.

## Один загрузчик для нескольких приложений

В случае, если проект состоит из нескольких приложений (frontend, admin, api) или вы используете бандлы Symfony2,
вы можете захотеть, чтобы тесты для всех приложений запускались одним загрузчиком.
В таком случае вы получите один отчет для всего проекта.

Начиная с версии Codeception 1.6.3 появилась возможность создать meta-config, включающий конфигурационные файлы из разных каталогов.

Поместите `codeception.yml` в корень проекта, и укажите пути к других `codeception.yml` конфигам, которые вы хотите подключить.

``` yaml
include:
  - frontend
  - admin
  - api/rest
paths:
  log: log
settings:
  colors: false
```

Кроме того, вам следует указать путь к каталогу `log`, в котором будут храниться отчеты и логи всего проекта.

### Пространства имен

Для того, чтобы избежать конфликтов имен между Guy и Helper классами, они должны быть добавлены в пространства имен.
Для создания набора тестов с использованием пространств имен, добавьте опцию `--namespace` к команде bootstrap.

``` bash
php codecept.phar bootstrap --namespace frontend

```

Это создаст новый проект с параметром `namespace: frontend` в файле конфигурации `codeception.yml`.
Помощники будут использовать пространство имен `frontend\Codeception\Module` а классы Guy пространство имен `frontend`.
Сгенерированные классы будут выглядеть примерно так:

``` php
<?php use frontend\WebGuy;
$I = new WebGuy($scenario);
//...
?>
```

Codeception имеет утилиты для того, чтобы обновить тесты существующего проекта, добавив в них использование пространств имен. Этого можно добиться выполнив следующую комманду

``` bash
php codecept.phar refactor:add-namespace frontend

```

Вы получите guy классы и помощники, а так же cept тесы обновленные для использования с пространствами имен. Запомните, что Cest файлы должны быть обновлены вручную. Коме того опция `namespace` не изменит пространство имен тестов или Cest классов. Она используется только для Guys классов или классов Помощников.

В то время, когда каждое приложение (бандл) имеет собственное пространство имен и разные классы помощники и guy классы, вы можете выполнить их с помощью одной команды.
Используйте meta-config который мы создали выше для запуска тестов как вы делаете это обычно.

```
php codecept.phar run

```

Это выполнит тесты для всех 3 приложений и сольет отчеты в один. По существу, это может быть довольно удобно, в случае, если вы запускаете тесты на сервере непрерывной интеграции и хотите получить один отчет в формате JUnit или HTML. Отчет о покрытии кода тестами, будет так же объединен в один.

Если ваше приложение должно использовать одинаковые классы помощников обратитесь к следующему разделу

## Autoload Helper classes

В Codeception 1.6.3 был анонсирован глобальный файл `_bootstrap.php`. По умолчанию вы можете поместить его в каталог `tests`. Если данный файл существует, он будет подключен в самом начале последовательности запуска тестов. Мы рекомендуем использовать его для инициализации загрузчиков и констант. Он довольно полезен, если вы хотите к примеру подключить классы модулей или помощников, которые хранятся в каталоге отличном от `tests/_helpers`.

``` php
<?php
require_once __DIR__.'/../lib/tests/helpers/MyHelper.php'
?>
```

Как альтернатива, вы можете использовать Composer загрузчик. Codeception имеет так же свой автозагрузчик.
Пока что он не совместим с PSR-0, однако он довольно полезен, если вам необходимо объявить альтернативный путь к вашим классам помощникам:

``` php
<?php
Codeception\Util\Autoload::registerSuffix('Helper', __DIR__.'/../lib/tests/helpers');
?>
```

Теперь все классы с суффиксом `Helper` дополнительно будут искаться в каталоге `__DIR__.'/../lib/tests/helpers'. Вы можете объявить загрузку помощников из определенного пространства имен.

``` php
<?php
Codeception\Util\Autoload::register('MyApp\\Test','Helper', __DIR__.'/../lib/tests/helpers');
?>
```

Это укажет загрузчику искать классы типа `MyApp\Test\MyHelper` в каталоге `__DIR__.'/../lib/tests/helpers'`.

Кроме этого вы можете использовать автощагрузчик для указания пути к классам **PageObject and Controller** если они имеют подходщие суффиксы в своих именах.

Пример файла `tests/_bootstrap.php`:

``` php
<?php
Codeception\Util\Autoload::register('MyApp\\Test','Helper', __DIR__.'/../lib/tests/helpers');
Codeception\Util\Autoload::register('MyApp\\Test','Page', __DIR__.'/pageobjects');
Codeception\Util\Autoload::register('MyApp\\Test','Controller', __DIR__.'/controller');
?>
```

## Extension classes

<div class="alert">This section requires advanced PHP skills and some knowlegde of Codeception and PHPUnit internals.</div>

Codeception has limited capabilities to extend its core features.
Extensions are not supposed to override current functionality, but are pretty useful if you are experienced developer and you want to hook into testing flow.

Basically speaking, Extensions are nothing more then event listeners based on [Symfony Event Dispatcher](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) component.

Here are the events and event classes. The events are listed in order they happen during execution. Each event has a class which are passed into listener and contains objects of this event.

### Events


|    Event             |     When?                               | What?                       
|:--------------------:| --------------------------------------- | --------------------------:
| `suite.before`       | Before suite is executed                | [Suite, Settings](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Suite.php)
| `test.start`         | Before test is executed                 | [Test](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Test.php)
| `test.before`        | At the very beginning of test execution | [Codeception Test](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Test.php)
| `step.before`        | Before step                             | [Step](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Step.php)
| `step.after`         | After step                              | [Step](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Step.php)
| `step.fail`          | After failed step                       | [Step](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Step.php)
| `test.fail`          | After failed test                       | [Test, Fail](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Fail.php)
| `test.error`         | After test ended with error             | [Test, Fail](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Fail.php)
| `test.incomplete`    | After executing incomplete test         | [Test, Fail](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Fail.php)
| `test.skipped`       | After executing skipped test            | [Test, Fail](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Fail.php)
| `test.success`       | After executing successful test         | [Test](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Test.php)
| `test.after`         | At the end of test execution            | [Codeception Test](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Test.php)
| `test.end`           | After test execution                    | [Test](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Test.php)
| `suite.after`        | After suite was executed                | [Suite, Result, Settings](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Suite.php)
| `test.fail.print`    | When test fails are printed             | [Test, Fail](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/Fail.php)
| `result.print.after` | After result was printed                | [Result, Printer](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Event/PrintResult.php)


There may be a confusion between `test.start`/`test.before` and `test.after`/`test.end`. Start/end events are triggered by PHPUnit itself.
But before/after events are triggered by Codeception. Thus, when you have classical PHPUnit test (extended from `PHPUnit_Framework_TestCase`)
before/after events won't be triggered for them. On `test.before` event you can mark test as skipped or incomplete, which is not possible in test/start. You can learn more from [Codeception internal event listeners](https://github.com/Codeception/Codeception/tree/master/src/Codeception/Subscriber).

The extension class itself is inherited from `Codeception\Platform\Extension`.

``` php
<?php
class MyCustomExtension extends \Codeception\Platform\Extension
{
    // list events to listen to
    static $events = array(
        'suite.after' => 'afterSuite',
        'test.before' => 'beforeTest',
        'step.before' => 'beforeStep',
        'test.fail' => 'testFailed',
        'result.print.after' => 'print',
    );

    // methods that handles events

    function afterSuite(\Codeception\Event\Suite $e) {}

    function beforeTest(\Codeception\Event\Test $e) {}

    function beforeStep(\Codeception\Event\Step $e) {}

    function print(\Codeception\Event\PrintResult $e) {}
}
?>
```  

By implementing evend handling methods you can listen to event and even update passed objects.
Extensions have some basic methods you can use:

* `write` - prints to screen
* `writeln` - prints to screen with line end char at the end
* `getModule` - allows you to access a module.
* `_reconfigure` - can be implemented instead of overriding constructor. 

### Enabling Extension

Once you implemented a simple extension class you should include it into `tests/_bootstrap.php` file:

``` php
<?php
include '/path/to/my/MyCustomExtension.php'
?>
```

Then you can enabled it in `codeception.yml`:

```
extensions:
    enabled: [MyCustomExtension]

```

### Configuring Extension

In extension you can access current passed options in `options` property.
You can access global config via `\Codeception\Configuration::config()` method. 
But if you want to have custom options for your extension you can pass them in `codeception.yml` file:

```
extensions:
    enabled: [MyCustomExtension]
    config:
        MyCustomExtension:
            param: value

```

Passed configuration is accessible via `config` property `$this->config['param']`.

Check out a very basic extension [Notifier](https://github.com/Codeception/Notifier).

## Group Classes

Group Classes are extensions listening to events of a tests belonging to a specific group.
When a test is added to a group:

``` php
<?php 
$scenario->group('admin');
$I = new WebGuy($scenario);
?>
```

This test will trigger events:

* `test.before.admin`
* `step.before.admin`
* `step.after.admin`
* `test.success.admin`
* `test.fail.admin`
* `test.after.admin`

A group class is built to listen to this events. It is pretty useful when you require additional setup for some of your tests. Let's say you want to load fixtures for tests belonging to `admin` group.

``` php
<?php
class AdminGroup extends \Codeception\Platform\Group
{
    static $group = 'admin';

    public function _before(\Codeception\Event\Test $e)
    {
        $this->writeln("inserting additional admin users...");

        $db = $this->getModule('Db');
        $db->haveInDatabase('users', array('name' => 'bill', 'role' => 'admin'));
        $db->haveInDatabase('users', array('name' => 'jon', 'role' => 'admin'));
        $db->haveInDatabase('users', array('name' => 'mark', 'role' => 'banned'));
    }

    public function _after(\Codeception\Event\Test $e)
    {
        $this->writeln("cleaning up admin users...");
    }
}
?>
```

A group class can be created with `php codecept.phar generate:group groupname` command.
Group class will be stored in `tests/_groups` directory.

A group class should just the same manner you can enable extension class. In file `codeception.yml`:

``` yaml
extensions:
    enabled: [AdminGroup]    
```

Now Admin group class will listen to all events of tests that belong to the `admin` group.

## Conclusion

Each mentioned feature above may dramaticly help when using Codeception to automate large projects. 
Each of feature requires advanced knowledge of PHP. There is no "best practice" or "use cases" when we talk about groups, extensions, or other power features of Codeception. If you see you have a problem that can be solved using this extensions, then give them a try. 
