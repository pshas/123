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
    // собираем значения из формы
    $nameIzd       = trim($_POST['name_izd']        ?? '');
    $idI           = trim($_POST['id_i']            ?? '');
    $dateMan       = trim($_POST['date_man']        ?? '');
    $country       = trim($_POST['country']         ?? '');
    $countDetail   = trim($_POST['count_detail']    ?? '');
    $dateDel       = trim($_POST['date_del']        ?? '');
    $materials     = [];
    if (!empty($_POST['material1'])) $materials[] = $_POST['material1'];
    if (!empty($_POST['material2'])) $materials[] = $_POST['material2'];
    if (!empty($_POST['material3'])) $materials[] = $_POST['material3'];
    $processings   = [];
    if (!empty($_POST['processing1'])) $processings[] = $_POST['processing1'];
    if (!empty($_POST['processing2'])) $processings[] = $_POST['processing2'];

    $dateDetail    = trim($_POST['date_detail']     ?? '');
    $workDept      = trim($_POST['WORK_DEPARTMENT'] ?? '');
    $fullName      = trim($_POST['full_name']       ?? '');
    $phone         = trim($_POST['personal_phone']  ?? '');
    $position      = trim($_POST['work_position']   ?? '');
    $workPhone     = trim($_POST['work_phone']      ?? '');
    $vin           = trim($_POST['VIN']             ?? '');
    $task          = trim($_POST['task']            ?? '');
    $note          = trim($_POST['note']            ?? '');
    $accept        = trim($_POST['accept']          ?? '');
    $handover      = trim($_POST['handover']        ?? '');

    // простая валидация
    if ($fullName === '' ) $errors[] = 'Укажите ФИО';
    if ($phone    === '' ) $errors[] = 'Укажите телефон';
    if ($nameIzd  === '' ) $errors[] = 'Укажите наименование изделия';

    if (empty($errors)) {
        $el = new CIBlockElement;

        $fields = [
            'IBLOCK_ID'         => $ordersIblockId,
            'NAME'              => $nameIzd, // имя элемента = название изделия
            'ACTIVE'            => 'Y',
            'IBLOCK_SECTION_ID' => $blockRef,
            'PROPERTY_VALUES'   => [
                'USER_ID'       => $USER->GetID(),
                'BLOCK_REF'     => $blockRef,
                'NAME_IZD'      => $nameIzd,
                'ID_I'          => $idI,
                'DATE_MAN'      => $dateMan,
                'COUNTRY'       => $country,
                'COUNT_DETAIL'  => $countDetail,
                'DATE_DEL'      => $dateDel,
                'MATERIALS'     => $materials,
                'PROCESSINGS'   => $processings,
                'DATE_DETAIL'   => $dateDetail,
                'WORK_DEPARTMENT' => $workDept,
                'FULL_NAME'     => $fullName,
                'PHONE'         => $phone,
                'POSITION'      => $position,
                'WORK_PHONE'    => $workPhone,
                'VIN'           => $vin,
                'TASK'          => $task,
                'NOTE'          => $note,
                'ACCEPT'        => $accept,
                'HANDOVER'      => $handover,
                'STATUS'        => 'NEW',
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
