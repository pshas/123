Поскольку вы хотите рисовать меню «своими руками», но при этом использовать логику Битрикса (загрузку ссылок из `.top.menu.php` и проставление флага `SELECTED`), можно обойтись без вызова компонента и вывести всё вручную, но на базе класса `CMenu`. Он умеет подгружать ваши файлы `.top.menu.php`/`.top.menu_ext.php`, кешировать и помечать активный пункт.

---

### Шаг 1. Убедитесь, что в `/local/glab/.top.menu.php` у вас лежит вот такой массив (мы уже делали):

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();

$aMenuLinks = [
  ["Главная",    "/local/glab/",               [], ["class"=>"ga-nav main"],      ""],
  ["Мои заявки", "/local/glab/my_application/", [], ["class"=>"ga-nav application"],""],
  ["В работе",   "/local/glab/tasks/",         [], ["class"=>"ga-nav tasks"],     ""],
  ["Отчёты",     "/local/glab/reports/",       [], ["class"=>"ga-nav reports"],   ""],
  ["Архив",      "/local/glab/archive/",       [], ["class"=>"ga-nav archive"],   ""],
  ["Контакты",   "/local/glab/contacts/",      [], ["class"=>"ga-nav contacts"],  ""],
];
```

(И опционально `/local/glab/.top.menu_ext.php` для динамики.)

---

### Шаг 2. Вставляем самописный вывод на всех ваших `/local/glab/.../index.php`

В самом начале каждой страницы (после `require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");`) или в вашем общем `header.php`, замените старый блок на такой код:

```php
<?php
use Bitrix\Main\Loader;
Loader::includeModule('main');

// Инициализируем CMenu для «top» типa
$menu = new CMenu("top");
$menu->Init([
  "ROOT_MENU_TYPE"       => "top",
  "MAX_LEVEL"            => 1,
  "CHILD_MENU_TYPE"      => "",
  "USE_EXT"              => "Y",
  "MENU_CACHE_TYPE"      => "A",
  "MENU_CACHE_TIME"      => 3600,
  "MENU_CACHE_USE_GROUPS" => "Y",
  "MENU_CACHE_GET_VARS"  => []
]);
?>
<div id="navigation" class="navig">
  <div id="menu-container">
    <?php foreach ($menu->arResult as $item):
      // базовый класс из PARAMS + active, если SELECTED=true
      $cls = $item["PARAMS"]["class"]
           . ($item["SELECTED"] ? " active" : "");
    ?>
      <a href="<?= htmlspecialcharsbx($item["LINK"])?>"
         class="<?= htmlspecialcharsbx($cls)?>">
        <?= htmlspecialcharsbx($item["TEXT"]) ?>
      </a>
    <?php endforeach; ?>
  </div>
</div>
```

**Что здесь происходит**:

1. `new CMenu("top")` инициализирует загрузку из `/local/glab/.top.menu.php` (тип меню `top`).
2. Параметр `USE_EXT => "Y"` попробует подцепить файл `.top.menu_ext.php`.
3. Битрикс сам проставит у каждого пункта `$item["SELECTED"] = true`, если текущий URL начинается с его `LINK`.
4. Вы просто берёте `class` из `$item["PARAMS"]` (куда мы раньше записывали `["class"=>"ga-nav ..."]`) и дописываете `" active"`, когда выбран.

---

### Шаг 3. Убедитесь, что:

* Все ваши страницы в `/local/glab/...` подключают `header.php`/`footer.php`.
* Вы **не** открываете URL с `?bitrix_include_areas=Y`, иначе увидите админ-панель слева.
* Сбросили кеш (Админка → Настройки → Производительность → Удалить кеш).

---

С этим подходом у вас остаётся **100% свой HTML/CSS**, но активный пункт меню подсветится автоматически силами Битрикса, и вы можете дальше расширять `.top.menu_ext.php` или править массив в `.top.menu.php` без дублирования логики.
