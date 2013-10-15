# Тестирование Веб Сервисов

Точно так же, как вы тестируете свой сайт, Codeception позволяет тестировать веб сервисы. Достаточно сложно тестировать их вручную, поэтому автоматизация тестирования в данном случае, является достаточно хорошей идеей.
В Codeception имеется поддержка двух стандартов, SOAP и REST, представленных соответствующими модулями, которые и будут рассмотрены в данной статье.

Для начала, вам необходимо создать новый тестовый набор, который не предоставляется командой `bootstrap`. Мы рекомендуем назвать его **api** и использовать для него класс ApiGuy.

```
$ php codecept.phar generate:suite api ApiGuy
```

Там мы будем вставлять все наши api тесты.


## REST

REST веб сервис работает через HTTP используя стандартные методы: GET, POST, PUT (PATCH), DELETE. Они позволяют получить и управлять данными сервиса. Использование веб сервисов требует наличия HTTP клиента, таким образом вам необходим модуль `PhpBrowser` или один из настроенных модулей доступных фреймворков. К примеру мы можем использовать модуль Symfony2 для Symfony2 приложений с целью игнорирования веб сервера и тестирования веб сервиса изнутри.

Конфигурация модулей в `api.suite.yml`:

``` yaml
class_name: ApiGuy
modules:
    enabled: [PhpBrowser, REST, ApiHelper]
    config:
		PhpBrowser:
			url: http://serviceapp/
		REST:
		    url: http://serviceapp/api/v1/
```

Модуль REST автоматически подключится к PhpBrowser. В случае исползования Symfony2, Zend, или модулей других фреймворков, он будет подключаться к ним. Не забудьте выполнить команду `build` после того, как закончите редактировать конфигурацию.

Создадим первый простой тест:

```
php codecept.phar generate:cept api CreateUser
```

Назовем его `CreateUserCept.php`. Будем использовать его для создания пользователя используя веб сервис.

``` php
<?php
$I = new ApiGuy($scenario);
$I->wantTo('create a user via API');
$I->amHttpAuthenticated('service_user','123456');
$I->haveHttpHeader('Content-Type','application/x-www-form-urlencoded');
$I->sendPOST('users', array('name' => 'davert', 'email' => 'davert@codeception.com'));
$I->seeResponseCodeIs(200);
$I->seeResponseIsJson();
$I->seeResponseContains('{ result: ok}');
?>
```

Модуль REST разработан для использования с сервисами оперирующими данными в JSON формате. К примеру метод `seeResponseContainsJson` конвертирует указанный массив в json и проверит его наличие в ответе от сервиса.

Возможно вам захочется использовать более сложные утверждения `assertions` для проверки ответов. Это может быть реализовано с помощью написания собственных методов в клаccах [Helper](http://codeception.com/docs/03-Modules#helpers). Для доступа к последнему JSON ответу вам необходимо получить свойство `response` модуля REST. Продемонстрируем это на примере метода `seeResponseIsHtml`:

``` php
<?php
class ApiHelper extends \Codeception\Module {

	public function seeResponseIsHtml()
	{
		$response = $this->getModule('REST')->response;
		\PHPUnit_Framework_Assert::assertRegex('^<html>.*?<\/html>$', $response);
	}
}
?>
```

Точно так же вы можете передавать параметры запроса и заголовки.

## SOAP

Веб сервисы SOAP являются более сложным случаем. Вам необходим PHP [собранный с поддержкой SOAP](http://php.net/manual/en/soap.installation.php). Кроме того необходимо хорошее знание XML. SOAP модуль использует специально форматированный POST запрос для подключения к WSDL веб сервисам. Codeception использует PhpBrowser или один из модулей фреймворков для реализации данного взаимодействия. Если вы решите использовать один из модулей фреймворков, модуль SOAP будет автоматически подключен к фреймворку, и вы сможете увеличить скорость выполнения тестов а так же получить более детализированную трассировку и вывод.

Сконфигурируем модуль SOAP для использования с PhpBrowser:

``` yaml

class_name: ApiGuy
modules:
    enabled: [PhpBrowser, SOAP, ApiHelper]
    config:
		PhpBrowser:
			url: http://serviceapp/
		SOAP:
		    endpoint: http://serviceapp/api/v1/
```

Запрос SOAP может содержать специфическую для приложения информацию, к примеру информацию об аутерификации или об оплате. Эта информация обеспечивается SOAP заголовком внутри элемента `<soap:Header>` XML запроса. В случае, если вам нужно сгенерировать такой заголовок, вы можете использовать действие `haveSoapHeader`. К примеру, следующая строка кода

``` php
<?php
$I->haveSoapHeader('Auth', array('username' => 'Miles', 'password' => '123456'));
?>
```
сгенерирует данный XML заголовок

``` xml
<soap:Header>
<Auth>
	<username>Miles</username>
	<password>123456</password>
</Auth>
</soap:Header>
```

Используйте метод `sendSoapRequest` для определения тела вашего запроса.

``` php
<?php
$I->sendSoapRequest('CreateUser', '<name>Miles Davis</name><email>miles@davis.com</email>');
?>
```

Этот вызов будет конвертирован в следующий XML код:

``` xml
<soap:Body>
<ns:CreateUser>
	<name>Miles Davis</name>
	<email>miles@davis.com</email>
</ns:CreateUser>
</soap:Body>
```

А вот список простых утверждений которые могут быть использованы с SOAP.

``` php
<?php
$I->seeSoapResponseEquals('<?xml version="1.0"?><error>500</error>');
$I->seeSoapResponseIncludes('<result>1</result>');
$I->seeSoapResponseContainsStructure('<user><name></name><email></email>');
$I->seeSoapResponseContainsXPath('//result/user/name[@id=1]');
?>
```

В случае, если вы не хотите использовать блинные XML строки, подумайте об использовании [XmlBuilder](http://codeception.com/docs/reference/xmlbuilder) класса. Он поможет вам создавать сложные XML записи в стиле jQuery.
В следующем примере покажем использование `XmlBuilder` (созданного с помощью фабрики SoapUtils) вместо использования XML.

``` php
<?php
use \Codeception\Utils\Soap;

$I = new ApiGuy($scenario);
$I->wantTo('create user');
$I->haveSoapHeader('Session', array('token' => '123456'));
$I->sendSoapRequest('CreateUser', Soap::request()->user->email->val('miles@davis.com'));
$I->seeSoapResponseIncludes(Soap::response()
	->result->val('Ok')
		->user->attr('id', 1)
);
?>
```
Решение о том, что использовать, XmlBuilder или XML строки остается за вами. XmlBuilder возвращает XML строки, как вы можете видеть.

ВЫ можете расширить существующую функциональность при помощи использования модуля SOAP в вашем классе помощнике. Для доступа к такому SOAP ответу как `\DOMDocument` вы можете использовать свойство `response` модуля SOAP.

``` php
<?php
class ApiHelper extends \Codeception\Module {

	public function seeResponseIsValidOnSchema($schema)
	{
		$response = $this->getModule('SOAP')->response;
		$this->assertTrue($response->schemaValidate($schema));
	}
}
?>
```

## Заключение

Codeception имеет два модуля позволяющих тестировать большинство веб сервисов. Для тестирования, необходимо создать новый набор тестов `api`. Помните, что вы не ограничены в том, чтобы тестировать только тело ответа. Например включив модуль Db вы можете проверить, был ли создан пользователь после вызова `CreateUser`. Вы можете улучшить тестовые сценарии используя REST или SOAP ответы в своих методах помощников.