<?php
$host = "localhost";
$port = "3306";
$dbname = "testdb";
$user = "root";
$password = "root";

try {
    // DSN (Data Source Name)
    $dsn = "mysql:host=$host;port=$port;dbname=$dbname;charset=utf8mb4";

    // Подключение
    $pdo = new PDO($dsn, $user, $password, [
        PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION, // ошибки как исключения
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,       // fetch возвращает ассоц.массив
        PDO::ATTR_EMULATE_PREPARES   => false,                  // нативные prepared statements
    ]);

    echo "✅ Подключение к MySQL успешно<br>";

    // Проверим версию сервера
    $stmt = $pdo->query("SELECT VERSION() AS ver");
    $row = $stmt->fetch();
    echo "Версия MySQL: " . htmlspecialchars($row['ver']) . "<br>";

} catch (PDOException $e) {
    echo "❌ Ошибка подключения: " . $e->getMessage() . "<br>";
    echo "Код ошибки: " . $e->getCode() . "<br>";
    echo "<pre>" . $e->getTraceAsString() . "</pre>";
}
