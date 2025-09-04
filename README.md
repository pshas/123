Отлично 👍 Тогда давай сразу завернём это в **Bootstrap 5**, чтобы всё было адаптивно (на телефоне элементы будут красиво перестраиваться).

Я перепишу обе страницы (**survey.php** и **results.php**) с современным дизайном.

---

## 📌 survey.php (страница опроса)

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=test;charset=utf8", "root", "");

// Получаем список машин
$cars = $pdo->query("SELECT * FROM cars")->fetchAll(PDO::FETCH_ASSOC);

// Обработка формы
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $car_id = (int)$_POST['car_id'];
    $comfort = (int)$_POST['comfort'];
    $design = (int)$_POST['design'];
    $price = (int)$_POST['price'];
    $comment = trim($_POST['comment']);

    $stmt = $pdo->prepare("
        INSERT INTO cars_feedback (car_id, rating_comfort, rating_design, rating_price, comment)
        VALUES (?, ?, ?, ?, ?)
    ");
    $stmt->execute([$car_id, $comfort, $design, $price, $comment]);

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
        <div class="col-lg-6 col-md-8 col-sm-12">
            <div class="card shadow-sm">
                <div class="card-body">
                    <h3 class="card-title text-center mb-4">Опрос по автомобилям</h3>

                    <?php if (!empty($success)): ?>
                        <div class="alert alert-success">Спасибо за отзыв!</div>
                    <?php endif; ?>

                    <form method="post">
                        <div class="mb-3">
                            <label class="form-label">Выберите автомобиль</label>
                            <select name="car_id" class="form-select" required>
                                <?php foreach ($cars as $car): ?>
                                    <option value="<?= $car['id'] ?>">
                                        <?= htmlspecialchars($car['name']) ?> (<?= htmlspecialchars($car['model']) ?>)
                                    </option>
                                <?php endforeach; ?>
                            </select>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Комфорт</label>
                            <input type="range" class="form-range" name="comfort" min="1" max="5" required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Дизайн</label>
                            <input type="range" class="form-range" name="design" min="1" max="5" required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Цена/качество</label>
                            <input type="range" class="form-range" name="price" min="1" max="5" required>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">Комментарий</label>
                            <textarea name="comment" class="form-control" rows="3" placeholder="Ваш отзыв..."></textarea>
                        </div>

                        <button type="submit" class="btn btn-primary w-100">Отправить отзыв</button>
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

## 📌 results.php (страница результатов)

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=test;charset=utf8", "root", "");

// Получаем список машин
$cars = $pdo->query("SELECT * FROM cars")->fetchAll(PDO::FETCH_ASSOC);

$car_id = $_GET['car_id'] ?? null;
$feedback = [];
$averages = [];

if ($car_id) {
    $stmt = $pdo->prepare("SELECT * FROM cars_feedback WHERE car_id = ?");
    $stmt->execute([$car_id]);
    $feedback = $stmt->fetchAll(PDO::FETCH_ASSOC);

    $stmt = $pdo->prepare("
        SELECT 
            AVG(rating_comfort) as avg_comfort,
            AVG(rating_design) as avg_design,
            AVG(rating_price) as avg_price
        FROM cars_feedback
        WHERE car_id = ?
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
                            <th>Комфорт</th>
                            <th>Дизайн</th>
                            <th>Цена/качество</th>
                            <th>Комментарий</th>
                            <th>Дата</th>
                        </tr>
                        </thead>
                        <tbody>
                        <?php foreach ($feedback as $row): ?>
                            <tr>
                                <td><?= $row['rating_comfort'] ?></td>
                                <td><?= $row['rating_design'] ?></td>
                                <td><?= $row['rating_price'] ?></td>
                                <td><?= htmlspecialchars($row['comment']) ?></td>
                                <td><?= $row['created_at'] ?></td>
                            </tr>
                        <?php endforeach; ?>
                        </tbody>
                    </table>
                </div>

                <h5 class="mt-4">Средние оценки</h5>
                <canvas id="chart" height="150"></canvas>
                <script>
                    const ctx = document.getElementById('chart');
                    new Chart(ctx, {
                        type: 'bar',
                        data: {
                            labels: ['Комфорт', 'Дизайн', 'Цена/качество'],
                            datasets: [{
                                label: 'Средняя оценка',
                                data: [
                                    <?= round($averages['avg_comfort'],2) ?>,
                                    <?= round($averages['avg_design'],2) ?>,
                                    <?= round($averages['avg_price'],2) ?>
                                ],
                                backgroundColor: ['#0d6efd','#198754','#ffc107']
                            }]
                        },
                        options: {
                            responsive: true,
                            scales: { y: { min: 0, max: 5, ticks: { stepSize: 1 } } }
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

### 📱 Что улучшено для мобильных:

* Используется **Bootstrap grid** → элементы красиво центрируются.
* Поля ввода — **range sliders** (удобно на телефоне).
* Таблица с отзывами обернута в `table-responsive` → скролл на маленьких экранах.
* Кнопки и поля формы занимают **100% ширины на мобилке**.

---

Хочешь, я ещё добавлю **ссылку-переключатель** (например "Оставить отзыв / Смотреть результаты"), чтобы можно было легко переходить между `survey.php` и `results.php` без ввода URL?
