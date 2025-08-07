Ниже пример скрипта, который создаст в вашем Битриксе через API D7/CIBlock:

```php
<?php
use Bitrix\Main\Loader;
use Bitrix\Iblock\IblockTable;

require_once($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
Loader::includeModule('iblock');

// 1) Создаём (если нужно) тип инфоблока «lab»
if (!CIBlockType::GetByID("lab")->Fetch()) {
    CIBlockType::Add([
        "ID" => "lab",
        "SECTIONS" => "Y",
        "IN_RSS" => "N",
        "SORT" => 500,
        "LANG" => [
            "ru" => [
                "NAME" => "Laboratory",
                "SECTION_NAME" => "Разделы лаборатории",
                "ELEMENT_NAME" => "Элементы лаборатории"
            ]
        ]
    ]);
}

// 2) Проверяем, нет ли уже инфоблока с кодом lab_orders
$exists = IblockTable::getList([
    'select' => ['ID'],
    'filter' => ['=CODE' => 'lab_orders']
])->fetch();

if ($exists) {
    $iblockId = $exists['ID'];
    echo "Инфоблок lab_orders уже существует (ID={$iblockId})\n";
} else {
    // 3) Добавляем инфоблок «Заявки лаборатории»
    $ib = new CIBlock;
    $id = $ib->Add([
        'ACTIVE' => 'Y',
        'NAME' => 'Заявки лаборатории',
        'CODE' => 'lab_orders',
        'IBLOCK_TYPE_ID' => 'lab',
        'SITE_ID' => ['s1'],       // замените на код вашего сайта
        'SORT' => 500,
        'GROUP_ID' => [           // права: админы — полный доступ, все — создание
            1 => 'X',
            2 => 'W',
        ],
    ]);
    if (!$id) {
        die("Ошибка создания инфоблока: ".$ib->LAST_ERROR);
    }
    $iblockId = $id;
    echo "Создан инфоблок lab_orders (ID={$iblockId})\n";

    // 4) Добавляем свойства

    $prop = new CIBlockProperty;

    // USER_ID
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Пользователь',
        'CODE' => 'USER_ID',
        'PROPERTY_TYPE' => 'N',  // числовое
        'SORT' => 100,
    ]);
    echo "Добавлено свойство USER_ID\n";

    // BLOCK_REF
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Подразделение (SECTION_ID)',
        'CODE' => 'BLOCK_REF',
        'PROPERTY_TYPE' => 'N',
        'SORT' => 200,
    ]);
    echo "Добавлено свойство BLOCK_REF\n";

    // FULL_NAME
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'ФИО',
        'CODE' => 'FULL_NAME',
        'PROPERTY_TYPE' => 'S',  // строковое
        'SORT' => 300,
    ]);
    echo "Добавлено свойство FULL_NAME\n";

    // PHONE
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Телефон',
        'CODE' => 'PHONE',
        'PROPERTY_TYPE' => 'S',
        'SORT' => 400,
    ]);
    echo "Добавлено свойство PHONE\n";

    // POSITION
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Должность',
        'CODE' => 'POSITION',
        'PROPERTY_TYPE' => 'S',
        'SORT' => 500,
    ]);
    echo "Добавлено свойство POSITION\n";

    // TASK
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Задача',
        'CODE' => 'TASK',
        'PROPERTY_TYPE' => 'S',
        'MULTIPLE' => 'Y',
        'SORT' => 600,
    ]);
    echo "Добавлено свойство TASK\n";

    // NOTE
    $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Примечание',
        'CODE' => 'NOTE',
        'PROPERTY_TYPE' => 'S',
        'MULTIPLE' => 'Y',
        'SORT' => 700,
    ]);
    echo "Добавлено свойство NOTE\n";

    // STATUS - список
    $statusPropId = $prop->Add([
        'ACTIVE' => 'Y',
        'IBLOCK_ID' => $iblockId,
        'NAME' => 'Статус',
        'CODE' => 'STATUS',
        'PROPERTY_TYPE' => 'L',  // список
        'SORT' => 800,
    ]);
    if ($statusPropId) {
        $enum = new CIBlockPropertyEnum;
        $enum->Add(['PROPERTY_ID' => $statusPropId, 'VALUE' => 'NEW',      'DEF' => 'Y']);
        $enum->Add(['PROPERTY_ID' => $statusPropId, 'VALUE' => 'IN_PROGRESS']);
        $enum->Add(['PROPERTY_ID' => $statusPropId, 'VALUE' => 'COMPLETED']);
        echo "Добавлено свойство STATUS с вариантами\n";
    } else {
        echo "Ошибка создания свойства STATUS\n";
    }
}

echo "Готово.\n";
```

**Как пользоваться:**

1. Сохраните этот код в файл, например `/local/tools/create_lab_orders_iblock.php`.
2. Запустите в браузере:

   ```
   https://ваш-домен/local/tools/create_lab_orders_iblock.php
   ```
3. Проверьте результат в админке — должен появиться инфоблок **lab\_orders** с перечисленными свойствами.
4. Удалите или закомментируйте скрипт после успешного выполнения, чтобы не создавать дубли.
