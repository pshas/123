Ниже — полный список правок, если вы хотите держать всё своё «лабораторное» приложение в папке `/local/glab` (вместо `/lab` или `/local/lab`). Мы сохраняем ваши существующие CSS-классы и минимально правим шаблоны.

---

## 1. Перенос кода в `/local/glab`

1. В корне вашего сайта выполните (bash/ftp/IDE):

   ```bash
   mv /local/lab   /local/glab
   ```

2. Убедитесь, что в каждой странице (`index.php`, `my_application/index.php` и т.д.) в самом верху подключён пролог:

   ```php
   <?php
   require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");
   // …ваш код…
   require($_SERVER["DOCUMENT_ROOT"]."/bitrix/footer.php");
   ```

   Если отсутствует — добавьте.

---

## 2. Меню в `/local/glab/.top.menu.php`

Создайте (или перезапишите) файл `/local/glab/.top.menu.php`:

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

$aMenuLinks = [
    [
      "Главная",
      "/local/glab/",
      [],
      ["class" => "ga-nav main"],
      ""
    ],
    [
      "Мои заявки",
      "/local/glab/my_application/",
      [],
      ["class" => "ga-nav application"],
      ""
    ],
    [
      "В работе",
      "/local/glab/tasks/",
      [],
      ["class" => "ga-nav tasks"],
      ""
    ],
    [
      "Отчёты",
      "/local/glab/reports/",
      [],
      ["class" => "ga-nav reports"],
      ""
    ],
    [
      "Архив",
      "/local/glab/archive/",
      [],
      ["class" => "ga-nav archive"],
      ""
    ],
    [
      "Контакты",
      "/local/glab/contacts/",
      [],
      ["class" => "ga-nav contacts"],
      ""
    ],
];
```

---

## 3. (Опционально) `/local/glab/.top.menu_ext.php`

Если хотите динамически дополнять пункты (например, из инфоблока), создайте рядом:

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

// Пример: разделы инфоблока Reports (ID=5)
$iblockId = 5;
$rs = CIBlockSection::GetList(
    ["SORT"=>"ASC"],
    ["IBLOCK_ID"=>$iblockId, "ACTIVE"=>"Y"],
    false,
    ["NAME","SECTION_PAGE_URL"]
);
while ($sec = $rs->Fetch()) {
    $aMenuLinks[] = [
      $sec["NAME"],
      $sec["SECTION_PAGE_URL"],
      [],
      ["class"=>"ga-nav reports"],
      ""
    ];
}
```

---

## 4. Подключение компонента в шаблоне

В вашем шаблоне (обычно `header.php` в `/local/templates/<ваш_шаблон>/`) уберите старый HTML-блок меню и вставьте:

```php
<?php
// header.php (начало файла)
use Bitrix\Main\Page\Asset;

// Подключаем стили — если вы их ещё не вставили где-то выше
Asset::getInstance()->addCss("/local/templates/<ваш_шаблон>/components/bitrix/menu/top/style.css");

// Подключаем компонент меню
$APPLICATION->IncludeComponent(
    "bitrix:menu",
    "top",
    [
        "ROOT_MENU_TYPE"       => "top",
        "MAX_LEVEL"            => "1",
        "CHILD_MENU_TYPE"      => "",
        "USE_EXT"              => "Y",
        "MENU_CACHE_TYPE"      => "A",
        "MENU_CACHE_TIME"      => "3600",
        "MENU_CACHE_USE_GROUPS" => "Y",
        "MENU_CACHE_GET_VARS"  => []
    ],
    false
);
?>
```

---

## 5. Шаблон компонента `/local/templates/<ваш_шаблон>/components/bitrix/menu/top/`

Создайте эту папку и два файла:

### 5.1. `template.php`

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();
?>
<div id="navigation" class="navig">
  <div id="menu-container">
    <?php foreach ($arResult as $item): 
        // базовый класс + «active», если текущая страница
        $cls = $item["PARAMS"]["class"] . ($item["SELECTED"] ? " active" : "");
    ?>
      <a href="<?= htmlspecialcharsbx($item["LINK"])?>"
         class="<?= htmlspecialcharsbx($cls)?>">
        <?= htmlspecialcharsbx($item["TEXT"]) ?>
      </a>
    <?php endforeach; ?>
  </div>
</div>
```

### 5.2. `style.css`

Если вы уже подключаете и используете свои CSS-правила для `.navig`, `#menu-container`, `.ga-nav` и `.active`, **никаких** новых стилей добавлять не надо.
Если же нужно, вот базовый набор (вдруг поможет):

```css
.menu__fallback { /* если понадобятся общие обёртки */ }
.navig {
  background: #fafafa;
  padding: 0 20px;
}
#menu-container {
  display: flex;
}
.ga-nav {
  margin-right: 30px;
  text-decoration: none;
  color: #333;
  padding: 10px 0;
  transition: color .2s;
}
.ga-nav:hover,
.ga-nav.active {
  color: #0073e6;
}
```

---

## 6. Очистка кеша

После правок не забудьте сбросить кеш:

* **Админка** → Настройки → Производительность → Удалить кеш
* Или добавьте `?clear_cache=Y` к любому URL в браузере.

---

Теперь:

* Меню будет подхватываться из `/local/glab/.top.menu.php` и `.top.menu_ext.php`.
* Ваши старые классы `ga-nav`, `main`, `active` и т.п. сохранятся.
* Компонент автоматически соберёт и выведет ссылки на всех страницах `/local/glab/...`.

Если что-то не „цепляется“ — дайте знать, разберёмся!
