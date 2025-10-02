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
echo "Пароль: " . htmlspecialchars($config['password']) . "<br><br>";

try {
    $result = $connection->query("SELECT version() AS ver;");
    $row = $result->fetch();
    echo "<h3>✅ Соединение успешно!</h3>";
    echo "Версия сервера БД: " . htmlspecialchars($row['ver']) . "<br>";
} catch (\Exception $e) {
    echo "<h3>❌ Ошибка подключения:</h3>";
    echo htmlspecialchars($e->getMessage());
}
