# EPAGES-CS-1 - E-PAGES Coding Standart 1

<h1>Основной стандарт кодирования</h1>

Этот раздел стандарта описывает какие элементы кода следует считать стандартными, необходимыми для обеспечения высокого уровня технического взаимодействия между общим (разделяемым) кодом PHP.

<h2>1. Обзор</h2>
<ul>
    <li>В файлах ВОЗМОЖНО использовать теги <code>&lt;?php</code>, <code>&lt;?=</code>, а так же <code>&lt;?</code>.</li>
    <li>В файлах с кодом PHP НЕОБХОДИМО использовать только UTF-8 без BOM (для новых проектов).</li>
    <li>Разработка НОВЫХ классов, шаблонов, компонетнов, модулей ДЛЯ ВСЕХ проектов ДОЛЖНА вестись в папке <code>/local/</code></li>
    <li>НЕОБХОДИМО, чтобы пространства имён и классы ВНЕ МОДУЛЯ соответствовали стандартам PSR, описывающим автозагрузку: [<a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-0.md">PSR-0</a>, <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>].</li>
    <li>Базовые правила при разработке под Битрикс</li>
</ul>

<h2>2. Файлы</h2>
<h3>2.1. Теги PHP</h3>

В коде PHP ВОЗМОЖНО использовать полные теги <code>&lt;?php</code> или сокращённую форму <code>&lt;?=</code>. Так же ВОЗМОЖНО использование тега <code>&lt;?</code>

<h3>2.2. Кодировка символов</h3>

Для кода PHP НЕОБХОДИМО использовать только UTF-8 без BOM. Уловие ОБЯЗАТЕЛЬНО для новых проектов.

<h2>3. Папка <code>/local/<code></h2>

НЕОБХОДИМО, чтобы работа по новому функционалу производилась в папке <code>/local/</code>. 
Документация - <a href="http://dev.1c-bitrix.ru/community/blogs/vad/local-folder.php">/local/</a>

<h2>4. Организация автозагрузки классов ВНЕ МОДУЛЯ</h2>

НЕОБХОДИМО, чтобы пространства имён и классы соответствовали стандартам "автозагрузки" [<a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-0.md">PSR-0</a>, <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>].

Это значит что каждый класс должен располагаться в отдельном файле и находиться в пространстве имён как минимум первого уровня — соответствующем названию разработчика.

Пример организации автозагрузки классов ВНЕ МОДУЛЯ используя папку <code>/local/</code>:

Структура папки <code>/local/</code>:
<ul>
	<li>/local/</li>
	<ul>
		<li>/lib/</li>
		<ul>
			<li>/Epages/</li>
			<ul>
				<li>CTestClass.php</li>
			</ul>
		</ul>
	</ul>
	<li>autoload.php</li>
</ul>

Пример файла autoload.php:

```php
<?php
function localAutoload($className)
{
    $className = ltrim($className, '\\');
    $fileName  = '';
    $namespace = '';
    if ($lastNsPos = strrpos($className, '\\')) 
    {
        $namespace = substr($className, 0, $lastNsPos);
        $className = substr($className, $lastNsPos + 1);
        $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
    }
    $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';
    include_once __DIR__ . "/lib/" . $fileName;
}
spl_autoload_register('localAutoload');
?>
```

Подключение функции автозагрузки в Битрикс:
```php
<?php
//init.php
// ...

require_once($_SERVER["DOCUMENT_ROOT"]."/local/autoload.php");

// ...
?>
```

Пример вызова класса <code>CTestClass</code>:

```php
<?php
// Класс подключать не нужно, он будет автоматически подгружен функцией localAutoload()
$ob = new \Epages\CTestClass();
?>
```

<h2>5. Базовые правила при разработке под Битрикс</h2>
<ul>
    <li>при добавлении кода в файл init.php РЕКОМЕНДУЕТСЯ выносить логически сгруппированный код в отдельные файлы и подключать их внутри init.php</li>
    <li>НЕ РЕКОМЕНДУЕТСЯ использовать цифровые значения в GetList, GetByID и схожих методах, которые принимают различные ID. РЕКОМЕНДУЕТСЯ создать файл со всеми необходимыми константами и вызывать их имена. У каждой константы ДОЛЖНО быть «говорящее» именование и комментарий.</li>
</ul>

<b>Не правильно:</b>
```php
<?php
$comments = CIBlockElement::GetList(Array(), Array("IBLOCK_ID" => 12));
?>
```

<b>Правильно: Создаем файл constants.php и указываем в нем:</b>
```php
<?php
//ИБ с комментариями пользователей
const COMMENTS_IBLOCK_ID = 12;
?>
```

<b>Подключаем этот файл в init.php</b>
```php
<?php
//Константы проекта
include_once($_SERVER["DOCUMENT_ROOT"] . '/bitrix/php_interface/includes/constants.php');
?>
```

<b>Используем константу</b>
```php
<?php
$comments = CIBlockElement::GetList(Array(), Array("IBLOCK_ID" => COMMENTS_IBLOCK_ID));
?>
```
<ul>
    <li>при выборках данных (например, GetList) ОБЯЗАТЕЛЬНО указывать поля, которые нужны для дальнейших манипуляций, кроме случаев, когда нужны все поля</li>
    <li>при необходмости выбрать несколько элементов по ID, ОБЯЗАТЕЛЬНО использовать GetList вместо GetByID</li>
</ul>

<b>Не правильно:</b>
```php
<?php
$element1 = CIBlockElement::GetByID(1);
$element2 = CIBlockElement::GetByID(2);
?>
```

<b>Правильно:</b>
```php
<?php
$elements = CIBlockElement::GetList(Array(), Array("ID" => Array(1, 2)));
?>
```

<ul>
    <li>НЕ РЕКОМЕНДУЕТСЯ использовать прямые запросы к базе данных без крайней необходимости</li>
    <li>если к файлу не предусмотрен прямой доступ ОБЯЗАТЕЛЬНО в первой строке файла добавить</li>
</ul>
```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();
?>
```
