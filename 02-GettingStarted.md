# Начнем!

Давайте посмотрим на архитектуру Codeception. Будем считать, что Codeception уже [установлен](http://yupe.ru/codeception/00-Install.html) и наборы тестов инициализированы. Codeception генерирует три стандартных набора для следующих категорий тестов: модульных (unit), функциональных (functional), и приемочных (acceptance). Все они были описаны в [предыдущей статье](http://yupe.ru/codeception/01-Introduction.htmls). Внутри каталога `/tests` мы имеем три конфигурационных файла и три каталога с названиями, соответствующими наборам тестов. "Наборы тестов" (suites) - это независимые группы тестов, объединенные общей целью. 

## Парни

Основной концепцией Codeception является представление тестов как действий человека. Мы будем называть этого человека "**парнем**" (Guy). У нас есть **CodeGuy**, который выполняет функции/методы и тестирует код. Также у нас есть **TestGuy** - опытный тестировщик, который тестирует приложение целиком и знает кое-что о его внутреннем устройстве. Еще у нас есть **WebGuy**, пользователь, который работает с нашим приложением через интерфейс, который мы предоставляем.

Каждый из этих "парней" представляет из себя PHP класс с набором действий, который он может совершать. Как Каждый из этих "парней" имеет различные способности. Эти способности не постоянны, Вы можете расширять их. Вы даже можете создавать новых "парней", но запомните: один "парень" на один набор тестов.


Классы "парней" не пишутся, они генерируются следующей командой:

~~~
$ php codecept.phar build
~~~

## Напишем простой сценарий

По умолчанию тесты пишутся в виде последовательных сценариев. Чтобы PHP-файл стал корректным сценарием, его имя должно содержать суффикс "Cept". 

Предположим, что мы создали файл  `tests/acceptance/SigninCept.php`

Мы можем сделать это следующей командой:

```
$ php codecept.phar generate:cept acceptance Signin
```

```php
<?php
$I = new WebGuy($scenario);
?>
```

Сценарий всегда начинается с инициализации класса нашего "парня". После этого написание сценария сводится к написанию `$I->` и выбору соответствующего действия из списка авто-дополнения.

Давайте авторизуемся на нашем сайте. Мы предполагаем, что у нас есть страница "login", на которой для авторизации необходимо ввести логин и пароль. После авторизации мы перекидываем пользователя на страницу, где он видит текст `Hello, %username%`. Давайте разберемся как написать этот сценарий, используя Codeception.

```php
<?php
$I = new WebGuy($scenario);
$I->wantTo('log in as regular user');
$I->amOnPage('/login');
$I->fillField('Username','davert');
$I->fillField('Password','qwerty');
$I->click('Login');
$I->see('Hello, davert');
?>
```

Прежде чем запустить тест, мы должны убедиться, что сайт работает на локальном веб-сервере. Откройте `tests/acceptance.suite.yml` и замените URL на URL Вашего приложения:

``` yaml
config:
    PhpBrowser:
        url: 'http://myappurl.local'
```

Если у Вас нет запущенного веб-сервера Вы можете использовать [PHP Built-in Web Server](http://php.net/manual/en/features.commandline.webserver.php) который доступен, начиная с версии PHP 5.4 

После того как Вы установили корректный URL, Вы можете запустить тест, используя следующую команду:

``` bash
$ php codecept.phar run
```

Вот, что мы должны увидеть на выходе: 

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

Это очень простой сценарий, который Вы можете воспроизвести для своего собственного сайта. Эмулируя действия пользователя, Вы можете протестировать весь Ваш сайт подобным образом.

Просто попробуйте!

## Модули и хелперы

Все действия в Guy-классе берутся из модулей. Команда "build", описанная выше, позволяет Codeception эмулировать множественное наследование. Модули спроектированы так, чтобы выполнять только одно действие на каждый метод. В соответствии с [DRY principle](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself), если Вы используете одни и те же компоненты в различных модулях - Вы можете объединить их и поместить в свой собственный модуль. По умолчанию каждый набор тестов имеет пустой модуль, который может быть использован для расширения Guy-класса. Они расположены в директории "_helpers".

## Bootstrap

Каждый набор тестов имеет свой собственный стартовый (bootstrap) файл.Он расположен в директории, содержащей набор тестов и называется`_bootstrap.php`. Этот файл исполняется перед каждым тестом из соответствующего набора. Любый подготовительные операции для набора тестов следует писать именно в этом файле.

## Тесты
Codeception поддерживает три формата тестов. Помимо уже описанных тестов в  Cept формате, Codeception также может выполнять PHPUnit-тесты для модульного тестирования и смешанные (scenario-unit) тесты в формате "Cest". Они будут описаны в следующих статьях. There is no difference in the way the tests of either format will be run in the suite.

## Важные замечания

Codeception довольно умен когда выполняет тестовые сценарии. Но Вы должны помнить, что Codeception выполняет каждый сценарий два раза: первый раз для анализа и второй раз собственно для исполнения. Таким образом, любой PHP-код, добавленный в тестовый сценарий будет выполнен два раза! Скорее всего в этом нет необходимости. Перед добавлением в тест любого PHP-кода (который не использует объект "$I"), пожалуйста, указывайте стадию, на которой этот код должен быть выполнен

__Повторим еще раз: каждый тест выполняется два раза: первый раз для анализа и второй раз собственно для выполнения.__

Например, этот код будет выполняться на этапе анализа.

```php
<?php
if ($scenario->preload()) {
    Logger::log('analyzing');
}
?>
```

а этот - во время выполнения теста:

```php
<?php
if ($scenario->running()) {
    Logger::log('running');
}
?>
```

Вы должны стараться делать Ваши тесты очень простыми и по возможности избегать использования произвольного PHP-кода. Если это возможно - используйте только объект "$I", пишите простые и понятные тесты.

## Конфигурация

Codeception имеет глобальный файл конфигурации `codeception.yml` и отдельный конфигурационный файл для каждого набора тестов. We also support `.dist` configuration files. Если у Вас в проекте несколько разработчиков - разместите общие параметры конфигрурации в файле `codeception.dist.yml`, а индивидуальные настройки каждого разработчика в файле `codeception.yml`. Тоже самое справедливо и для конфигураций наборов тестов. Например, файл `unit.suite.yml` будет объединен с файлом `unit.suite.dist.yml`. 

По умолчанию главный файл конфигурации выгляди приблизительно вот так:

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

Конфигурация для приемочных (acceptance) тестов acceptance.yml

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


## Запуск тестов

Тесты могут быть запущены командой 'run'.

```bash
$ php codecept.phar run
```

Если передать один аргумент - можно запустить тесты только из указанного набора.

```bash
$ php codecept.phar run acceptance
```

Чтобы запустить только один тест - передайте второй аргумент. Указывайте локальный путь, начиная от директории с набором тестов.

```bash
$ php codecept.phar run acceptance SigninCept.php
```

Существует множество опций, которые Вы можете использовать.

* `steps` - все выполняемые действия будут выводиться в консоль.
* `debug` - будет выводиться дополнительная отладочная информация.
* `config` - можно указать специальный конфигурационный файл для этого запуска.
* `colors` - включить цвета (если поддерживает терминал)
* `silent` - выводить только минимальную информацию.
* `report` - выводить результат в виде отчета.
* `coverage` - собирать статистику о покрытии тестами
* `no-exit` - не отдавать код завершения. Это может быть полезно при исползовании Codeception с CI серверами, например с Bamboo.

Следующие опции позволяют сохранить вывод результатов в наиболее подходящих форматах:

* `html` - генерировать HTML-файл с результатами. Файл будет сохранен как 'report.html' в директории tests/_log.
* `xml` - сохранить вывод в формате JUnit для CI Серверов. Файл будет сохранен как 'report.xml' в директории tests/_log.
* `tap` - сохранить вывод в формате TAP. Файл будет сохранен как 'report.tap.log' в директории tests/_log.
* `json` - сохранить вывод в формате JSON. Файл будет сохранен как 'report.json' в директории tests/_log.

Пример:

```bash
$ php codecept.phar run --steps --xml --html
```

Эта команда запустит все тесты из всех тестовых наборов, все шаги будут выводиться на консоль, результаты тестов буду сохранены в HTML и XML файлах-отчетах.

### Генераторы

Codeception имеет множество полезных команд для генерации всего и вся.

* `generate:cept` *suite* *filename* - Генерировать  Cept сценарий.
* `generate:cest` *suite* *filename* - Генерировать  Cest тест.
* `generate:test` *suite* *filename* - Генерировать  PHPUnit Test c Codeception хуками.
* `generate:phpunit` *suite* *filename* - Генерировать классический PHPUnit Test.
* `generate:suite` *suite* *guy* - Генерировать новый набор тестов с указанным Guy-классом.
* `generate:scenarios` *suite* - Генерировать текстовый файл содержащий сценарии из тестов.


## Заключение

Мы рассмотрели структуру Codeception. Большая часть вещей, которые вам понадобятся уже сгенерированы командой `bootstrap`. После того как вы рассмотрели базовые понятия и конфигурацию, вы можете приступить к написанию своего первого сценария.




Трудились и переводили ребята из [amyLabs](http://amylabs.ru/)
