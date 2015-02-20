# EPAGES-CS-2 - E-PAGES Coding Standart 2

<h1>Форматирование кода</h1>

Этот раздел стандарта описывает как принято оформлять код на PHP работая в 1С-Битрикс

<h2>1. Обзор</h2>
<ul>
    <li>Структурирование текста</li>
    <li>Инструкции, выражения</li>
    <li>Пустые строки и пробелы</li>
    <li>Прочее</li>
</ul>

<h2>2. Структурирование текста</h2>
<h3>2.1. Длина строки</h3>
Нужно стараться избегать строк длиной более 120 символов. Если строка превышает этот размер, то нужно использовать правила переноса строки.

<h3>2.2. Правила переноса строки</h3>
Если длина строки превышает 120 символов, то необходимо пользоваться следующими правилами переноса:
<ul>
    <li>переносить можно после запятой или перед оператором;</li>
    <li>переносимая строка должна быть сдвинута относительно верхней на один символ табуляции;</li>
</ul>

<b>Пример 1:</b>

для строки
```php
<?php
$arAuthResult = $USER->ChangePassword($USER_LOGIN, $USER_CHECKWORD, $USER_PASSWORD, $USER_CONFIRM_PASSWORD, $USER_LID);
```

допустимым будет следующий вариант переноса:
```php
<?php
$arAuthResult = $USER->ChangePassword($USER_LOGIN, $USER_CHECKWORD,
$USER_PASSWORD, $USER_CONFIRM_PASSWORD, $USER_LID);
```

<b>Пример 2:</b>

для строки
```php
<?php
if(COption::GetOptionString("main", "new_user_registration", "N")=="Y" && $_SERVER['REQUEST_METHOD']=='POST' && 
$TYPE=="REGISTRATION" && (!defined("ADMIN_SECTION") || ADMIN_SECTION!==true))
```

допустимым будет следующий вариант переноса:
```php
<?php
if (COption::GetOptionString("main", "new_user_registration", "N") == "Y"
&& $_SERVER['REQUEST_METHOD'] == 'POST' && $TYPE == "REGISTRATION"
&& (!defined("ADMIN_SECTION") || ADMIN_SECTION !== true))
{
}
```

<h3>2.3. Пробелы и табуляция</h3>
<h3>2.4. Форматирование подчиненности</h3>


<h2>3. Инструкции, выражения</h2>


<h2>4. Пустые строки и пробелы</h2>


<h2>5. Прочее</h2>

