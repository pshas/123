<?php
$host = "localhost";   // или IP сервера
$port = "5432";        // порт Postgres Pro
$dbname = "sitemanager"; // имя базы
$user = "bitrix";      // логин
$password = "supersecret"; // пароль

try {
    // Формируем DSN
    $dsn = "pgsql:host=$host;port=$port;dbname=$dbname;";
    
    // Создаем подключение
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]);

    echo "<h3>✅ Подключение успешно!</h3>";

    // Проверим версию Postgres Pro
    $stmt = $pdo->query("SELECT version() AS ver;");
    $row = $stmt->fetch();
    echo "Версия сервера БД: " . htmlspecialchars($row['ver']) . "<br>";

} catch (PDOException $e) {
    echo "<h3>❌ Ошибка подключения:</h3> " . htmlspecialchars($e->getMessage());
}
