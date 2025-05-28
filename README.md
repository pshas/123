Давайте проверим по шагам, почему у нас ничего не выводится, и убедимся, что ваш самописный `CMenu` действительно грузит файл `/local/glab/.top.menu.php` и проставляет флаг `SELECTED`.

---

## 1. Удаляем админ-реквизиты

1. Зайдите на любую страницу `/local/glab/...` **без** `?bitrix_include_areas=Y` в URL.
2. Очистите кеш Битрикс (Админка → Настройки → Производительность → Удалить кеш).

---

## 2. Быстрый дебаг массива пунктов

Временно добавьте перед вашим HTML-блоком меню следующий код (лучше прямо в шаблон вашего header.php или сразу после `require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");`):

```php
<?php
// 1) Проверим, что файл .top.menu.php вообще читается
echo '<h3>Проверка включения .top.menu.php</h3><pre>';
include $_SERVER['DOCUMENT_ROOT'].'/local/glab/.top.menu.php';
print_r($aMenuLinks);
echo '</pre>';

// 2) Проверим, что CMenu его загружает и проставляет SELECTED
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
echo '<h3>Результат CMenu->arResult</h3><pre>';
print_r($menu->arResult);
echo '</pre>';
?>
```

* Если **в первом** выводе (`$aMenuLinks`) вы видите ваш массив ссылок — значит файл лежит там, где нужно, и читается.
* Если **во втором** (`$menu->arResult`) он пустой или без флага `"SELECTED" => true`, значит `CMenu` не видит/не грузит ваш `.top.menu.php`.

---

## 3. Возможные причины и решения

| Симптом                                                   | Причина                                         | Решение                                                                                                                |
| --------------------------------------------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Первый вывод пустой или ошибка «No such file»             | Неправильный путь/имя файла                     | Проверьте, что файл именно `/local/glab/.top.menu.php` (с точкой в начале). Убедитесь в регистре букв и правах чтения. |
| Второй вывод пустой (хоть `$aMenuLinks` есть)             | CMenu инициализируется не в том месте, где ждёт | Убедитесь, что вы вызываете `new CMenu("top")` и `Init()` **после** подключения `header.php`.                          |
| Во втором выводе есть элементы, но нет `"SELECTED"=>true` | Текущий URL не совпадает ни с одной ссылкой     | Проверьте возвращаемое `$APPLICATION->GetCurPage(false)` и поле `LINK` в массиве. Они должны совпадать по префиксу.    |

---

## 4. Когда отладка покажет, что `arResult` правильный

Уберите временный `print_r` и оставьте вывод меню:

```php
<?php
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

Убедитесь, что:

* Файл `/local/glab/.top.menu.php` содержит корректные пути (`"/local/glab/"` и т.д.).
* Вы **не** заходите с `?bitrix_include_areas=Y`.
* Код стоит **после** `require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");`.

---

С этими шагами вы точно увидите, где теряется ваш массив и почему меню не рендерится/не подсвечивается. Дайте знать результаты отладки — вместе быстро решим!
