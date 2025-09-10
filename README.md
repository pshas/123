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

$blockRef = isset($_GET['block_id']) ? (int)$_GET['block_id'] : 0;

$errors  = [];
$success = false;

if ($_SERVER['REQUEST_METHOD']==='POST' && check_bitrix_sessid()) {
    $fullName      = trim($_POST['full_name']       ?? '');
    $phone         = trim($_POST['personal_phone']  ?? '');
    $position      = trim($_POST['work_position']   ?? '');
    $task          = trim($_POST['task']            ?? '');
    $note          = trim($_POST['note']            ?? '');

    // Простая валидация
    if ($fullName ==='' ) $errors[] = 'Укажите ФИО';
    if ($phone    ==='' ) $errors[] = 'Укажите телефон';

    if (empty($errors)) {
        $el = new CIBlockElement;
        $fields = [
            'IBLOCK_ID'       => $ordersIblockId,
            'NAME'            => "Заявка от {$fullName}",
            'ACTIVE'          => 'Y',
            'IBLOCK_SECTION_ID' => $blockRef, 
            'PROPERTY_VALUES' => [
                'USER_ID'    => $USER->GetID(),
                'BLOCK_REF'  => $blockRef,
                'FULL_NAME'  => $fullName,
                'PHONE'      => $phone,
                'POSITION'   => $position,
                'TASK'       => $task,
                'NOTE'       => $note,
                'STATUS'     => 'NEW',
            ],
        ];
        if ($newId = $el->Add($fields)) {
            $success = true;
        } else {
            $errors[] = 'Ошибка сохранения: '.$el->LAST_ERROR;
        }
    }
}

if ($success): ?>
    <div class="ok">Ваша заявка успешно сохранена. №<?= $newId ?></div>
<?php elseif ($errors): ?>
    <div class="errors">
      <?php foreach($errors as $e): ?>
        <p style="color:red"><?= htmlspecialcharsbx($e) ?></p>
      <?php endforeach; ?>
    </div>
<?php endif; ?>
