Учитывая, что ваш шаблон лежит в `/bitrix/templates/lab/`, получаем такую структуру:

```
/bitrix
  └── templates
      └── lab
          ├── header.php
          ├── footer.php
          └── styles.css
```

Ниже — готовый код для каждого файла.

---

## 1. `/bitrix/templates/lab/header.php`

```php
<?php
// Подключаем пролог и запускаем админ-панель
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php');

use Bitrix\Main\Loader;
if (!Loader::includeModule('main')) {
    // Логировать или вывести ошибку при отсутствии модуля
}

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

// Подключаем свой CSS из этой же папки
Bitrix\Main\Page\Asset::getInstance()->addCss($templateFolder."/styles.css");
?>
<!DOCTYPE html>
<html lang="ru">
<head>
    <?php $APPLICATION->ShowHead(); ?>
</head>
<body>
<?php $APPLICATION->ShowPanel(); ?>

<header class="site-header">
  <div class="wrapper">
    <div class="logo">
      <a href="/local/glab/">GLab</a>
    </div>
    <nav id="navigation" class="navig">
      <div id="menu-container">
        <?php foreach ($menu->arResult as $item):
            $cls = $item["PARAMS"]["class"]
                 . ($item["SELECTED"] ? " active" : "");
        ?>
          <a href="<?=htmlspecialcharsbx($item["LINK"])?>"
             class="<?=htmlspecialcharsbx($cls)?>">
            <?=htmlspecialcharsbx($item["TEXT"])?>
          </a>
        <?php endforeach; ?>
      </div>
    </nav>
  </div>
</header>

<main class="site-content wrapper">
```

---

## 2. `/bitrix/templates/lab/footer.php`

```php
</main>

<footer class="site-footer">
  <div class="wrapper">
    &copy; <?=date("Y")?> GLab. Все права защищены.
  </div>
</footer>

<?php
// Подключаем футер Битрикса, закрываем теги и подключаем скрипты
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php');
?>
```

---

## 3. `/bitrix/templates/lab/styles.css`

```css
/* Обёртка */
.wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 15px;
}

/* Шапка */
.site-header {
  background: #fafafa;
  border-bottom: 1px solid #ddd;
  padding: 10px 0;
}
.site-header .logo a {
  font-size: 24px;
  font-weight: bold;
  color: #333;
  text-decoration: none;
}

/* Меню */
.navig {
  margin-top: 5px;
}
#menu-container {
  display: flex;
}
.ga-nav {
  margin-right: 20px;
  text-decoration: none;
  color: #333;
  padding: 5px 0;
  transition: color 0.2s;
}
.ga-nav:hover,
.ga-nav.active {
  color: #0073e6;
}

/* Контент */
.site-content {
  padding: 20px 0;
}

/* Футер */
.site-footer {
  background: #f0f0f0;
  text-align: center;
  padding: 15px 0;
  font-size: 14px;
  color: #777;
}
```

---

### Что дальше

1. **Убедитесь**, что в разделе `/local/glab/` ваши страницы подключают корневые `header.php` и `footer.php`.
2. **Сбросьте кеш** (Настройки → Производительность → Удалить кеш).
3. **Проверьте** в браузере `/local/glab/` — должно отрисоваться ваше меню с подсветкой активного пункта.
