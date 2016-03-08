# EPAGES-CS-2 - E-PAGES Coding Standart 2

[psr-1]: http://www.php-fig.org/psr/psr-1/
[psr-2]: http://www.php-fig.org/psr/psr-2/

# Форматирование кода

Этот раздел стандарта описывает правила форматирования кода на PHP.

Цель данного руководства — уменьшить когнитивное сопротивление при
визуальном восприятии кода, написанного разными авторами.

## Содержимое
- [1. Базовый стандарт](#1-Базовый-стандарт)
- [2. Именование свойств класса, переменных и функций вне класса](#2-Именование-свойств-класса-переменных-и-функций-вне-класса)
- [3. Операторы](#3-Операторы)
- [4. Правила переноса строки в выражениях](#4-Правила-переноса-строки-в-выражениях)
- [5. Строковые литералы](#5-Строковые-литералы)
- [6. Массивы](#6-Массивы)
- [7. Прочее](#7-Прочее)
- [8. Форматирование в шаблонах](#8-Форматирование-в-шаблонах)

## 1. Базовый стандарт

НЕОБХОДИМО придерживаться [PSR-1: Basic Coding Standard][psr-1] и [PSR-2: Coding Style Guide][psr-2].

## 2. Именование свойств класса, переменных и функций вне класса

РЕКОМЕНДУЕТСЯ использовать `$camelCase`.

## 3. Операторы

РЕКОМЕНДУЕТСЯ
- операторы от операндов отделять пробелом, за исключением оператора конкатенации
- размещать унарные операторы слитно с затронутым ими выражением
- всегда использовать операторы тождественного сравнения (===, !==) если не требуется неявное преобразование типов

**Пример**
```php
<?php
if ('cli' === $mode
        && !$forceHtml) {
    $lineEnding = "\n";
} else {
    $lineEnding = '<br>';
}
$text .= $line.$lineEnding;
```

## 4. Правила переноса строки в выражениях

- переносить можно после запятой или перед оператором;
- первый перенос ДОЛЖЕН быть сдвинут относительно верхней строки на 1 отступ;
- если в выражении несколько переносов, то следующие переносы выравниваются на уровне первого;


**Пример**

```php
<?php
if (COption::GetOptionString('main', 'new_user_registration', 'N') == 'Y'
        && $_SERVER['REQUEST_METHOD'] == 'POST' && $TYPE == 'REGISTRATION'
        && (!defined('ADMIN_SECTION') || ADMIN_SECTION !== true)) {
    //body
}

if (
    COption::GetOptionString('main', 'new_user_registration', 'N') == 'Y'
    && $_SERVER['REQUEST_METHOD'] == 'POST' && $TYPE == 'REGISTRATION'
    && (!defined('ADMIN_SECTION') || ADMIN_SECTION !== true)
) {
    //body
}
```

Очень длинные/сложные управляющие структуры рекомендуется разбивать на несколько более простых.

Например,
```php
<?php
$publicStatisticOnly = false;
if (
	defined('STATISTIC_ONLY')
	&& STATISTIC_ONLY
	&& substr($APPLICATION->GetCurPage(), 0, strlen(BX_ROOT.'/admin/')) !== BX_ROOT.'/admin/'
) {
	$publicStatisticOnly = true;
}

if (
	!$publicStatisticOnly
	&& strlen(LANG_CHARSET) > 0
	&& COption::GetOptionString('main', 'include_charset', 'Y') == 'Y'
) {
    //body
}
```

## 5. Строковые литералы

РЕКОМЕНДУЕТСЯ использовать одинарные кавычки для строковых литералов, которые не требуют использования двойных кавычек.

**Пример**
```php
<?php
$someString = 'some string';
$anotherString = "another string\n";
$yetAnotherString = "some variable: $someVariable";
```



## 6. Массивы

РЕКОМЕНДУЕТСЯ запятую после последнего элемента при:
- однострочной записи - не ставить
- многострочной записи - ставить

```php
<?php
$select = array('ID', 'NAME', 'SECTION_ID');

$filter = array(
	'key1' => 'value1',
	'key2' => array(
		'key21' => 'value21',
		'key22' => 'value22',
	),
);
```

Табличное форматирование с помощью символов табуляции НЕ ДОЛЖНО использоваться.

Пример неправильного форматирования:
```php
<?php
$array = array(
	'key1-really-long' => 'value1',
	'key2'			   => 'value2',
);
?>
```

## 7. Прочее
В сложных выражениях рекомендуется группировать операции с помощью скобок вне зависимости от того, требует это приоритет операций или нет.

Пример:
```php
<?php
$r = $a || ($b && $c);
```

## 8. Форматирование в шаблонах
Управляющие структуры `if()`, `foreach()`, `while()` и т.д. в шаблоне компонента вместе с html-версткой
НЕ ДОЛЖНЫ использовать фигурные скобки.
СЛЕДУЕТ заменить фигурные скобки
[альтернативным синтаксисом управляющих структур](http://php.net/manual/ru/control-structures.alternative-syntax.php)

Пример плохого кода, где из-за фигурных скобок пострадала читабельность:
```php
<?php if ($condition === true) {?>
	<a href="/programm.php">Link text</a>
<?php } elseif ($condition === true) {?>
	<a href="/programm.php">Link text</a>
<?php } else {?>
	<a href="/programm.php">Link text</a>
<?php }?>
```

Примеры правильного кода:
```php
<?php if ($condition === true):?>
	<a href="<?=$link?>">Link text</a>
<?php elseif ($condition === true):?>
	<a href="/programm.php">Link text</a>
<?php else:?>
	<a href="/programm.php">Link text</a>
<?php endif;?>
```
- НЕ РЕКОМЕНДУЕТСЯ выводить элементы верстки используя конструкцию языка `echo` или функцию `print`. СЛЕДУЕТ прервать PHP-код тэгом `?>` и разместить после него нужные html-элементы
- НЕ РЕКОМЕНДУЕТСЯ выносить связные элементы верстки за шаблон компонента

Пример плохого кода:
```php
<div class="navbar">
	<ul class="top-menu">
		<?php
		$APPLICATION->IncludeComponent(
			'bitrix:menu',
			'top',
			array(
				'ROOT_MENU_TYPE' => 'top',
				'MENU_CACHE_TYPE' => 'A',
				'MENU_CACHE_TIME' => '36000000',
				'MENU_CACHE_USE_GROUPS' => 'Y',
				'MENU_CACHE_GET_VARS' => array(),
				'MAX_LEVEL' => '1',
				'USE_EXT' => 'N',
				'ALLOW_MULTI_SELECT' => 'N',
			)
		);
		?>
	</ul>
</div>
```
Правильно будет перенести `<ul class="top-menu">` в шаблон
