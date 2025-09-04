Отлично, вижу твою структуру таблицы **cars\_feedback** 👌

У тебя такие поля:

* **ID\_FEEDBACK** (PK)
* **ID\_CAR** (FK)
* **move\_time, clean, driver\_clean, driver\_tactic, move\_safety, move\_comfort, smell\_cabin, volume\_music, temperature\_cabin** → все `TINYINT(1)` (оценки 1–5)
* **comment** (текст отзыва)
* **comment\_date** (DATE)
* **telephone** (номер телефона)

Тогда адаптируем код.

---

## 📌 survey.php (страница опроса)

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=qr_opros;charset=utf8", "root", "");

// Получаем список машин
$cars = $pdo->query("SELECT * FROM cars")->fetchAll(PDO::FETCH_ASSOC);

// Обработка формы
if ($_SERVER["REQUEST_METHOD"] === "POST") {
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

    $success = true;
}
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Опрос по автомобилям</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
<div class="container py-4">
    <div class="row justify-content-center">
        <div class="col-lg-7 col-md-9 col-sm-12">
            <div class="card shadow-sm">
                <div class="card-body">
                    <h3 class="card-title text-center mb-4">Опрос по автомобилям</h3>

                    <?php if (!empty($success)): ?>
                        <div class="alert alert-success">Спасибо за отзыв!</div>
                    <?php endif; ?>

                    <form method="post" class="row g-3">
                        <div class="col-12">
                            <label class="form-label">Выберите автомобиль</label>
                            <select name="car_id" class="form-select" required>
                                <?php foreach ($cars as $car): ?>
                                    <option value="<?= $car['id'] ?>">
                                        <?= htmlspecialchars($car['name']) ?> (<?= htmlspecialchars($car['model']) ?>)
                                    </option>
                                <?php endforeach; ?>
                            </select>
                        </div>

                        <?php 
                        $criteria = [
                            "move_time" => "Время поездки",
                            "clean" => "Чистота салона",
                            "driver_clean" => "Опрятность водителя",
                            "driver_tactic" => "Манера вождения",
                            "move_safety" => "Безопасность движения",
                            "move_comfort" => "Комфорт поездки",
                            "smell_cabin" => "Запах в салоне",
                            "volume_music" => "Громкость музыки",
                            "temperature_cabin" => "Температура в салоне"
                        ];
                        foreach ($criteria as $name => $label): ?>
                            <div class="col-12">
                                <label class="form-label"><?= $label ?></label>
                                <input type="range" class="form-range" name="<?= $name ?>" min="1" max="5" required>
                            </div>
                        <?php endforeach; ?>

                        <div class="col-12">
                            <label class="form-label">Комментарий</label>
                            <textarea name="comment" class="form-control" rows="3" placeholder="Ваш отзыв..."></textarea>
                        </div>

                        <div class="col-12">
                            <label class="form-label">Телефон (необязательно)</label>
                            <input type="text" name="telephone" class="form-control" placeholder="+7...">
                        </div>

                        <div class="col-12">
                            <button type="submit" class="btn btn-primary w-100">Отправить отзыв</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```

---

## 📌 results.php (результаты)

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=qr_opros;charset=utf8", "root", "");

$cars = $pdo->query("SELECT * FROM cars")->fetchAll(PDO::FETCH_ASSOC);

$car_id = $_GET['car_id'] ?? null;
$feedback = [];
$averages = [];

if ($car_id) {
    $stmt = $pdo->prepare("SELECT * FROM cars_feedback WHERE ID_CAR = ?");
    $stmt->execute([$car_id]);
    $feedback = $stmt->fetchAll(PDO::FETCH_ASSOC);

    $stmt = $pdo->prepare("
        SELECT 
            AVG(move_time) as avg_move_time,
            AVG(clean) as avg_clean,
            AVG(driver_clean) as avg_driver_clean,
            AVG(driver_tactic) as avg_driver_tactic,
            AVG(move_safety) as avg_move_safety,
            AVG(move_comfort) as avg_move_comfort,
            AVG(smell_cabin) as avg_smell_cabin,
            AVG(volume_music) as avg_volume_music,
            AVG(temperature_cabin) as avg_temperature_cabin
        FROM cars_feedback
        WHERE ID_CAR = ?
    ");
    $stmt->execute([$car_id]);
    $averages = $stmt->fetch(PDO::FETCH_ASSOC);
}
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Результаты опроса</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="bg-light">
<div class="container py-4">
    <div class="card shadow-sm">
        <div class="card-body">
            <h3 class="card-title text-center mb-4">Результаты опроса</h3>

            <form method="get" class="mb-3">
                <label class="form-label">Выберите автомобиль</label>
                <select name="car_id" class="form-select" onchange="this.form.submit()">
                    <option value="">-- выберите --</option>
                    <?php foreach ($cars as $car): ?>
                        <option value="<?= $car['id'] ?>" <?= ($car['id'] == $car_id ? "selected" : "") ?>>
                            <?= htmlspecialchars($car['name']) ?> (<?= htmlspecialchars($car['model']) ?>)
                        </option>
                    <?php endforeach; ?>
                </select>
            </form>

            <?php if ($car_id): ?>
                <h5 class="mb-3">Отзывы</h5>
                <div class="table-responsive">
                    <table class="table table-striped table-hover">
                        <thead class="table-dark">
                        <tr>
                            <th>Время</th>
                            <th>Чистота</th>
                            <th>Водитель</th>
                            <th>Манера</th>
                            <th>Безопасность</th>
                            <th>Комфорт</th>
                            <th>Запах</th>
                            <th>Музыка</th>
                            <th>Температура</th>
                            <th>Комментарий</th>
                            <th>Дата</th>
                            <th>Телефон</th>
                        </tr>
                        </thead>
                        <tbody>
                        <?php foreach ($feedback as $row): ?>
                            <tr>
                                <td><?= $row['move_time'] ?></td>
                                <td><?= $row['clean'] ?></td>
                                <td><?= $row['driver_clean'] ?></td>
                                <td><?= $row['driver_tactic'] ?></td>
                                <td><?= $row['move_safety'] ?></td>
                                <td><?= $row['move_comfort'] ?></td>
                                <td><?= $row['smell_cabin'] ?></td>
                                <td><?= $row['volume_music'] ?></td>
                                <td><?= $row['temperature_cabin'] ?></td>
                                <td><?= htmlspecialchars($row['comment']) ?></td>
                                <td><?= $row['comment_date'] ?></td>
                                <td><?= htmlspecialchars($row['telephone']) ?></td>
                            </tr>
                        <?php endforeach; ?>
                        </tbody>
                    </table>
                </div>

                <h5 class="mt-4">Средние оценки</h5>
                <canvas id="chart" height="200"></canvas>
                <script>
                    const ctx = document.getElementById('chart');
                    new Chart(ctx, {
                        type: 'radar',
                        data: {
                            labels: [
                                'Время', 'Чистота', 'Опрятность водителя', 'Манера вождения',
                                'Безопасность', 'Комфорт', 'Запах', 'Музыка', 'Температура'
                            ],
                            datasets: [{
                                label: 'Средняя оценка',
                                data: [
                                    <?= round($averages['avg_move_time'],2) ?>,
                                    <?= round($averages['avg_clean'],2) ?>,
                                    <?= round($averages['avg_driver_clean'],2) ?>,
                                    <?= round($averages['avg_driver_tactic'],2) ?>,
                                    <?= round($averages['avg_move_safety'],2) ?>,
                                    <?= round($averages['avg_move_comfort'],2) ?>,
                                    <?= round($averages['avg_smell_cabin'],2) ?>,
                                    <?= round($averages['avg_volume_music'],2) ?>,
                                    <?= round($averages['avg_temperature_cabin'],2) ?>
                                ],
                                backgroundColor: 'rgba(13,110,253,0.2)',
                                borderColor: '#0d6efd',
                                pointBackgroundColor: '#0d6efd'
                            }]
                        },
                        options: {
                            responsive: true,
                            scales: {
                                r: {
                                    min: 0,
                                    max: 5,
                                    ticks: { stepSize: 1 }
                                }
                            }
                        }
                    });
                </script>
            <?php endif; ?>
        </div>
    </div>
</div>
</body>
</html>
```

---

Теперь:

* В **survey.php** добавлены все поля из таблицы.
* В **results.php** выводятся отзывы в таблице и **радарная диаграмма** (наглядно для 9 критериев).

---

Хочешь, чтобы в форме **оценка отображалась цифрой рядом со слайдером** (например: "3/5") при перемещении?
