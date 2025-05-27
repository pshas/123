Если файлы `.section.php` есть в каждой папке проекта, настройка активного пункта меню через них может быть неудобной. Вместо этого можно реализовать подсветку активного пункта меню прямо в коде шаблона, сравнивая текущий URL с ссылками меню.

Измените код меню в вашем шаблоне:

```php
<div id="navigation" class="navig">
    <div id="menu-container">
        <?php
        $currentPage = $APPLICATION->GetCurPage(false);
        $menuItems = [
            ["link" => "/local/lab/", "class" => "ga-nav main", "text" => "Главная"],
            ["link" => "/local/lab/my_application", "class" => "ga-nav application", "text" => "Мои заявки"],
            ["link" => "/local/lab/tasks", "class" => "ga-nav tasks", "text" => "В работе"],
            ["link" => "/local/lab/", "class" => "ga-nav reports", "text" => "Отчёты"],
            ["link" => "/local/lab/", "class" => "ga-nav archive", "text" => "Архив"],
            ["link" => "/local/lab/contacts/", "class" => "ga-nav contacts", "text" => "Контакты"],
        ];

        foreach ($menuItems as $item) {
            $isActive = (strpos($currentPage, $item["link"]) === 0) ? "active" : "";
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

Использование `strpos` вместо строгого сравнения учитывает возможные вложенные пути. Это решение не требует изменений в `.section.php` и работает централизованно в шаблоне.
