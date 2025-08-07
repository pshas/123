Чтобы подразделение сразу открывало форму вашего «application» (тот самый набор `index.php`/`get-info.php`/`submit.php` в папке `glab/application`), достаточно превратить ссылку на подразделение в прямой переход на

```
/local/glab/application/index.php?block_id=<ID_подразделения>
```

где `block_id` берётся из поля `$_GET['block_id']` внутри `application/index.php` .

---

### Обновлённый фрагмент `subdivisions.php`

```php
<?php
require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php';
\Bitrix\Main\Loader::includeModule('iblock');

// Определяем IBLOCK_ID по символьному коду "lab_blocks" (как делали ранее)
$rsI = CIBlock::GetList([], ["CODE"=>"lab_blocks","ACTIVE"=>"Y"], false);
$ib = $rsI->Fetch();
$iblockId = (int)$ib['ID'];

// Получаем ID компании
$companyId = (int)($_GET['company_id'] ?? 0);
$rsC = CIBlockSection::GetByID($companyId);
if (!$company = $rsC->Fetch()) {
    ShowError("Компания не найдена");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}
$APPLICATION->SetTitle("Подразделения «".htmlspecialcharsbx($company['NAME'])."»");
?>

<div id="companies" class="row">
  <?php
  // Для каждого подразделения берём ID и генерируем ссылку на application/index.php
  $rsSubs = CIBlockSection::GetList(
      ["SORT"=>"ASC"],
      ["IBLOCK_ID"=>$iblockId,"SECTION_ID"=>$companyId,"ACTIVE"=>"Y"],
      false,
      ["ID","NAME","DESCRIPTION"]
  );
  while ($sub = $rsSubs->Fetch()):
      $subId   = (int)$sub["ID"];
      $name    = htmlspecialcharsbx($sub["NAME"]);
      $desc    = nl2br(htmlspecialcharsbx($sub["DESCRIPTION"]));
      // Ссылка на ваш application:
      $link    = "/local/glab/application/index.php?block_id=".$subId;
  ?>
    <a class="item col-md" href="<?= $link ?>">
      <div class="icon"
           style="background-image:url('<?= $templateFolder ?>/img/company.png');"></div>
      <div class="title"><?= $name ?></div>
      <div class="text"><?= $desc ?></div>
    </a>
  <?php endwhile; ?>
</div>

<?php require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php'; ?>
```

* Мы берём из `$sub["ID"]` именно тот `block_id`, который приложение ожидает в `$_GET['block_id']` .
* Папка `glab/application` содержит свой `index.php`, который делает `require("../header.php")` и затем обрабатывает `block_id` для вывода формы .

После этого клик по карточке подразделения приведёт пользователя в нужный раздел вашего «application».
