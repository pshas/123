Вот переработанный пример `subdivisions.php`, в котором **IBLOCK\_ID** автоматически определяется по символьному коду инфоблока (`lab_blocks`), а не прописывается вручную:

```php
<?php
// 1) Запускаем движок и подключаем prolog
require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php';
\Bitrix\Main\Loader::includeModule('iblock');

// 2) Определяем ID инфоблока по его символьному коду
$iblockCode = 'lab_blocks';    // ваш CODE инфоблока
$rsIblock = CIBlock::GetList(
    [],
    ["CODE" => $iblockCode, "ACTIVE" => "Y"],
    false
);
if (!$ib = $rsIblock->Fetch()) {
    ShowError("Инфоблок с кодом «{$iblockCode}» не найден");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}
$iblockId = (int)$ib['ID'];

// 3) Получаем ID компании из GET и проверяем
$companyId = isset($_GET['company_id']) ? (int)$_GET['company_id'] : 0;
if ($companyId <= 0) {
    ShowError("Компания не выбрана");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}

// 4) Получаем данные самой компании, чтобы поставить заголовок
$rsCompany = CIBlockSection::GetByID($companyId);
if ($company = $rsCompany->Fetch()) {
    $APPLICATION->SetTitle("Подразделения «" . htmlspecialcharsbx($company['NAME']) . "»");
} else {
    ShowError("Компания не найдена");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}

?>
<div id="companies" class="row">
    <?php
    // 5) Выбираем подразделения — разделы второго уровня инфоблока
    $rsSubs = CIBlockSection::GetList(
        ["SORT" => "ASC"],
        [
            "IBLOCK_ID"   => $iblockId,
            "SECTION_ID"  => $companyId,
            "ACTIVE"      => "Y"
        ],
        false,
        ["ID", "NAME", "DESCRIPTION", "DETAIL_PAGE_URL"]
    );
    while ($sub = $rsSubs->Fetch()):
        $link = htmlspecialcharsbx($sub["DETAIL_PAGE_URL"] ?: "#");
        $name = htmlspecialcharsbx($sub["NAME"]);
        $desc = nl2br(htmlspecialcharsbx($sub["DESCRIPTION"]));
    ?>
    <a class="item col-md" href="<?= $link ?>">
        <div class="icon" style="background-image:url('<?= $templateFolder ?>/img/company.png');"></div>
        <div class="title"><?= $name ?></div>
        <div class="text"><?= $desc ?></div>
    </a>
    <?php endwhile; ?>
</div>
<?php
// 6) Подключаем footer
require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
```

### Пояснения

1. **Определение IBLOCK\_ID по коду**

   ```php
   $iblockCode = 'lab_blocks';
   $rsIblock = CIBlock::GetList([], ["CODE"=>$iblockCode,"ACTIVE"=>"Y"], false);
   $ib = $rsIblock->Fetch();
   $iblockId = (int)$ib['ID'];
   ```

   — так вы не зависите от цифр, а от символьного кода, который всегда можно проверить в админке.

2. **Проверка входных данных**
   — Если `company_id` не передан или раздел не найден, выводится ошибка и дальнейший код не выполняется.

3. **Выборка подразделений**
   — `CIBlockSection::GetList` с фильтром `SECTION_ID => $companyId` вытягивает только вложенные разделы.

4. **Вывод**
   — Карточки подразделений рендерятся точно по тому же шаблону `.item`, что и на главной странице со списком компаний.

После этого `subdivisions.php` будет универсально работать на любом сайте, где инфоблок имеет код `lab_blocks`, без жестко прописанного числового `IBLOCK_ID`.
