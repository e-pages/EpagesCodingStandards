# EPAGES-CS-1 - E-PAGES Coding Standart 1

<h1>Основной стандарт кодирования</h1>

Этот раздел стандарта описывает какие элементы кода следует считать стандартными, необходимыми для обеспечения высокого уровня технического взаимодействия между общим (разделяемым) кодом PHP.

<h2>1. Обзор</h2>
<ul>
    <li>В файлах ВОЗМОЖНО использовать теги <code>&lt;?php</code>, <code>&lt;?=</code>, а так же <code>&lt;?</code>.</li>
    <li>В файлах с кодом PHP НЕОБХОДИМО использовать только UTF-8 без BOM (для новых проектов).</li>
    <li>Разработка НОВЫХ классов, шаблонов, компонетнов, модулей ДЛЯ ВСЕХ проектов ДОЛЖНА вестись в папке <code>/local/</code></li>
    <li>НЕОБХОДИМО, чтобы пространства имён и классы ВНЕ МОДУЛЯ соответствовали <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>.</li>
    <li>Базовые правила при разработке под Битрикс</li>
</ul>

<h2>2. Файлы</h2>
<h3>2.1. Теги PHP</h3>

В коде PHP ВОЗМОЖНО использовать полные теги <code>&lt;?php</code> или сокращённую форму <code>&lt;?=</code>. Так же ВОЗМОЖНО использование тега <code>&lt;?</code>

<h3>2.2. Кодировка символов</h3>

Для кода PHP НЕОБХОДИМО использовать только UTF-8 без BOM. Уловие ОБЯЗАТЕЛЬНО для новых проектов.

<h2>3. Папка /local/</h2>

НЕОБХОДИМО, чтобы работа по новому функционалу производилась в папке <code>/local/</code>.
Документация - <a href="http://dev.1c-bitrix.ru/community/blogs/vad/local-folder.php">/local/</a>

<h2>4. Организация автозагрузки классов ВНЕ МОДУЛЯ</h2>

НЕОБХОДИМО, чтобы пространства имён и классы соответствовали стандарту "автозагрузки" <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>.

Это значит что каждый класс должен располагаться в отдельном файле и находиться в пространстве имён как минимум первого уровня — соответствующем названию разработчика. Рекомендуемый вариант размещения классов: папка <code>/local/lib/[Vendor]</code>.
Например, для <code>\\Epages</code> путь от корня сайта будет <code>/local/lib/Epages/</code>

Пример организации автозагрузки классов ВНЕ МОДУЛЯ:

Структура папок:
<ul>
    <li>/local/</li>
    <ul>
        <li>lib/</li>
        <ul>
            <li>Epages/</li>
            <ul>
                <li>SomeModule/</li>
                <ul>
                    <li>SomeClass1.php</li>
                </ul>
                <li>SomeClass2.php</li>
            </ul>
        </ul>
    </ul>
    <li>autoloader.php</li>
</ul>

Пример файла autoloader.php:

```php
<?php

/**
 * An example of a general-purpose implementation that includes the optional
 * functionality of allowing multiple base directories for a single namespace
 * prefix.
 */
class Psr4Autoloader
{
    /**
     * An associative array where the key is a namespace prefix and the value
     * is an array of base directories for classes in that namespace.
     *
     * @var array
     */
    protected $prefixes = array();

    /**
     * Register loader with SPL autoloader stack.
     *
     * @return void
     */
    public function register()
    {
        spl_autoload_register(array($this, 'loadClass'));
    }

    /**
     * Adds a base directory for a namespace prefix.
     *
     * @param string $prefix The namespace prefix.
     * @param string $base_dir A base directory for class files in the
     * namespace.
     * @param bool $prepend If true, prepend the base directory to the stack
     * instead of appending it; this causes it to be searched first rather
     * than last.
     * @return void
     */
    public function addNamespace($prefix, $base_dir, $prepend = false)
    {
        // normalize namespace prefix
        $prefix = trim($prefix, '\\');

        // normalize the base directory with a trailing separator
        $base_dir = rtrim($base_dir, DIRECTORY_SEPARATOR) . '/';

        // initialize the namespace prefix array
        if (isset($this->prefixes[$prefix]) === false) {
            $this->prefixes[$prefix] = array();
        }

        // retain the base directory for the namespace prefix
        if ($prepend) {
            array_unshift($this->prefixes[$prefix], $base_dir);
        } else {
            array_push($this->prefixes[$prefix], $base_dir);
        }
    }

    /**
     * Loads the class file for a given class name.
     *
     * @param string $class The fully-qualified class name.
     * @return mixed The mapped file name on success, or boolean false on
     * failure.
     */
    public function loadClass($class)
    {
        // the current namespace prefix
        $prefix = $class;

        // work backwards through the namespace names of the fully-qualified
        // class name to find a mapped file name
        while (false !== $pos = strrpos($prefix, '\\')) {


            $prefix = substr($class, 0, $pos);

            // the rest is the relative class name
            $relative_class = substr($class, $pos + 1);

            // try to load a mapped file for the prefix and relative class
            $mapped_file = $this->loadMappedFile($prefix, $relative_class);
            if ($mapped_file) {
                return $mapped_file;
            }

        }

        // never found a mapped file
        return false;
    }

    /**
     * Load the mapped file for a namespace prefix and relative class.
     *
     * @param string $prefix The namespace prefix.
     * @param string $relative_class The relative class name.
     * @return mixed Boolean false if no mapped file can be loaded, or the
     * name of the mapped file that was loaded.
     */
    protected function loadMappedFile($prefix, $relative_class)
    {
        // are there any base directories for this namespace prefix?
        if (isset($this->prefixes[$prefix]) === false) {
            return false;
        }

        // look through base directories for this namespace prefix
        foreach ($this->prefixes[$prefix] as $base_dir) {

            // replace the namespace prefix with the base directory,
            // replace namespace separators with directory separators
            // in the relative class name, append with .php
            $file = $base_dir
                . str_replace('\\', '/', $relative_class)
                . '.php';

            // if the mapped file exists, require it
            if ($this->requireFile($file)) {
                // yes, we're done
                return $file;
            }
        }

        // never found it
        return false;
    }

    /**
     * If a file exists, require it from the file system.
     *
     * @param string $file The file to require.
     * @return bool True if the file exists, false if not.
     */
    protected function requireFile($file)
    {
        if (file_exists($file)) {
            require $file;
            return true;
        }
        return false;
    }
}
?>
```

Подключение класса автозагрузки в Битрикс:
```php
<?php
//init.php
// ...

include_once $_SERVER["DOCUMENT_ROOT"] . '/local/autoloader.php';
$loader = new Psr4Autoloader;
$loader->register();
$loader->addNamespace('Epages', $_SERVER["DOCUMENT_ROOT"] . '/local/lib/Epages');

// ...
?>
```

Пример вызова класса <code>SomeClass1</code>:

```php
<?php
/* Будет автоматически подключен файл /local/lib/Epages/SomeModule/SomeClass1.php */
$ob = new \Epages\SomeModule\SomeClass1();
?>
```

Пример вызова класса <code>SomeClass2</code>:

```php
<?php
/* Будет автоматически подключен файл /local/lib/Epages/SomeClass2.php */
$ob = new \Epages\SomeClass2();
?>
```
<h3>4.1. Обработчики событий</h3>
РЕКОМЕНДУЕТСЯ обработчики событий размещать в классах, таким образом:
<code>\\Epages\\[модуль]\\Event\\[событие]</code>
Пример подключения:
```php
AddEventHandler(
'iblock',
'OnBeforeIBlockElementUpdate',
array('\Epages\IBlock\Event\OnBeforeIBlockElementUpdate', 'doSomethingCool')
);
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
include_once($_SERVER["DOCUMENT_ROOT"] . '/local/constants.php');
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
    <li>при необходмости выбрать несколько элементов по ID, ОБЯЗАТЕЛЬНО использовать GetList вместо GetByID
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
