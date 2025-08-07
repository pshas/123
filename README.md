<?php
// Модуль инфоблоков уже подключён и $iblockId, $companyId определены выше

// 1) Получаем список подразделений (вложенных разделов)
$rsSubs = CIBlockSection::GetList(
    ["SORT" => "ASC"],
    [
        "IBLOCK_ID"  => $iblockId,
        "SECTION_ID" => $companyId,
        "ACTIVE"     => "Y"
    ],
    false,
    ["ID","NAME","DESCRIPTION"]
);

while ($sub = $rsSubs->Fetch()):
    $subId = (int)$sub["ID"];
    // 2) Линк на приложение – передаём block_id
    $link  = "/local/glab/application/index.php?block_id=" . $subId;

    $name  = htmlspecialcharsbx($sub["NAME"]);
    $desc  = nl2br(htmlspecialcharsbx($sub["DESCRIPTION"]));

    // 3) Путь до иконки (положите вашу картинку в /local/glab/img/company.png)
    $iconPath = "/local/glab/img/company.png";
?>
    <a class="item col-md" href="<?= $link ?>">
        <div class="icon"
             style="background-image:url('<?= $iconPath ?>');"></div>
        <div class="title"><?= $name ?></div>
        <div class="text"><?= $desc ?></div>
    </a>
<?php endwhile; ?>
