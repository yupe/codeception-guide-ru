# Продвинутое использование

В данном разделе мы рассмотрим некоторые техники и опции использование которых поможет вам улучшить ваши навыки в тестировании и сохранить качественную организацию вашего проекта.

## Интерактивная консоль

Интерактивная консоль добавлена для опробования команд Codeception перед тем, как они будут выполнены внутри тестов.
Данная возможность была анонсирована  начиная с версии 1.6.0.

![console](http://img267.imageshack.us/img267/204/003nk.png)

Вы можете запустить консоль с помощью команды

``` bash
php codecept.phar console suitename
```

Теперь вы можете выполнять любые команды соответствующего Guy класса и сразу же видеть результаты их выполнения. Это особенно полезно в случае, работы с модулями Selenium. Запуск Selenium и браузера для тестирования обычно занимает довольно продолжительное время. Однако при использовании коносли вы можете попробовать разнообразные селекторы и команды, после чего вы сможете быть уверенны в том, что написаный вами тест выполнится.

Полезный совет: покажите начальству, как резво вы манипулируете веб страницами с помощью консоли и Selenium. Так вы сможете убедить его в том, что довольно просто автоматизировать подобные вещи и начать использовать приемочное тестирование в проекте.

## Запуск из разных каталогов.

Если у вас есть несколько проектов содержащих Coeception тесты, вы можете использовать один `codecept.phar` файл для их запуска. Чтобы запустить Codeception из другой директории, вы можете добавить опцию `-c` к любой Codeception команде кроме `bootstrap`.

```

php codecept.phar run -c ~/projects/ecommerce/
php codecept.phar run -c ~/projects/drupal/
php codecept.phar generate:cept acceptance CreateArticle -c ~/projects/drupal/

```

Для создания проекта в директории отличной от той, где вы находитесь, просто добавьте ее путь как параметр.


```

php codecept.phar bootstrap ~/projects/drupal/


```

В сущности опция `-c` позволяет указать не только путь, но и файл конфигурации. Таким образом у вас может быть несколько различных файлов `codeception.yml` в одном тестовом наборе. Вы можете использовать их для указания разных переменных окружения и настроек. Просто передайте имя конфигурационного файла в параметре `-c` для того, чтобы выполнить тесты с настройками указанными в данном конфигурационном файле.

## Группы

Есть несколько путей позволяющих выполнить группу тестов. Вы можете выполнить тесты в определенной директории:

```

php codecept.phar run tests/acceptance/admin


```

Или выполнить одну (или несколько) групп тестов:

```

php codecept.phar run -g admin -g editor

```

В данном случае, будут выполнены все тесты принадлежащие группам admin или editor. Концепция групп взята из PHPUnit и его классических тестов которые ведут себя подобным образом. Чтобы добавить Cept тест к группе, используйте переменную `$scenario`:

``` php
<?php
$scenario->group('admin');
$scenario->group('editor');
// or
$scenario->group(array('admin', 'editor'))
// or
$scenario->groups(array('admin', 'editor'))

$I = new WebGuy($scenario);
$I->wantToTest('admin area');
?>
```
Для классических тестов и Cests тестов, чтобы добавить тест к группе можно использовать аннотацию `@group`.

``` php
<?php
/**
 * @group admin
 */
public function testAdminUser()
{
    $this->assertEquals('admin', User::find(1)->role);
}
?>
```
Та же аннотация может быть использована в Cest классах.

## Cest Классы

In case you want to get a class-like structure for your Cepts, instead of plain PHP, you can use Cest format.
It is very simple and is fully compatible with Cept scenarios. It means If you feel like your test is long enough and you want to split it - you can easily move it into class. 

You can start Cest file by running the command:

```
php codecept.phar generate:cest suitename CestName
```

The generated file will look like:

``` php
<?php
class BasicCest
{

    public function _before()
    {
    }

    public function _after()
    {
    }

    // tests
    public function tryToTest(\WebGuy $I) {
    
    }
}
?>
```

**Each public method of Cest (except, those starting from `_`) will be executed as a test** and will receive Guy class as the first parameter and `$scenario` variable as a second. 

In `_before` and `_after` method you can use common setups, teardowns for the tests in the class. That actually makes Cest tests more flexible, then Cepts that rely only on similar methods in Helper classes.

As you see we are passing Guy class into `tryToTest` stub. That allows us to write a scenarios the way we did before.

``` php
<?php
class BasicCest
{
    // test
    public function checkLogin(\WebGuy $I) {
        $I->wantTo('log in to site');
        $I->amOnPage('/');
        $I->click('Login');
        $I->fillField('username', 'jon');
        $I->fillField('password','coltrane');
        $I->click('Enter');
        $I->see('Hello, Jon');
        $I->seeInCurrentUrl('/account');
    }
}
?>
```

But there is a limiation in Cest files. It can't work with `_bootstrap.php` the way we did in scenario tests.
It was useful to store some variables in bootstraps that should be passed into scenario.
In Cest files you should inject all external variables manually, using static or global variables.

As a workaround you can choose [Fixtures](https://github.com/Codeception/Codeception/blob/master/src/Codeception/Util/Fixtures.php) class which is nothing more then global storage to your variables. You can pass variables from `_bootstrap.php` or any other place just with `Fixtures::add()` call. But probably you can use Cest classes `_before` and `_after` methods to load fixtures on the start of test, and deleting them afterwards. Pretty useful too.

As you see, Cest class have no parent like `\Codeception\TestCase\Test` or `PHPUnit_Framework_TestCase`. That was done intentionally. This allows you to extend class any time you wnat by attaching any meta-testing class to it's parent. In meta class you can write common behaviors and workarounds that may be used in child class. But don't forget to make them `protected` so they won't be executed as a tests themselves.

Also you can define `_failed` method in Cest class which will be called if test finished with `error` or fail.

## Refactoring

As test base growth they will require refactoring, sharing common variables and behaviors. The classical example for this is `login` action which will be called for maybe every test of your test suite. It's wise to make it written one time and use it in all tests. 

It's pretty obvious that for such cases you can use your own PHP classes to define such methods. 

``` php
<?php class TestCommons 
{
    public static $username = 'jon';
    public static $password = 'coltrane';

    public static logMeIn($I)
    {
        $I->amOnPage('/login');
        $I->fillField('username', 'jon');
        $I->fillField('password','coltrane');
        $I->click('Enter');
    }
}
?>
```

This file can be required in `_bootstrap.php` file

``` php
<?php
// bootstrap
require_once '/path/to/test/commons/TestCommons.php';
?>
```

and used in your scenarios:

``` php
<?php
$I = new WebGuy($scenario);
TestCommons::logMeIn($I);
?>
``` 

You should get the idea by now. Codeception doesn't provide any particular strategy for you to manage the tests. But it is flexible enough to create all the support classes you need for your test suites. The same way, with custom classes you can implement `PageObject` and `StepObject` patterns.

## PageObject and StepObjects

In next versions Codeception will provide PageObjects and StepObjects out of the box. 
But today we encourage you to try your own implementations. Whe have some nice blogpost for you to learn what are PageObjects, why you ever want to use them and how they can be implemented.

* [Ruling the Swarm (of Tests)](http://phpmaster.com/ruling-the-swarm-of-tests-with-codeception/) by Michael Bodnarchuk.
* [Implementing Page Objects in Codeception](http://jonstuff.blogspot.ca/2013/05/implementing-page-objects-in.html) by Jon Phipps.

## Conclusion

Codeception is a framework which may look simple at first sight. But it allows you to build powerful test with one  APIs, refactor them, and write them faster using interactive console. Codeception tests can easily be organized with groups or cest classes. Probably too much abilities for the one framework. But nevertheless Codeception follows the KISS pricinple: it's easy to start, easy to learn, easy to extend. 