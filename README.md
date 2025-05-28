Поскольку с прямым вызовом `CMenu::Init()` что-то не сходится (Bitrix действительно жёстко ожидает все параметры-строки), давайте вернёмся к «официальному» пути и подключим готовый компонент `bitrix:menu` прямо в вашем `header.php`. Он корректно принимает массивы и сам внутри вызывает именно `CMenu` с правильными параметрами.

---

## 1. Подключаем компонент в шапке

Откройте свой файл шапки

```
/bitrix/templates/lab/header.php
```

и вместо всего блока с `new CMenu` вставьте:

```php
<?php
// …должен быть подключён пролог через require '/bitrix/header.php' …

use Bitrix\Main\Page\Asset;
Asset::getInstance()->addCss($templateFolder."/styles.css");

// Подключаем стандартный компонент меню
$APPLICATION->IncludeComponent(
  "bitrix:menu",
  "top",            // ваш шаблон компонента (см. ниже)
  [
    "ROOT_MENU_TYPE"        => "top",
    "MAX_LEVEL"             => "1",
    "CHILD_MENU_TYPE"       => "",
    "USE_EXT"               => "Y",
    "DELAY"                 => "N",
    "ALLOW_MULTI_SELECT"    => "N",
    "MENU_CACHE_TYPE"       => "A",
    "MENU_CACHE_TIME"       => "3600",
    "MENU_CACHE_USE_GROUPS" => "Y",
    "MENU_CACHE_GET_VARS"   => []
  ],
  false
);
?>
```

> Важно: здесь мы передаём `MENU_CACHE_GET_VARS` как пустой массив — именно так компонент закодирован и ожидает его именно в таком виде.

---

## 2. Создаём шаблон компонента

Путь для шаблона:

```
/bitrix/templates/lab/components/bitrix/menu/top/template.php
```

и файл стилей (если нужно):

```
/bitrix/templates/lab/components/bitrix/menu/top/style.css
```

### 2.1. `template.php`

```php
<?php if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die(); ?>
<div id="navigation" class="navig">
  <div id="menu-container">
    <?php foreach ($arResult as $item): ?>
      <?php
        // вытаскиваем ваш класс и добавляем active, если SELECTED
        $cls = $item["PARAMS"]["class"]
             . ($item["SELECTED"] ? " active" : "");
      ?>
      <a href="<?=htmlspecialcharsbx($item["LINK"])?>"
         class="<?=htmlspecialcharsbx(trim($cls))?>">
        <?=htmlspecialcharsbx($item["TEXT"])?>
      </a>
    <?php endforeach; ?>
  </div>
</div>
```

### 2.2. Если нужны локальные стили

**style.css** (опционально, ваши старые правила уже в `/bitrix/templates/lab/styles.css`):

```css
/* Переопределять тут можно что угодно */
#navigation.navig { /* … */ }
#menu-container a { /* … */ }
```

---

## 3. Как это работает

1. `IncludeComponent("bitrix:menu", "top", …)` берёт файлы

   * `/bitrix/.top.menu.php` или `/local/glab/.top.menu.php` (ROOT\_MENU\_TYPE=top)
   * плюс (`USE_EXT=Y`) `/local/glab/.top.menu_ext.php`
2. Внутри компонента Bitrix правильно передаёт все параметры в `CMenu::Init()`, никакой ошибки `trim(array)` не будет.
3. Массив `$arResult` попадает в ваш `template.php`, где вы берёте `TEXT`, `LINK`, `PARAMS['class']` и `SELECTED`.

---

## 4. Итоговые шаги

1. Вставьте вызов `IncludeComponent` в `/bitrix/templates/lab/header.php`.
2. Скопируйте `template.php` в `/bitrix/templates/lab/components/bitrix/menu/top/`.
3. (Опционально) добавьте/правьте `style.css` в том же каталоге компонента.
4. Очистите кеш Битрикс.
5. Обновите `/local/glab/` — меню должно заработать с вашими классами и активными ссылками.
