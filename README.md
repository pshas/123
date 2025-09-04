<?php
// ====== ВКЛЮЧАЕМ ОТЛАДКУ ======
error_reporting(E_ALL);
ini_set('display_errors', 1);
ini_set('log_errors', 1);
ini_set('error_log', __DIR__ . "/survey_debug.log"); // лог рядом со скриптом

// Функция для отладки в лог
function debug_log($msg) {
    error_log(date("[Y-m-d H:i:s] ") . print_r($msg, true) . "\n", 3, __DIR__."/survey_debug.log");
}

try {
    $pdo = new PDO("mysql:host=localhost;dbname=qr_opros;charset=utf8", "root", "");
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    debug_log("Подключение к БД успешно");
} catch (PDOException $e) {
    die("Ошибка подключения к БД: " . $e->getMessage());
}

$cars = $pdo->query("SELECT * FROM cars")->fetchAll(PDO::FETCH_ASSOC);

// Обработка формы
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    debug_log("POST данные: " . print_r($_POST, true));

    $car_id = (int)$_POST['car_id'];
    $data = [
        'move_time' => (int)$_POST['move_time'],
        'clean' => (int)$_POST['clean'],
        'driver_clean' => (int)$_POST['driver_clean'],
        'driver_tactic' => (int)$_POST['driver_tactic'],
        'move_safety' => (int)$_POST['move_safety'],
        'move_comfort' => (int)$_POST['move_comfort'],
        'smell_cabin' => (int)$_POST['smell_cabin'],
        'volume_music' => (int)$_POST['volume_music'],
        'temperature_cabin' => (int)$_POST['temperature_cabin'],
    ];
    $comment = trim($_POST['comment']);
    $telephone = trim($_POST['telephone']);
    $date = date("Y-m-d");

    try {
        $stmt = $pdo->prepare("
            INSERT INTO cars_feedback 
            (ID_CAR, move_time, clean, driver_clean, driver_tactic, move_safety, move_comfort, smell_cabin, volume_music, temperature_cabin, comment, comment_date, telephone)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
        ");
        $stmt->execute([
            $car_id,
            $data['move_time'],
            $data['clean'],
            $data['driver_clean'],
            $data['driver_tactic'],
            $data['move_safety'],
            $data['move_comfort'],
            $data['smell_cabin'],
            $data['volume_music'],
            $data['temperature_cabin'],
            $comment,
            $date,
            $telephone
        ]);

        debug_log("Отзыв успешно сохранён для car_id=$car_id");
        $success = true;
    } catch (Exception $e) {
        debug_log("Ошибка вставки: " . $e->getMessage());
        echo "<div style='color:red'>Ошибка сохранения: " . htmlspecialchars($e->getMessage()) . "</div>";
    }
}
?>
