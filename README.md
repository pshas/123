Ниже полный код вашего `template.php` с поддержкой “аккордеона”: при клике на название компании будут плавно показываться или скрываться её подразделения. Положите его в

```
/bitrix/templates/lab/components/bitrix/news.list/lab_blocks_template/template.php
```

и обновите `style.css` того же шаблона, чтобы всё работало.

---

### `template.php`

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

// Подключаем CSS
$cssFile = $templateFolder . "/style.css";
if (file_exists($_SERVER['DOCUMENT_ROOT'] . $cssFile)) {
    \Bitrix\Main\Page\Asset::getInstance()->addCss($cssFile);
}

// Подключаем jQuery, если ещё не подключён (можно использовать и чистый JS)
\CJSCore::Init(['jquery']);
?>

<div id="companies" class="company-list">
    <?php
    // Загружаем модуль инфоблоков
    \CModule::IncludeModule("iblock");

    // 1) Выбираем компании (разделы 1 уровня)
    $rsCompanies = CIBlockSection::GetList(
        ["SORT" => "ASC"],
        ["IBLOCK_ID" => $arParams["IBLOCK_ID"], "SECTION_ID" => 0, "ACTIVE" => "Y"],
        false,
        ["ID", "NAME"]
    );
    while ($company = $rsCompanies->Fetch()):
        $companyId   = (int)$company["ID"];
        $companyName = htmlspecialcharsbx($company["NAME"]);
    ?>
    <div class="company-group" data-company-id="<?= $companyId ?>">
        <div class="company-header">
            <?= $companyName ?>
            <span class="toggle-icon">+</span>
        </div>
        <div class="subdivisions">
            <?php
            // 2) Берём подразделения этой компании (разделы 2 уровня)
            $rsSubs = CIBlockSection::GetList(
                ["SORT" => "ASC"],
                [
                  "IBLOCK_ID"  => $arParams["IBLOCK_ID"],
                  "SECTION_ID" => $companyId,
                  "ACTIVE"     => "Y"
                ],
                false,
                ["ID", "NAME", "DESCRIPTION", "DETAIL_PAGE_URL", "UF_ICON"]
            );
            while ($sub = $rsSubs->Fetch()):
                $subId    = (int)$sub["ID"];
                $subName  = htmlspecialcharsbx($sub["NAME"]);
                $subDesc  = htmlspecialcharsbx($sub["DESCRIPTION"]);
                // ссылка на подраздел
                $subLink  = htmlspecialcharsbx($sub["DETAIL_PAGE_URL"] ?: "#");
                // иконка подразделения, если есть UF_ICON
                $iconUrl  = !empty($sub["UF_ICON"])
                    ? CFile::GetPath($sub["UF_ICON"])
                    : ($templateFolder . "/img/company.png");
                $iconUrl  = htmlspecialcharsbx($iconUrl);
            ?>
            <a class="item col-md" href="<?= $subLink ?>">
                <div class="icon" style="background-image:url('<?= $iconUrl ?>');"></div>
                <div class="title"><?= $subName ?></div>
                <div class="text"><?= nl2br($subDesc) ?></div>
            </a>
            <?php endwhile; ?>
        </div>
    </div>
    <?php endwhile; ?>
</div>

<script>
  (function($){
    $('#companies .company-header').on('click', function(){
      var $grp = $(this).closest('.company-group'),
          $subs = $grp.find('> .subdivisions'),
          $icon = $(this).find('.toggle-icon');
      $subs.slideToggle(200);
      $icon.text($subs.is(':visible') ? '–' : '+');
    });
  })(jQuery);
</script>
```

---

### `style.css`

```css
/* Список компаний */
.company-list {
  margin: 30px 0;
}

/* Блок компании */
.company-group {
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
}

/* Заголовок компании */
.company-header {
  background: #78ccfd;
  color: #fff;
  padding: 15px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  position: relative;
}

/* Плюс/минус */
.company-header .toggle-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 20px;
}

/* Контейнер подразделений (по умолчанию скрыт) */
.subdivisions {
  display: none;
  padding: 15px;
  background: #f9f9f9;
}

/* Вёрстка подразделений */
.subdivisions.row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

/* Одна карточка подразделения */
.subdivisions .item {
  display: block;
  width: 268px;
  margin-bottom: 26px;
  margin-right: 26px;
  background: #fff;
  padding: 15px;
  box-sizing: border-box;
  text-decoration: none;
  color: inherit;
}

.subdivisions .item:nth-child(4n) {
  margin-right: 0;
}

/* Иконка */
.subdivisions .item .icon {
  height: 75px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  margin-bottom: 10px;
}

/* Заголовок подразделения */
.subdivisions .item .title {
  margin: 10px 0;
  font-size: 20px;
  font-weight: 600;
}

/* Описание подразделения */
.subdivisions .item .text {
  font-size: 16px;
  line-height: 1.4;
  color: #555;
  height: 160px;
  overflow: hidden;
}
```

---

**Что получится**

* Вы увидите список компаний — только их заголовки, каждая с “+” справа.
* При клике на заголовок компания разворачивается (подразделения плавно появляются) и “+” меняется на “–”.
* Подразделения показываются в знакомом вам стиле карточек.
