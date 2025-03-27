<?
require("../header.php");

// Получаем ID заявки
$application_id = intval($_GET['id']);

// Подключаемся к БД
$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';
$pdo = new PDO($dsn, $username, $password);

// Получаем данные заявки
$stmt = $pdo->prepare("SELECT * FROM user_applications WHERE id = ?");
$stmt->execute([$application_id]);
$application = $stmt->fetch(PDO::FETCH_ASSOC);

if (!$application) {
    die("Заявка не найдена");
}
?>

<!-- Bootstrap CSS -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<div class="container mt-4">
    <h2>Форма отчета по заявке №<?= htmlspecialchars($application['id']) ?></h2>
    
    <form method="POST" action="save_report.php">
        <input type="hidden" name="application_id" value="<?= $application['id'] ?>">
        
        <div class="form-group">
            <label>Объект исследования</label>
            <input type="text" class="form-control" value="<?= htmlspecialchars($application['NAME_IZD']) ?>" readonly>
        </div>
        
        <div class="form-group">
            <label>Инициатор</label>
            <input type="text" class="form-control" value="<?= htmlspecialchars($application['full_name']) ?>" readonly>
        </div>
        
        <div class="form-group">
            <label>Дата создания</label>
            <input type="text" class="form-control" value="<?= htmlspecialchars($application['created_at']) ?>" readonly>
        </div>
        
        <div class="form-group">
            <label>Несоответствие</label>
            <input type="text" class="form-control" value="<?= htmlspecialchars($application['task']) ?>" readonly>
        </div>
        
        <div class="form-group">
            <label>Цель испытания</label>
            <textarea class="form-control" readonly><?= htmlspecialchars($application['note']) ?></textarea>
        </div>
        
        <div class="form-group">
            <label for="results">Результаты исследования</label>
            <textarea class="form-control" name="results" id="results" rows="3" required></textarea>
        </div>
        
        <div class="form-group">
            <label for="description">Описание</label>
            <textarea class="form-control" name="description" id="description" rows="3" required></textarea>
        </div>
        
        <div class="form-group">
            <label for="conclusion">Выводы</label>
            <textarea class="form-control" name="conclusion" id="conclusion" rows="3" required></textarea>
        </div>
        
        <button type="submit" class="btn btn-primary">Сохранить отчет</button>
    </form>
</div>
