<?php
// Привязываем движок и шаблон
require_once($_SERVER['DOCUMENT_ROOT'].'/header.php');
$APPLICATION->SetTitle("Добавить подразделение");

// Подключаем модуль «Инфоблоки»
\Bitrix\Main\Loader::includeModule('iblock');

$iblockId = 10; // <-- ваш ID инфоблока «Блоки лаборатории»
$errors = [];

// Обработка отправки формы
if ($_SERVER['REQUEST_METHOD'] === 'POST' && check_bitrix_sessid()) {
    $parentId   = (int)$_POST['PARENT_ID'];
    $name       = trim($_POST['NAME']);
    $description= trim($_POST['DESCRIPTION']);

    if ($parentId <= 0) {
        $errors[] = "Нужно выбрать компанию-родителя.";
    }
    if ($name === "") {
        $errors[] = "Введите название подразделения.";
    }

    if (empty($errors)) {
        $bs = new CIBlockSection;
        $arFields = [
            "ACTIVE"            => "Y",
            "IBLOCK_ID"         => $iblockId,
            "NAME"              => $name,
            "DESCRIPTION"       => $description,
            "IBLOCK_SECTION_ID" => $parentId,
        ];
        if ($newId = $bs->Add($arFields)) {
            LocalRedirect("/local/glab/index.php?sub_added=Y");
        } else {
            $errors[] = "Ошибка при добавлении: " . $bs->LAST_ERROR;
        }
    }
}

// Получим список компаний (разделов верхнего уровня)
$rs = CIBlockSection::GetList(
    ["NAME"=>"ASC"],
    ["IBLOCK_ID"=>$iblockId, "SECTION_ID"=>0, "ACTIVE"=>"Y"],
    false,
    ["ID","NAME"]
);
$companies = [];
while ($row = $rs->Fetch()) {
    $companies[$row['ID']] = $row['NAME'];
}
?>

<div class="form-container">
    <h2>Добавить подразделение</h2>

    <?php if ($errors): ?>
        <div class="errors">
            <?php foreach ($errors as $e): ?>
                <p style="color:red;"><?= htmlspecialcharsbx($e) ?></p>
            <?php endforeach; ?>
        </div>
    <?php endif; ?>

    <form method="post">
        <?= bitrix_sessid_post() ?>
        <div class="field">
            <label>Компания-родитель</label><br>
            <select name="PARENT_ID">
                <option value="">-- выберите компанию --</option>
                <?php foreach ($companies as $id => $nm): ?>
                    <option value="<?= $id ?>"
                      <?= ($_POST['PARENT_ID']==$id?'selected':'') ?>>
                      <?= htmlspecialcharsbx($nm) ?>
                    </option>
                <?php endforeach; ?>
            </select>
        </div>

        <div class="field">
            <label>Название подразделения</label><br>
            <input type="text" name="NAME"
                   value="<?= htmlspecialcharsbx($_POST['NAME'] ?? '') ?>"
                   style="width:100%;">
        </div>

        <div class="field">
            <label>Описание</label><br>
            <textarea name="DESCRIPTION"
                      style="width:100%; height:120px;"><?= htmlspecialcharsbx($_POST['DESCRIPTION'] ?? '') ?></textarea>
        </div>

        <div class="field">
            <button type="submit">Добавить</button>
            <a href="/local/glab/index.php">Отмена</a>
        </div>
    </form>
</div>

<?php require_once($_SERVER['DOCUMENT_ROOT'].'/footer.php'); ?>
