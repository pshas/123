Отлично 👍 сделаем так, чтобы рядом со слайдером отображалась цифра в реальном времени (например: `3 / 5`).

Это можно сделать на **JS** → при изменении `input[type=range]` обновляем `span`.

---

## 📌 Обновлённый фрагмент в **survey.php**

Заменим часть с критериями:

```php
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
        <div class="d-flex align-items-center">
            <input type="range" class="form-range flex-grow-1 me-3" 
                   name="<?= $name ?>" min="1" max="5" value="3" 
                   oninput="document.getElementById('val_<?= $name ?>').innerText = this.value">
            <span id="val_<?= $name ?>" class="fw-bold">3</span> / 5
        </div>
    </div>
<?php endforeach; ?>
```

---

## 📌 Полный **survey.php** (с цифрами у слайдеров)

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
                                <div class="d-flex align-items-center">
                                    <input type="range" class="form-range flex-grow-1 me-3" 
                                           name="<?= $name ?>" min="1" max="5" value="3" 
                                           oninput="document.getElementById('val_<?= $name ?>').innerText = this.value">
                                    <span id="val_<?= $name ?>" class="fw-bold">3</span> / 5
                                </div>
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

✅ Теперь при движении слайдера рядом сразу видно число (`1–5`).
📱 На мобильном тоже работает отлично.

---

Хочешь, я ещё добавлю **цветовую шкалу** (например: красный при `1`, жёлтый при `3`, зелёный при `5`) для слайдеров?
