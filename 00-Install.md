# Установка

**Phar**

Скачайте phar-архив Codeception с http://codeception.com/thanks

~~~
wget http://codeception.com/codecept.phar .
~~~

Подготовьте необходимую структуру каталогов и конфигурационных файлов

~~~
php codecept.phar bootstrap
~~~

**Composer**

Установите Composer в корень вашего проекта http://getcomposer.org/

Выполните команду

~~~
php composer.phar require "codeception/codeception:*"
~~~

С этого момента Codeception (с установленным PHPUnit) может быть запущен как:

~~~
vendor/bin/codecept
~~~

Инициализируйте тестовое окружение следующей командой 

~~~
vendor/bin/codecept bootstrap
~~~

**Git**

Дополнительный способ установки для исправления ошибок, участия в разработке Codeception и хакерства

Склонируйте репозиторий с GitHub:

~~~
git clone git@github.com:Codeception/Codeception.git
~~~

С помощью Composer установите все зависимости

~~~
cd Codeception
curl -s http://getcomposer.org/installer | php
php composer.phar install
~~~

Инициализируйте  необходимые каталоги и файлы конфигурации, указав путь к каталогу вашего проекта

~~~
php codecept bootstrap /path/to/my/project
~~~

Для запуска тестов используйте ключ -c для указания директории с тестами.

~~~
php codecept run -c /path/to/my/project
~~~

Если вы хотите собрать phar архив - выполните команду

~~~
php bin/build_phar.php
~~~

Не забудьте прислать pull-request!

Трудились и переводили ребята из [amyLabs](http://amylabs.ru/)