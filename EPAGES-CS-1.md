# EPAGES-CS-1 - E-PAGES Coding Standart 1

<h1>Основной стандарт кодирования</h1>

Этот раздел стандарта описывает какие элементы кода следует считать стандартными, необходимыми для обеспечения высокого уровня технического взаимодействия между общим (разделяемым) кодом PHP.

<h2>1. Обзор</h2>
<ul>
    <li>В файлах ВОЗМОЖНО использовать теги <code>&lt;?php</code>, <code>&lt;?=</code>, а так же <code>&lt;?</code>.</li>
    <li>В файлах с кодом PHP НЕОБХОДИМО использовать только UTF-8 без BOM (для новых проектов).</li>
    <li>Разработка новых классов, шаблонов, компонетнов, модулей ДЛЯ ВСЕХ проеков ДОЛЖНА вестись в папке /local/</li>
    <li>НЕОБХОДИМО, чтобы пространства имён и классы ВНЕ МОДУЛЯ соответствовали стандартам PSR, описывающим автозагрузку: [<a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-0.md">PSR-0</a>, <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>].</li>
</ul>

<h2>2. Файлы</h2>
<h3>2.1. Теги PHP</h3>

В коде PHP ВОЗМОЖНО использовать полные теги <code>&lt;?php</code> или сокращённую форму <code>&lt;?=</code>. Так же ВОЗМОЖНО использование тега <code>&lt;?</code>

<h3>2.2. Кодировка символов</h3>

Для кода PHP НЕОБХОДИМО использовать только UTF-8 без BOM.

<h2>3. Папка /local/</h2>

НЕОБХОДИМО, чтобы работа по новому функционалу производилась в папке <code>/local/</code>. Документация - <a href="http://dev.1c-bitrix.ru/community/blogs/vad/local-folder.php">/local/</a>

<h2>4. Организация автозагрузки классов ВНЕ МОДУЛЯ</h2>

НЕОБХОДИМО, чтобы пространства имён и классы соответствовали стандартам "автозагрузки" [<a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-0.md">PSR-0</a>, <a href="https://github.com/php-fig/fig-standards/blob/master/accepted/ru/PSR-4-autoloader-examples.md">PSR-4</a>].

Это значит что каждый класс должен располагаться в отдельном файле и находиться в пространстве имён как минимум первого уровня — соответствующем названию разработчика.

Пример организации автозагрзки классов ВНЕ МОДУЛЯ используя папку <code>/local/</code>

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
<code>
<?php
function localAutoload($className)
{
    $className = ltrim($className, '\\');
    $fileName  = '';
    $namespace = '';
    if ($lastNsPos = strrpos($className, '\\')) {
        $namespace = substr($className, 0, $lastNsPos);
        $className = substr($className, $lastNsPos + 1);
        $fileName  = str_replace('\\', DIRECTORY_SEPARATOR, $namespace) . DIRECTORY_SEPARATOR;
    }
    $fileName .= str_replace('_', DIRECTORY_SEPARATOR, $className) . '.php';

    include_once __DIR__ . "/lib/" . $fileName;
}
spl_autoload_register('localAutoload');
?>
</code>
