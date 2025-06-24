```php
<?php if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die(); ?>

<div id="companies" class="row">
    <?php foreach ($arResult["ITEMS"] as $item): 
        // Ссылка
        $link = htmlspecialcharsbx(
            $item["DETAIL_PAGE_URL"]
            ?: $item["LIST_PAGE_URL"]
            ?: "/local/glab/"
        );

        // Заголовок
        $name = htmlspecialcharsbx($item["NAME"]);

        // Описание из свойства DESCRIPTION
        $desc = "";
        if (!empty($item["PROPERTIES"]["DESCRIPTION"]["VALUE"])) {
            $val = $item["PROPERTIES"]["DESCRIPTION"]["VALUE"];
            $desc = is_array($val)
                ? htmlspecialcharsbx(implode(", ", $val))
                : htmlspecialcharsbx($val);
        }

        // Картинка: PREVIEW_PICTURE или плейсхолдер
        $pictureUrl = !empty($item["PREVIEW_PICTURE"])
            ? CFile::GetPath($item["PREVIEW_PICTURE"])
            : $templateFolder . "/img/company.png";
        $pictureUrl = htmlspecialcharsbx($pictureUrl);
    ?>
        <a class="item col-md" href="<?= $link ?>">
            <div class="icon" style="background-image: url('<?= $pictureUrl ?>');"></div>
            <div class="title"><?= $name ?></div>
            <div class="text"><?= nl2br($desc) ?></div>
        </a>
    <?php endforeach; ?>
</div>
```
