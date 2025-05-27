В `.section.php` добавьте проверку текущей страницы и установите класс `active`:

```php
$this->setFrameMode(true);
$APPLICATION->AddChainItem($arResult["NAME"], $arResult["LINK"]);
$currentPage = $APPLICATION->GetCurPage(false);
if ($currentPage === $arResult["LINK"]) {
    $arResult["PARAMS"]["class"] = "active";
}
```

Измените HTML меню в вашем коде:

```php
<div id="navigation" class="navig">
    <div id="menu-container">
        <?php
        $menuItems = [
            ["link" => "/local/lab/", "class" => "ga-nav main", "text" => "Главная"],
            ["link" => "/local/lab/my_application", "class" => "ga-nav application", "text" => "Мои заявки"],
            ["link" => "/local/lab/tasks", "class" => "ga-nav tasks", "text" => "В работе"],
            ["link" => "/local/lab/", "class" => "ga-nav reports", "text" => "Отчёты"],
            ["link" => "/local/lab/", "class" => "ga-nav archive", "text" => "Архив"],
            ["link" => "/local/lab/contacts/", "class" => "ga-nav contacts", "text" => "Контакты"],
        ];

        foreach ($menuItems as $item) {
            $isActive = $APPLICATION->GetCurPage(false) === $item["link"] ? "active" : "";
            ?>
            <a href="<?= $item["link"] ?>" class="<?= $item["class"] ?> <?= $isActive ?>"><?= $item["text"] ?></a>
            <?php
        }
        ?>
    </div>
</div>
```

В CSS добавьте стиль:

```css
.active {
    background-color: #007bff;
    color: #fff;
}
```

Это подсветит активный пункт меню на основе текущей страницы.
