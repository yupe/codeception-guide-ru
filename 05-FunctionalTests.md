# Функциональное тестирование

Теперь, когда мы имеем написанные приемочные тсты, настало врмя рассмотреть фнукциональные тесты, функциональные тесты, это почти то же самое что и приемочные, однако есть одно существенное различие: Функциональные тесты не требуют использовния веб сервера для запуска своих сценариев. Другими словами мы будем запускать наше приложение внутри тестов, имитируя запросы и ответы.

Говоря простыми словами, мы устанавливаем переменные `$_REQUEST`, `$_GET` и `$_POST`, затем выполняем скрипт внутри тестаполучаем ответ и тестируем все это.
Функциональное тестирование часто может быть более лучшим реением чем приемочное потоку как такие тесты не требуют использования веб сервера и могут предложить более подробный отладочный вывод. К примеру, если ваш сайт выбросит исключение, оно будет напечатано в консоли.

Codeception может подключатся к различным веб фреймворкам поддерживающим функциональное тестирование. К примеру вы можете запустить функциональные тесты для приложения построенного поверх Zend Framework, Symfony или Symfony2 используя лишь модули поставляемые Codeception! Список поддерживаемых модулей будет разобран позже.

Модули для всех этих фреймворков имеют одинаковый интерфейс, поэтому ваши тесты не будут связханы друг с другом. Вот простой пример функционального теста.

``` php
<?php
$I = new TestGuy($scenario);
$I->amOnPage('/');
$I->click('Login');
$I->fillField('Username','Miles');
$I->fillField('Password','Davis');
$I->click('Enter');
$I->see('Hello, Miles', 'h1');
// $I->seeEmailIsSent() - special for Symfony2
?>
```

Такой же тест как и приемочный. Как видите можно использовать одинаковые методы и для приемочных и для функциональных тестов.
Мы рекомендуем тестировать нестабильные части приложения с помощью функциональных тестов, а стабильные с помощью приемочных.

## Ловушки

Приемочные тесты обычно намного медленнее, чем функциональные. Однако функциональные тесты менее стабильны и запускают тестовый фреймворк и приложение в одном окружении.

#### Headers, Cookies, Sessions

Одна из исзестных проблем функциональных тестов, использование PHP функций работающих с переменными из категории `headers`, `sessions`, `cookies`.
Как вы знаете, функция `header` возвратит ошибку, если будет выполнена более одного раза. В функциональных тестах мы запускаем наше приложение несколько раз, таким образом мы получим много ненужных ошибок при отображении результатов.

#### Разделяемая память

При функциональном тестировании в отличие от традиционного,  приложение PHP не останавливается после выполнения запроса.
Так как все запросы выполняются в одном контейнере памяти они не изолированны.
Таким образом **если вы заметили что ваши тесты магическим образом падают, однако не должны - попробуйте выполнить один тест.**
Это проверит изолированны ли тесты во время работы. Потому что довольно просто поломать окружение когда все тесты выполняются в разделяемой памяти.
Держите ваши память в чистоте, избегайте утечек памяти и очищайте глобальные и статические переменные.

## Основы функционального тестирования

Ваши функциональные тесты распологаются в каталоге `tests/functional`.
Для начала вам необходимо включить один из модулей фреймворков в конфигурационный файл тестового набора: `tests/functional.suite.yml`.
Примеры конфигурации фреймворков описаны ниже в данной главе.

После вам необходимо пересобрать Guy-классы

```
php codecept.phar build
```

Для генерации теста вы можете использовать стандартную команду генератор `generate:cept`:

```
php codecept.phar generate:cept functional myFirstFunctional
```

После чего выполнить тесты с помощью `run`:

```
php codecept.phar run functional
```

Используйте опцию `--debug` для более детального вывода.

## Сообщения об ошибках

По умолчанию Codeception использует значенич `E_ALL & ~E_STRICT & ~E_DEPRECATED`. 
В функциональных тестах вы можете захотеть сменить эти значения в зависимости от используемого фреймворка.
Сообщения об ошибках могут быть настроены в конфигурационном файле набора:

{% highlight yaml %}
class_name: TestGuy
modules:
    enabled: [Yii1, TestHelper]
error_level: "E_ALL & ~E_STRICT & ~E_DEPRECATED"
{% endhighlight %}

`error_level` может быть установлен глобально в файле `codeception.yml`.

## Frameworks

Codeception have integrations for the most popular PHP frameworks.
We aim to get modules for all of the most popular ones.
Please help to develop them if you don't see your favorite framework in a list.

### Symfony2

To perform Symfony2 integrations you don't need to install any bundles or perform any configuration changes.
You just need to include the Symfony2 module into your test suite. If you also use Doctrine2, don't forget to include it either.

Example for `functional.suite.yml`

```yaml
class_name: TestGuy
modules:
    enabled: [Symfony2, Doctrine2, TestHelper] 
```

By default this module wilyl search for Kernel in the `app` directory.

The module uses the Symfony Profiler to provide additional information and assertions.

[See the full reference](http://codeception.com/docs/modules/Symfony2)

### Laravel 4

[Laravel](http://codeception.com/docs/modules/Laravel4) module is zero configuration and can be easily set up.

```yaml
class_name: TestGuy
modules:
    enabled: [Laravel4, TestHelper]
```

### Yii

By itself Yii framework does not have an engine for functional testing.
So Codeception is the first and only functional testing framework for Yii.
To use it with Yii include `Yii1` module into config.

```yaml
class_name: TestGuy
modules:
    enabled: [Yii1, TestHelper]
```

To avoid common pitfalls we discussed earlier, Codeception provides basic hooks over Yii engine.
Please set them up [following the installation steps in module reference](http://codeception.com/docs/modules/Yii1).

### Zend Framework 2

Use [ZF2](http://codeception.com/docs/modules/ZF2) module to run functional tests inside Zend Framework 2.

```yaml
class_name: TestGuy
modules:
    enabled: [ZF2, TestHelper]
```

### Zend Framework 1.x

The module for Zend Framework is highly inspired by ControllerTestCase class, used for functional testing with PHPUnit. 
It follows similar approaches for bootstrapping and cleaning up. To start using Zend Framework in your functional tests, include the ZF1 module.

Example for `functional.suite.yml`

```yaml
class_name: TestGuy
modules:
    enabled: [ZF1, TestHelper] 
```

[See the full reference](http://codeception.com/docs/modules/ZF1)

### symfony

This module was the first one developed for Codeception. Because of this, its actions may differ from what are used in another frameworks.
It provides various useful operations like logging in a user with sfGuardAuth or validating the form inside a test.

Example for `functional.suite.yml`

```yaml
class_name: TestGuy
modules:
    enabled: [Symfony1, TestHelper] 
```

[See the full reference](http://codeception.com/docs/modules/Symfony1)

## Integrating Other Frameworks

Codeception doesn't provide any generic functional testing module because there are a lot of details we can't implement in general.
We already discussed the common pitfalls for functional testing. And there is no one single recipe how to solve them for all PHP applications.
So if you don't use any of the frameworks listed above, you might want to integrate your framework into Codeception. That task requires some knowledge of Codeception internals and some time. Probably, you are ok with just acceptance tests, but any help in extending Codeception functionality will be appreciated. We will review what should be done to have your framework integrated.

#### With HttpKernel

If you have a framework that uses Symfony's `HttpKernel`, using it with Codeception will be like a piece of cake.
You will need to create a module for it and test it on your application.
We already have a [guide for such integration](http://codeception.com/01-24-2013/connecting-php-frameworks-1.html).
Develop a module, try it and share with community.

#### Any Other

Integration is a bit harder if your framework is not using HttpKernel component.
The hardest part of it is resolving common pitfalls: memory management, and usage of `headers` function.
Codeception uses [BrowserKit](https://github.com/symfony/BrowserKit) from the Symfony Components to interact with applications in functional tests. This component provides all of the common actions we see in modules: click, fillField, see, etc... So you don't need to write these methods in your module. For the integration you should provide a bridge from BrowserKit to your application.

We will start with writing a helper class for framework integration.

```php
<?php
namespace Codeception\Module;
class SomeFrameworkHelper extends \Codeception\Util\Framework {
     
}
?>
```

Let's investigate [Codeception source code](https://github.com/Codeception/Codeception).
Look into the `src/Util/Framework.php` file that we extend. It implements all of the common actions for all frameworks.

You may see that all of the interactions are performed by a 'client' instance. We need to create a client that connects to a framework.
Client should extend Symfony\BrowserKit\Client module, sample clients are in the `src/Util/Connector` path. 
If the framework doesn't provide its own tools for functional testing you can try using the Universal connector. Otherwise, look at how the Zend connector is implemented, and implement your own connector.

Whether you decide to use the Universal connector or write your own, you can include it into your module.

```php
<?php
namespace Codeception\Module;
class SomeFrameworkHelper extends \Codeception\Util\Framework {
     
    public function _initialize() {
        $this->client = new \Codeception\Util\Connector\Universal();
        // or any other connector you implement
        
        // we need specify path to index file
        $this->client->setIndex('public_html/index.php');
    }     
}
?>
```

If you include this helper into your suite, it can already perform interactions with your applications.
You can extend its abilities by connecting to the framework internals. 
It's important to perform a proper clean up on each test run. 
This can be done in the _before and _after methods of the helper module. Check that the framework doesn't cache any data or configuration between tests.

After you get your module stabilized, share it with the community. Fork a Codeception repository, add your module and make a Pull Request.

There are some requirements for modules:

* It should be easy to configure.
* It should contain proper documentation.
* It should extend basic operations by using framework internals.
* It's preferred that it be able to print additional debug information.

We don't require unit tests, because there is no good way for writing proper unit tests for framework integration.
But you can demonstrate a sample application with your framework which uses Codeception tests and your module. 

## Conclusion

Functional tests are great if you are using powerful frameworks. By using functional tests you can access and manipulate their internal state. 
This makes your tests shorter and faster. In other cases, if you don't use frameworks, there is no practical reason to write functional tests.
If you are using a framework other than the ones listed here, create a module for it and share it with community.
