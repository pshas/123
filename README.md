Ниже пример, как разделить вывод «компаний» и «подразделений» на две разные страницы.

---

## 1. Меняем шаблон списка компаний

Вместо аккордеона на той же странице, теперь каждая компания будет ссылкой на отдельную страницу `/local/glab/subdivisions.php`. Откройте ваш файл

```
/bitrix/templates/lab/components/bitrix/news.list/lab_blocks_template/template.php
```

и замените его на:

```php
<?php if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die(); ?>

<div id="companies" class="row">
    <?php
    CModule::IncludeModule("iblock");
    $rsCompanies = CIBlockSection::GetList(
        ["SORT"=>"ASC"],
        ["IBLOCK_ID"=>$arParams["IBLOCK_ID"], "SECTION_ID"=>0, "ACTIVE"=>"Y"],
        false,
        ["ID","NAME","DESCRIPTION"]
    );
    while ($comp = $rsCompanies->Fetch()):
        $id   = (int)$comp["ID"];
        $name = htmlspecialcharsbx($comp["NAME"]);
        $desc = nl2br(htmlspecialcharsbx($comp["DESCRIPTION"]));
    ?>
    <a class="item col-md" href="/local/glab/subdivisions.php?company_id=<?= $id ?>">
        <div class="icon" style="background-image:url('<?= $templateFolder ?>/img/company.png');"></div>
        <div class="title"><?= $name ?></div>
        <div class="text"><?= $desc ?></div>
    </a>
    <?php endwhile; ?>
</div>
```

* убрали вложенные циклы, каждая `<a>` ведёт на `subdivisions.php?company_id=ID`.

---

## 2. Создаём страницу вывода подразделений

Создайте файл `/local/glab/subdivisions.php` со следующим содержанием:

```php
<?php
// 1) Запускаем шаблон сайта
require_once $_SERVER['DOCUMENT_ROOT'].'/header.php';
\Bitrix\Main\Loader::includeModule('iblock');

// 2) Получаем ID компании из GET
$companyId = isset($_GET['company_id']) ? (int)$_GET['company_id'] : 0;
if ($companyId <= 0) {
    ShowError("Компания не выбрана");
    require_once $_SERVER['DOCUMENT_ROOT'].'/footer.php';
    exit;
}

// 3) Заголовок страницы — название компании
$rs = CIBlockSection::GetByID($companyId);
if ($ar = $rs->Fetch()) {
    $APPLICATION->SetTitle("Подразделения: ".$ar['NAME']);
} else {
    ShowError("Компания не найдена");
    require_once $_SERVER['DOCUMENT_ROOT'].'/footer.php';
    exit;
}

// 4) Выводим подразделения
?>
<div id="companies" class="row">
    <?php
    $rsSubs = CIBlockSection::GetList(
        ["SORT"=>"ASC"],
        ["IBLOCK_ID"=>10, "SECTION_ID"=>$companyId, "ACTIVE"=>"Y"], // замените 10 на ваш IBLOCK_ID
        false,
        ["ID","NAME","DESCRIPTION","DETAIL_PAGE_URL"]
    );
    while ($sub = $rsSubs->Fetch()):
        $id   = (int)$sub["ID"];
        $name = htmlspecialcharsbx($sub["NAME"]);
        $desc = nl2br(htmlspecialcharsbx($sub["DESCRIPTION"]));
        $link = htmlspecialcharsbx($sub["DETAIL_PAGE_URL"] ?: "#");
    ?>
    <a class="item col-md" href="<?= $link ?>">
        <div class="icon" style="background-image:url('<?= $templateFolder ?>/img/company.png');"></div>
        <div class="title"><?= $name ?></div>
        <div class="text"><?= $desc ?></div>
    </a>
    <?php endwhile; ?>
</div>

<?php
// 5) Подключаем общий футер
require_once $_SERVER['DOCUMENT_ROOT'].'/footer.php';
```

* **Не забудьте** заменить `10` в фильтре на реальный `IBLOCK_ID` вашего инфоблока.

---

## 3. Стили

Если у вас уже есть в `style.css` правила для `#companies .item`, `.icon`, `.title`, `.text` — достаточно сбросить кеш. Иначе проверьте, что в `/bitrix/templates/lab/components/bitrix/news.list/lab_blocks_template/style.css` присутствуют:

```css
#companies { display:flex; flex-wrap:wrap; gap:26px; margin:30px 0; }
#companies .item {
  width:268px; background:#fff; padding:15px; text-decoration:none; color:inherit;
}
#companies .item .icon {
  height:75px; background-size:contain; background-repeat:no-repeat; margin-bottom:10px;
}
#companies .item .title {
  margin:10px 0; font-size:20px; font-weight:600;
}
#companies .item .text {
  font-size:18px; line-height:1.4; height:160px; overflow:hidden;
}
```

---

### Итог

1. **template.php** теперь выводит только компании и ссылки на `subdivisions.php?company_id=...`
2. **subdivisions.php** получает `company_id`, выводит заголовок и подгружает только подразделения этой компании.
3. **Стили** остаются теми же, что вы уже настроили для карточек.

После очистки кеша и перехода на `/local/glab/subdivisions.php?company_id=15` (пример) вы увидите список подразделений выбранной компании на отдельной странице.
