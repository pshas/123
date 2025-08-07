<?php
    // 5) Выбираем подразделения — разделы второго уровня инфоблока
    $rsSubs = CIBlockSection::GetList(
        ["SORT" => "ASC"],
        [
            "IBLOCK_ID"   => $iblockId,
            "SECTION_ID"  => $companyId,
            "ACTIVE"      => "Y"
        ],
        false,
        ["ID", "NAME", "DESCRIPTION", "DETAIL_PAGE_URL"]
    );
    while ($sub = $rsSubs->Fetch()):
        $link = htmlspecialcharsbx($sub["DETAIL_PAGE_URL"] ?: "#");
        $name = htmlspecialcharsbx($sub["NAME"]);
        $desc = nl2br(htmlspecialcharsbx($sub["DESCRIPTION"]));
    ?>
    <a class="item col-md" href="<?= $link ?>">
        <div class="icon" style="background-image:url('<?= $templateFolder ?>/img/company.png');"></div>
        <div class="title"><?= $name ?></div>
        <div class="text"><?= $desc ?></div>
    </a>
    <?php endwhile; ?>
