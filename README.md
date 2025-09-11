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

$elementId = (int)($_GET['id'] ?? 0);
if (!$elementId) {
    ShowError("Не указан ID заявки");
    require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
    exit;
}

// Загружаем заявку
$res = CIBlockElement::GetList([], ["IBLOCK_ID"=>$ordersIblockId,"ID"=>$elementId], false, false);
if (!($obElement = $res->GetNextElement())) {
    ShowError("Заявка не найдена");
    require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
    exit;
}

$fields = $obElement->GetFields();     // стандартные поля
$props  = $obElement->GetProperties(); // все свойства инфоблока

$status = $props['STATUS']['VALUE'];
if ($status !== 'new') {
    ShowError("Редактирование доступно только для заявок со статусом NEW");
    require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
    exit;
}

// Обработка формы
if ($_SERVER['REQUEST_METHOD']==='POST' && check_bitrix_sessid()) {
    $el = new CIBlockElement;

    $PROP = [
        "WORK_DEPARTMENT"   => $_POST['work_department'],
        "DETAIL_NAME"       => $_POST['detail_name'],
        "DETAIL_NUMBER"     => $_POST['detail_number'],
        "DATE_PRODUCTION"   => $_POST['date_production'],
        "BATCH_NUMBER"      => $_POST['batch_number'],
        "NONCONFORMANCE"    => $_POST['nonconformance'],
        "STATUS"            => $_POST['status'],
    ];

    $arLoad = [
        "MODIFIED_BY"    => $USER->GetID(),
        "IBLOCK_ID"      => $ordersIblockId,
        "PROPERTY_VALUES"=> $PROP,
        "NAME"           => $_POST['detail_name'], 
    ];

    if ($el->Update($elementId, $arLoad)) {
        LocalRedirect("/lab_orders/orders_list.php");
    } else {
        echo "Ошибка: ".$el->LAST_ERROR;
    }
}

// Форма
?>
<form method="POST">
    <?= bitrix_sessid_post() ?>
    <p>Цех/Отдел: <input type="text" name="work_department" value="<?=htmlspecialchars($props['WORK_DEPARTMENT']['VALUE'])?>"></p>
    <p>Наименование детали: <input type="text" name="detail_name" value="<?=htmlspecialchars($props['DETAIL_NAME']['VALUE'])?>"></p>
    <p>Номер детали: <input type="text" name="detail_number" value="<?=htmlspecialchars($props['DETAIL_NUMBER']['VALUE'])?>"></p>
    <p>Дата изготовления: <input type="date" name="date_production" value="<?=htmlspecialchars($props['DATE_PRODUCTION']['VALUE'])?>"></p>
    <p>Номер партии: <input type="text" name="batch_number" value="<?=htmlspecialchars($props['BATCH_NUMBER']['VALUE'])?>"></p>
    <p>Несоответствие: <textarea name="nonconformance"><?=htmlspecialchars($props['NONCONFORMANCE']['VALUE'])?></textarea></p>
    <p>Статус: 
        <select name="status">
            <option value="new" <?=($status=="new"?"selected":"")?>>new</option>
            <option value="in_work" <?=($status=="in_work"?"selected":"")?>>in_work</option>
            <option value="done" <?=($status=="done"?"selected":"")?>>done</option>
        </select>
    </p>
    <p><button type="submit">Сохранить</button></p>
</form>

<?php
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
