<?php
// /local/glab/application/orders.php

// 1) Подключаем ядро и «шапку» сайта
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php');
use Bitrix\Main\Loader;
global $USER;

// 2) Проверяем авторизацию
if (!$USER->IsAuthorized()) {
    LocalRedirect('/login/');
}

// 3) Подключаем iblock и находим инфоблок
Loader::includeModule('iblock');
$iblockCode = 'lab_orders';
$rsI = CIBlock::GetList([], ['CODE'=>$iblockCode,'ACTIVE'=>'Y']);
if (!$ib = $rsI->Fetch()) {
    ShowError("Инфоблок заявок не найден");
    require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
    exit;
}
$iblockId = (int)$ib['ID'];

// 4) Определяем, лаборант ли пользователь
//    замените 8 на ID вашей группы лаборантов
$isLab = in_array(8, $USER->GetUserGroupArray(), true);

// 5) Формируем фильтр
$arFilter = [
    'IBLOCK_ID' => $iblockId,
    'ACTIVE'    => 'Y',
];
if (!$isLab) {
    // обычный пользователь — только свои
    $arFilter['PROPERTY_USER_ID'] = $USER->GetID();
}

// 6) Получаем список заявок
$rs = CIBlockElement::GetList(
    ['DATE_CREATE'=>'DESC'],
    $arFilter,
    false,
    false,
    [
      'ID','NAME','DATE_CREATE',
      'PROPERTY_FULL_NAME',
      'PROPERTY_PHONE',
      'PROPERTY_POSITION',
      'PROPERTY_TASK',
      'PROPERTY_NOTE',
      'PROPERTY_STATUS',
      'IBLOCK_SECTION_ID',
    ]
);

// 7) Выводим заголовок и таблицу
$APPLICATION->SetTitle('Мои заявки');
?>
<h1><?= $APPLICATION->GetTitle() ?></h1>

<table class="orders-table" border="1" cellpadding="5" cellspacing="0">
  <thead>
    <tr>
      <th>ID</th>
      <th>Дата</th>
      <th>Блок (подраздел.)</th>
      <th>ФИО</th>
      <th>Телефон</th>
      <th>Должность</th>
      <th>Задача</th>
      <th>Примечание</th>
      <th>Статус</th>
      <?php if ($isLab): ?>
      <th>Пользователь</th>
      <?php endif; ?>
      <th>Действия</th>
    </tr>
  </thead>
  <tbody>
  <?php while ($el = $rs->Fetch()):
      // получение названия подразделения
      $sectionName = '';
      if ($el['IBLOCK_SECTION_ID']) {
          $sec = CIBlockSection::GetByID($el['IBLOCK_SECTION_ID'])->Fetch();
          $sectionName = htmlspecialcharsbx($sec['NAME']);
      }
  ?>
    <tr>
      <td><?= $el['ID'] ?></td>
      <td><?= $el['DATE_CREATE'] ?></td>
      <td><?= $sectionName ?></td>
      <td><?= htmlspecialcharsbx($el['PROPERTY_FULL_NAME_VALUE']) ?></td>
      <td><?= htmlspecialcharsbx($el['PROPERTY_PHONE_VALUE']) ?></td>
      <td><?= htmlspecialcharsbx($el['PROPERTY_POSITION_VALUE']) ?></td>
      <td><?= nl2br(htmlspecialcharsbx($el['PROPERTY_TASK_VALUE'])) ?></td>
      <td><?= nl2br(htmlspecialcharsbx($el['PROPERTY_NOTE_VALUE'])) ?></td>
      <td><?= htmlspecialcharsbx($el['PROPERTY_STATUS_VALUE']) ?></td>
      <?php if ($isLab): ?>
      <td><?= (int)$el['PROPERTY_USER_ID_VALUE'] ?></td>
      <?php endif; ?>
      <td>
        <a href="/local/glab/application/index.php?block_id=<?= $el['IBLOCK_SECTION_ID'] ?>&order_id=<?= $el['ID'] ?>">
          Просмотр
        </a>
      </td>
    </tr>
  <?php endwhile; ?>
  </tbody>
</table>

<style>
  .orders-table { width:100%; border-collapse:collapse; }
  .orders-table th { background:#f0f0f0; }
  .orders-table td, .orders-table th { text-align:left; }
</style>

<?php
// 8) Подключаем футер
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
