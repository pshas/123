Отлично ✅
Тогда давай просто **переделаем SQL-запросы с MySQL на PostgreSQL**, без Bitrix-ORM и без лишней абстракции.
Я покажу именно те изменения, которые нужно внести, чтобы код на PHP/PDO, работавший с MySQL, стал корректно работать с PostgreSQL.

---

## 🔧 1. Подключение (PDO)

### Было (MySQL)

```php
$pdo = new PDO(
    "mysql:host=localhost;dbname=mydb;charset=utf8",
    "user",
    "pass",
    [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]
);
```

### Стало (PostgreSQL)

```php
$pdo = new PDO(
    "pgsql:host=localhost;port=5432;dbname=mydb",
    "user",
    "pass",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
    ]
);
$pdo->exec("SET client_encoding TO 'UTF8'");
```

> ⚠️ В `pgsql` нельзя указывать `charset` в DSN — только через `SET client_encoding`.

---

## 📄 2. `add_block.php`

### Было (MySQL)

```php
$stmt = $pdo->prepare("INSERT INTO cars (name_car, type_car, nom_car) VALUES (?, ?, ?)");
$stmt->execute([$nameCar, $typeCar, $nomCar]);
$carId = $pdo->lastInsertId();
```

### Стало (PostgreSQL)

```php
$stmt = $pdo->prepare("
    INSERT INTO cars (name_car, type_car, nom_car)
    VALUES (:name, :type, :nom)
    RETURNING id_car
");
$stmt->execute([
    ':name' => $nameCar,
    ':type' => $typeCar,
    ':nom'  => $nomCar
]);
$carId = (int)$stmt->fetchColumn();
```

> PostgreSQL не поддерживает `lastInsertId()` без указания sequence.
> Используем `RETURNING id_car`, чтобы сразу получить ID.

---

## 📄 3. `prepare.php`

### Было (MySQL)

```php
$stmt = $pdo->prepare("UPDATE cars SET poll_created = 1 WHERE id_car = ?");
$stmt->execute([$carId]);
```

### Стало (PostgreSQL)

```php
$stmt = $pdo->prepare("UPDATE cars SET poll_created = TRUE WHERE id_car = :id");
$stmt->execute([':id' => $carId]);
```

> Булевые значения в PostgreSQL — `TRUE/FALSE`, не `1/0`.

---

## 📄 4. `load_blocks.php`

### Было (MySQL)

```php
$stmt = $pdo->query("SELECT id_car, name_car, type_car, nom_car FROM cars ORDER BY id_car DESC");
$rows = $stmt->fetchAll();
```

### Стало (PostgreSQL)

```php
$stmt = $pdo->query("SELECT id_car, name_car, type_car, nom_car FROM cars ORDER BY id_car DESC");
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

> Тут изменений почти нет — SQL-синтаксис совпадает.

---

## 📄 5. `remove_block.php`

### Было (MySQL)

```php
$stmt = $pdo->prepare("DELETE FROM cars WHERE id_car = ?");
$stmt->execute([$id]);
```

### Стало (PostgreSQL)

```php
$stmt = $pdo->prepare("DELETE FROM cars WHERE id_car = :id");
$stmt->execute([':id' => $id]);
```

> Формально одинаково, просто для единообразия можно использовать именованные параметры.

---

## 🧱 6. Частые отличия MySQL → PostgreSQL

| Особенность        | MySQL                | PostgreSQL                               |   |    |
| ------------------ | -------------------- | ---------------------------------------- | - | -- |
| Булевый тип        | `TINYINT(1)` с `0/1` | `BOOLEAN` с `TRUE/FALSE`                 |   |    |
| Автоинкремент      | `AUTO_INCREMENT`     | `SERIAL` или `GENERATED ... AS IDENTITY` |   |    |
| NOW()              | ✅                    | ✅                                        |   |    |
| LIMIT              | `LIMIT offset,count` | `LIMIT count OFFSET offset`              |   |    |
| Конкатенация строк | `CONCAT(a,b)`        | `a                                       |   | b` |
| Escape ``` кавычки | ``` (бэктик)         | `"` (двойные кавычки)                    |   |    |

---

## ✅ Пример полностью работающего `add_block.php` под PostgreSQL

```php
<?php
$pdo = new PDO(
    "pgsql:host=localhost;port=5432;dbname=mydb",
    "user",
    "pass",
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false
    ]
);
$pdo->exec("SET client_encoding TO 'UTF8'");

$nameCar = $_POST['name_car'] ?? '';
$typeCar = $_POST['type_car'] ?? '';
$nomCar  = $_POST['nom_car'] ?? '';

$stmt = $pdo->prepare("
    INSERT INTO cars (name_car, type_car, nom_car)
    VALUES (:name, :type, :nom)
    RETURNING id_car
");
$stmt->execute([':name' => $nameCar, ':type' => $typeCar, ':nom' => $nomCar]);
$id = (int)$stmt->fetchColumn();

echo json_encode(['success' => true, 'id' => $id]);
```

---

Хочешь, я пройдусь по всем твоим файлам из репозитория `github.com/pshas/2134` и выдам готовую diff-версию (`.patch`) с переводом всех запросов на PostgreSQL (чтобы можно было просто применить)?
