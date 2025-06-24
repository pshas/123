<div id="companies" class="row">
    <?php foreach ($arResult["ITEMS"] as $item):
        // Собираем ссылку на детальную или на список, либо используем корневую
        $link = $item["DETAIL_PAGE_URL"] ?: $item["LIST_PAGE_URL"] ?: "/local/glab/";
        $link = htmlspecialcharsbx($link);

        // Текстовые данные
        $name    = htmlspecialcharsbx($item["NAME"]);
        $preview = htmlspecialcharsbx($item["PREVIEW_TEXT"]);

        // Картинка: предварительно проверяем PREVIEW_PICTURE, иначе - плейсхолдер
        if (!empty($item["PREVIEW_PICTURE"])) {
            $pictureUrl = CFile::GetPath($item["PREVIEW_PICTURE"]);
        } else {
            $pictureUrl = $templateFolder . "/img/company.png";
        }
        $pictureUrl = htmlspecialcharsbx($pictureUrl);
    ?>
        <a class="item col-md" href="<?= $link ?>">
            <div class="icon" style="background-image: url('<?= $pictureUrl ?>');"></div>
            <div class="title"><?= $name ?></div>
            <div class="text"><?= $preview ?></div>
        </a>
    <?php endforeach; ?>
</div>
