# EPAGES-CS-1 - E-PAGES Coding Standart 1

[psr-4]: http://www.php-fig.org/psr/psr-4/
[local-folder]: http://dev.1c-bitrix.ru/community/blogs/vad/local-folder.php
[composer-autoload]: https://getcomposer.org/doc/01-basic-usage.md#autoloading
[phinx]: https://github.com/robmorgan/phinx

# Основной стандарт кодирования

## Содержимое
- [1. Форматирование](#1-Форматирование)
- [2. Расположение кода](#2-Расположение-кода)
    - [2.1. Директория local](#21-Директория-local)
    - [2.2. Composer](#22-composer)
    - [2.3. Организация автозагрузки классов ВНЕ МОДУЛЯ](#23-Организация-автозагрузки-классов-ВНЕ-МОДУЛЯ)
    - [2.3. Миграции баз данных](#23-Миграции-баз-данных)
    - [2.4. Обработчики событий](#24-Обработчики-событий)
- [3. Базовые правила при разработке под Битрикс](#3-Базовые-правила-при-разработке-под-Битрикс)
    - [3.1. Избегать нагромождение кода в `init.php`](#31-Избегать-нагромождение-кода-в-initphp)
    - [3.2. Константы](#32-Константы)

## 1. Форматирование
НЕОБХОДИМО придерживаться [EPAGES-CS-2](EPAGES-CS-2.md).

## 2. Расположение кода

### 2.1. Директория local

НЕОБХОДИМО, чтобы работа по новому функционалу производилась в директории `local`, которая расположена в корне сайта.
[Подробнее.][local-folder]

### 2.2. [Composer][composer-autoload]

РЕКОМЕНДУЕТСЯ располагать `composer.json` непосредственно в директории `local`.

### 2.3. Организация автозагрузки классов ВНЕ МОДУЛЯ

Рекомендуемый вариант размещения классов:
директория `/local/lib/[Vendor]`.
Например, для `\Epages` путь от корня сайта будет `/local/lib/Epages/`

Пример организации автозагрузки классов ВНЕ МОДУЛЯ:

Структура директорий:

- /local/
    - lib/
        - Epages/
            - SomeModule/
                - SomeClass1.php
            - SomeClass2.php
- autoloader.php

РЕКОМЕНДУЕТСЯ использовать [Composer][composer-autoload] для автозагрузки.

Для сторонних библиотек не совместимых с [PSR-4][psr-4]
РЕКОМЕНДУЕТСЯ использовать автозагрузку на основе карты классов (`classmap`).

Пример `composer.json`:

```json
{
    "autoload": {
        "psr-4": {"Epages\\": "lib/Epages"},
        "classmap": ["lib/no-psr/"]
    }
}
```

Подключение класса автозагрузки в `init.php`:
```php
<?php

require_once $_SERVER['DOCUMENT_ROOT'].'/local/vendor/autoload.php';
```

Пример вызова класса `SomeClass1`:

```php
<?php

//Будет автоматически подключен файл /local/lib/Epages/SomeModule/SomeClass1.php
$ob = new \Epages\SomeModule\SomeClass1();
```

Пример вызова класса `SomeClass2`:

```php
<?php

//Будет автоматически подключен файл /local/lib/Epages/SomeClass2.php
$ob = new \Epages\SomeClass2();
```

### 2.3. Миграции баз данных

Для миграций РЕКОМЕНДУЕТСЯ использовать [Phinx][phinx].

`composer require robmorgan/phinx`

Файлы миграций РЕКОМЕНДУЕТСЯ хранить в директории `/local/migrations/`

Пример файла конфигурации Phinx `/local/phinx.php`

```php
<?php

define('NOT_CHECK_PERMISSIONS', true);
define('NO_AGENT_CHECK', true);
$GLOBALS['DBType'] = 'mysql';
$_SERVER['DOCUMENT_ROOT'] = realpath(__DIR__.'/..');
include $_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_before.php';
// manual saving of DB resource
global $DB;
$app = \Bitrix\Main\Application::getInstance();
$con = $app->getConnection();
$DB->db_Conn = $con->getResource();
// "authorizing" as admin
$_SESSION['SESS_AUTH']['USER_ID'] = 1;

$config = include realpath(__DIR__.'/../bitrix/.settings.php');

return array(
    'paths' => array(
        'migrations' => realpath(__DIR__.'/migrations/'),
    ),
    'environments' => array(
        'default_migration_table' => 'phinxlog',
        'default_database' => 'dev',
        'dev' => array(
            'adapter' => 'mysql',
            'host' => $config['connections']['value']['default']['host'],
            'name' => $config['connections']['value']['default']['database'],
            'user' => $config['connections']['value']['default']['login'],
            'pass' => $config['connections']['value']['default']['password'],
        ),
    ),
);
```

Для создании миграции нужно выполнить команду:

```
cd [корневая директория проекта]/local/
vendor/bin/phinx create [название класса миграции]
```

Примечание: в Windows надо вместо `vendor/bin/phinx` использовать `vendor\bin\phinx`

Далее в появившемся файле НУЖНО имплементировать методы `up()` и `down()`, которые будут выполняться при применении и откате миграции соответственно.

Например:
```php
<?php

use Phinx\Migration\AbstractMigration;
use Bitrix\Main\Loader;

/**
 * Create 'new' order property.
 */
class CreateOrderPropertyNew extends AbstractMigration
{
    public function up()
    {
        Loader::includeModule('sale');
        //create order property
    }

    public function down()
    {
        Loader::includeModule('sale');
        //delete order property
    }
}
```

Для запуска миграции: `vendor/bin/phinx migrate`

Для отката миграции: `vendor/bin/phinx rollback`

### 2.4. Обработчики событий
РЕКОМЕНДУЕТСЯ обработчики событий размещать в классах, таким образом:
`\Epages\[модуль]\Event\[событие]`

Пример подключения:
```php
AddEventHandler(
    'iblock',
    'OnBeforeIBlockElementUpdate',
    array('\Epages\IBlock\Event\OnBeforeIBlockElementUpdate', 'doSomethingCool')
);
```

## 3. Базовые правила при разработке под Битрикс

### 3.1. Избегать нагромождение кода в `init.php`

- при добавлении кода в файл init.php РЕКОМЕНДУЕТСЯ выносить логически сгруппированный код в отдельные классы/файлы
и подключать их внутри `init.php`.

- обработчики событий РЕКОМЕНДУЕТСЯ располагать в `init.php` и группировать по модулю.

### 3.2. Константы

- НЕ РЕКОМЕНДУЕТСЯ использовать цифровые значения в GetList, GetByID и схожих методах, которые принимают различные ID.
- РЕКОМЕНДУЕТСЯ создать файл со всеми необходимыми константами и вызывать их имена.
- У каждой константы ДОЛЖНО быть "говорящее" имя и комментарий.


**Не правильно:**
```php
<?php
$comments = CIBlockElement::GetList(array(), array('IBLOCK_ID' => 12));
```

**Правильно: Создаем файл constants.php и указываем в нем:**
```php
<?php
//ИБ с комментариями пользователей
const COMMENTS_IBLOCK_ID = 12;
```

**Подключаем этот файл в init.php**
```php
<?php
//Константы проекта
include_once($_SERVER['DOCUMENT_ROOT'].'/local/constants.php');
```

**Используем константу**
```php
<?php
$comments = CIBlockElement::GetList(array(), array('IBLOCK_ID' => COMMENTS_IBLOCK_ID));
```
- при выборках данных (например, GetList) ОБЯЗАТЕЛЬНО указывать поля, которые нужны для дальнейших манипуляций,
 кроме случаев, когда нужны все поля
- при необходмости выбрать несколько элементов по ID, ОБЯЗАТЕЛЬНО использовать GetList вместо GetByID

**Не правильно:**

```php
<?php
$element1 = CIBlockElement::GetByID(1);
$element2 = CIBlockElement::GetByID(2);
```

**Правильно:**
```php
<?php
$elements = CIBlockElement::GetList(array(), array('ID' => array(1, 2)));
```

- НЕ РЕКОМЕНДУЕТСЯ использовать прямые запросы к базе данных без крайней необходимости
- если к файлу не предусмотрен прямой доступ ОБЯЗАТЕЛЬНО в первой строке файла добавить

```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
    die();
}
```
