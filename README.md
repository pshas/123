Понял 👍
Для начала давай сделаем простой вывод списка заявок из инфоблока `lab_orders` (как у тебя в коде). В таблице будут колонки, которые ты перечислил, и последняя колонка **«Действие»** с кнопкой (ссылкой) на редактирование только если статус заявки `new`.

Пример кода (файл `orders_list.php`):

```php
<?php
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php');
use Bitrix\Main\Loader;
global $USER;

Loader::includeModule('iblock');

$iblockCode = 'lab_orders';
$rsI = CIBlock::GetList([], ['CODE'=>$iblockCode,'ACTIVE'=>'Y']);
if (!$ib = $rsI->Fetch()) {
    ShowError("Инфоблок заявок не найден");
    require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
    exit;
}
$ordersIblockId = (int)$ib['ID'];

// Выборка элементов
$arSelect = [
    "ID", "NAME", "DATE_CREATE", "CREATED_BY", 
    "PROPERTY_WORK_DEPARTMENT", "PROPERTY_DETAIL_NAME", "PROPERTY_DETAIL_NUMBER",
    "PROPERTY_DATE_PRODUCTION", "PROPERTY_BATCH_NUMBER", "PROPERTY_NONCONFORMANCE",
    "PROPERTY_STATUS"
];
$arFilter = ["IBLOCK_ID" => $ordersIblockId, "ACTIVE" => "Y"];
$res = CIBlockElement::GetList(["DATE_CREATE" => "DESC"], $arFilter, false, false, $arSelect);

// Таблица
echo '<table border="1" cellpadding="5" cellspacing="0" style="margin:20px auto; border-collapse:collapse;">';
echo '<tr>
        <th>Номер</th>
        <th>Создан</th>
        <th>Инициатор</th>
        <th>Цех/Отдел</th>
        <th>Наименование детали</th>
        <th>Номер детали</th>
        <th>Дата изготовления</th>
        <th>Номер партии</th>
        <th>Несоответствие</th>
        <th>Статус</th>
        <th>Действие</th>
      </tr>';

while ($arItem = $res->GetNext()) {
    // Получаем имя инициатора
    $initiator = '';
    if ($arItem['CREATED_BY']) {
        $rsUser = CUser::GetByID($arItem['CREATED_BY']);
        if ($arUser = $rsUser->Fetch()) {
            $initiator = $arUser['LAST_NAME'].' '.$arUser['NAME'];
        }
    }

    $status = $arItem['PROPERTY_STATUS_VALUE'];
    $actionBtn = '';
    if ($status === 'new') {
        $actionBtn = '<a href="/lab_orders/edit.php?id='.$arItem['ID'].'">Редактировать</a>';
    }

    echo '<tr>
            <td>'.$arItem['ID'].'</td>
            <td>'.$arItem['DATE_CREATE'].'</td>
            <td>'.$initiator.'</td>
            <td>'.$arItem['PROPERTY_WORK_DEPARTMENT_VALUE'].'</td>
            <td>'.$arItem['PROPERTY_DETAIL_NAME_VALUE'].'</td>
            <td>'.$arItem['PROPERTY_DETAIL_NUMBER_VALUE'].'</td>
            <td>'.$arItem['PROPERTY_DATE_PRODUCTION_VALUE'].'</td>
            <td>'.$arItem['PROPERTY_BATCH_NUMBER_VALUE'].'</td>
            <td>'.$arItem['PROPERTY_NONCONFORMANCE_VALUE'].'</td>
            <td>'.$status.'</td>
            <td>'.$actionBtn.'</td>
          </tr>';
}

echo '</table>';

require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
```

---

🔹 Что сделано:

* Вытаскиваем все заявки из инфоблока `lab_orders`.
* Выводим таблицу с колонками, которые ты перечислил.
* В колонке «Действие» выводим ссылку **Редактировать** только если `status = new`.
* Ссылка ведёт на `/lab_orders/edit.php?id=XXX`.

---

Хочешь, я сразу подготовлю и пример файла `edit.php`, где можно будет редактировать сохранённую заявку?
