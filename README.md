Отлично 👍
Вот пример, как можно **переписать ваш код под PostgreSQL с использованием встроенного ORM Bitrix**, без ручного PDO-подключения.

---

## 📁 Файл `add_block.php` (добавление записи в таблицу `cars`)

```php
<?php
use Bitrix\Main\Loader;
use Bitrix\Main\Application;
use Bitrix\Main\Entity;

// Подключаем ядро Bitrix
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
Loader::includeModule("main");

// === ОПИСАНИЕ ORM СУЩНОСТИ ===
class CarTable extends Entity\DataManager
{
    public static function getTableName()
    {
        return 'cars';
    }

    public static function getMap()
    {
        return [
            new Entity\IntegerField('ID_CAR', [
                'primary' => true,
                'autocomplete' => true
            ]),
            new Entity\StringField('NAME_CAR'),
            new Entity\StringField('TYPE_CAR'),
            new Entity\StringField('NOM_CAR'),
        ];
    }
}

// === ДОБАВЛЕНИЕ ЗАПИСИ ===
$request = Application::getInstance()->getContext()->getRequest();

$nameCar = $request->getPost("name_car");
$typeCar = $request->getPost("type_car");
$nomCar  = $request->getPost("nom_car");

if ($nameCar && $typeCar && $nomCar)
{
    $result = CarTable::add([
        'NAME_CAR' => $nameCar,
        'TYPE_CAR' => $typeCar,
        'NOM_CAR'  => $nomCar
    ]);

    if ($result->isSuccess())
    {
        $newId = $result->getId();
        echo json_encode(['success' => true, 'id' => $newId]);
    }
    else
    {
        echo json_encode(['success' => false, 'errors' => $result->getErrorMessages()]);
    }
}
else
{
    echo json_encode(['success' => false, 'error' => 'Missing fields']);
}
```

---

## 📁 Файл `prepare.php` (обновление флага `POLL_CREATED`)

```php
<?php
use Bitrix\Main\Application;
use Bitrix\Main\Loader;

require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
Loader::includeModule("main");

$connection = Application::getConnection();
$sqlHelper  = $connection->getSqlHelper();

$carId = (int)$_POST['car_id'];

if ($carId > 0)
{
    $connection->queryExecute("
        UPDATE cars
        SET poll_created = TRUE
        WHERE id_car = {$carId}
    ");
    echo json_encode(['success' => true]);
}
else
{
    echo json_encode(['success' => false, 'error' => 'Invalid car_id']);
}
```

---

## 📁 Файл `load_blocks.php` (чтение списка машин)

```php
<?php
use Bitrix\Main\Loader;
use Bitrix\Main\Application;

require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
Loader::includeModule("main");

$connection = Application::getConnection();

$result = $connection->query("
    SELECT id_car, name_car, type_car, nom_car
    FROM cars
    ORDER BY id_car DESC
");

$data = [];
while ($row = $result->fetch())
{
    $data[] = $row;
}

echo json_encode($data);
```

---

## 📁 Файл `remove_block.php` (удаление записи)

```php
<?php
use Bitrix\Main\Application;
use Bitrix\Main\Loader;

require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
Loader::includeModule("main");

$connection = Application::getConnection();
$carId = (int)$_POST['id_car'];

if ($carId > 0)
{
    $connection->queryExecute("DELETE FROM cars WHERE id_car = {$carId}");
    echo json_encode(['success' => true]);
}
else
{
    echo json_encode(['success' => false, 'error' => 'Invalid car_id']);
}
```

---

## ⚙️ Что нужно ещё сделать

1. **В `/bitrix/.settings.php`**:

   ```php
   'connections' => [
       'value' => [
           'default' => [
               'className' => '\\Bitrix\\Main\\DB\\PgsqlConnection',
               'host' => 'localhost',
               'database' => 'mydb',
               'login' => 'myuser',
               'password' => 'mypassword',
               'options' => 2,
           ],
       ],
   ],
   ```

2. **В базе PostgreSQL**:
   Убедитесь, что таблица `cars` создана правильно:

   ```sql
   CREATE TABLE cars (
       id_car SERIAL PRIMARY KEY,
       name_car VARCHAR(255),
       type_car VARCHAR(255),
       nom_car  VARCHAR(255),
       poll_created BOOLEAN DEFAULT FALSE
   );
   ```

---

Если хочешь, я могу подготовить diff-патч (`.patch` файл), который можно применить напрямую к репозиторию на GitHub — всё будет уже готово к PostgreSQL и Bitrix ORM.
Сделать такой патч?
