<?php
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

use Bitrix\Main\Application;

$connection = Application::getConnection();
$config = $connection->getConfiguration();

echo "<h3>🔑 Данные подключения к БД:</h3>";
echo "Тип: " . htmlspecialchars($connection->getType()) . "<br>";
echo "Хост: " . htmlspecialchars($config['host']) . "<br>";
echo "База: " . htmlspecialchars($config['database']) . "<br>";
echo "Логин: " . htmlspecialchars($config['login']) . "<br>";
echo "Пароль: " . htmlspecialchars($config['password']) . "<br>";
