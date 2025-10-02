<?php
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

use Bitrix\Main\Application;

$connection = Application::getConnection();

// базовая инфа о соединении
$config = $connection->getConfiguration();

echo "<h3>🔑 Данные подключения к БД:</h3>";
echo "Тип: " . htmlspecialchars($connection->getType()) . "<br>";
echo "Хост: " . htmlspecialchars($config['host']) . "<br>";
echo "База: " . htmlspecialchars($config['database']) . "<br>";
echo "Логин: " . htmlspecialchars($config['login']) . "<br>";
echo "Пароль: " . htmlspecialchars($config['password']) . "<br><br>";

try {
    // запрос версии PostgreSQL/Postgres Pro
    $res = $connection->query("SELECT version() AS ver")->fetch();
    echo "<h3>✅ Соединение успешно!</h3>";
    echo "Версия сервера БД: " . htmlspecialchars($res['ver']) . "<br>";

    // попробуем вытащить параметры Postgres Pro (если есть)
    $res2 = $connection->query("
        SELECT 
            current_setting('pgpro_version', true) AS pgpro_version,
            current_setting('pgpro_edition', true) AS pgpro_edition
    ")->fetch();

    if (!empty($res2['pgpro_version']) || !empty($res2['pgpro_edition'])) {
        echo "Postgres Pro edition: " . htmlspecialchars($res2['pgpro_edition']) . "<br>";
        echo "Postgres Pro version: " . htmlspecialchars($res2['pgpro_version']) . "<br>";
    } else {
        echo "Скорее всего это обычный PostgreSQL (pgpro_* параметры отсутствуют).<br>";
    }

} catch (\Exception $e) {
    echo "<h3>❌ Ошибка подключения:</h3>";
    echo htmlspecialchars($e->getMessage());
}
