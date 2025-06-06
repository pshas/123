Да, без проблем. Всё, что сейчас сделано “вручную” через index.php + load\_blocks.php + add\_block.php и т. д., можно перенести на стандартные механизмы Битрикс-24 (D7): инфоблоки, компоненты и AJAX-обработчики. Ниже по шагам покажу, как именно перевести «контейнер блоков» и кнопку «Оформить» на битриксовые инструменты.

---

## 1. Создаём инфоблок для «блоков»

### 1.1. В админке

1. Заходим в «Контент → Инфоблоки → Типы инфоблоков» и создаём новый тип, например `lab` (код `lab`).
2. Внутри этого типа создаём инфоблок «Блоки лаборатории» (код, скажем, `lab_blocks`).

### 1.2. Свойства инфоблока

Внутри «Блоки лаборатории» заводим свойства, которые у вас в блоках сейчас есть. Например:

* **Название** (NAME, строка) – у каждого элемента есть заголовок.
* **Описание** (Prop: `DESCRIPTION`, тип `Справочник` или `HTML/text`) – подробное описание.
* **Цена / Статус / Дата** (если у вас есть какие-то доп. поля).
* **Файл конфигурации** или **Шаблон** (если нужно хранить JSON/PHP-шаблон): тип «Файл» (UserType `File`).

После этого каждый «блок» на вашем старом сайте будет одним элементом этого инфоблока.

---

## 2. Заменяем ручную форму «Добавить лабораторию» на компонент

Сейчас в `/local/glab/index.php` у вас стоит простая HTML-форма, которая через POST идёт в add\_block.php. Мы заменим это на стандартный битриксовый компонент – либо `bitrix:iblock.element.add.form`, либо свой D7-компонент.

### 2.1. Подключаем компонент «Форма добавления элемента инфоблока»

В том месте, где сейчас у вас:

```html
<h2>Добавить лабораторию</h2>
<form method="POST" action="add_block.php">
  <label>Название</label>
  <input name="NAME" type="text" />
  <label>Описание</label>
  <textarea name="DESCRIPTION"></textarea>
  <button type="submit">Сохранить</button>
  <button type="button">Отмена</button>
</form>
```

заменим на:

```php
<?php
// index.php (или любая другая точка входа в вашем разделе /local/glab/)
require_once $_SERVER['DOCUMENT_ROOT'].'/header.php';
$APPLICATION->SetTitle("Добавить лабораторию");
?>

<?php
$APPLICATION->IncludeComponent(
  "bitrix:iblock.element.add.form",
  "",
  [
    "IBLOCK_TYPE"           => "lab",           // указываем тип инфоблока
    "IBLOCK_ID"             => "lab_blocks",    // ID инфоблока (например, 10)
    "ELEMENT_ID"            => "",
    "ELEMENT_CODE"          => "",
    "USE_CAPTCHA"           => "N",
    "USER_MESSAGE_ADD"      => "Новая лаборатория успешно добавлена.",
    "USER_MESSAGE_EDIT"     => "Данные обновлены",
    "DEFAULT_INPUT_SIZE"    => "30",
    "PROPERTY_CODES"        => [ "NAME", "PROPERTY_DESCRIPTION" ], 
      // кодовые (internal) имена свойств:
      // NAME – заголовок, PROPERTY_DESCRIPTION – ваш описательный текст
    "PROPERTY_CODES_REQUIRED" => [ "NAME", "PROPERTY_DESCRIPTION" ],
    "RESIZE_IMAGES"         => "N",
    "MAX_FILE_SIZE"         => "0",
    "MAX_LEVELS"            => "100000",
    "LEVEL_LAST"            => "Y",
    "SEF_MODE"              => "N",
    "SEF_RULE"              => "",
    "CUSTOM_TITLE_NAME"     => "Название лаборатории",
    "CUSTOM_TITLE_PROPERTY_DESCRIPTION" => "Описание лаборатории",
    "AJAX_MODE"             => "Y",
    "AJAX_OPTION_JUMP"      => "N",
    "AJAX_OPTION_STYLE"     => "Y",
    "AJAX_OPTION_HISTORY"   => "N"
  ]
);
?>

<?php require_once $_SERVER['DOCUMENT_ROOT'].'/footer.php'; ?>
```

**Что даёт этот компонент:**

* Он сам сформирует HTML-форму, в которой будут поля Название и Описание (как кодовые `PROPERTY_DESCRIPTION`).
* При отправке он добавит новый элемент в инфоблок «Блоки лаборатории» и покажет сообщение `USER_MESSAGE_ADD`.
* Вы можете сразу включить AJAX\_MODE, чтобы форма отправлялась без перезагрузки.

> **Важно**:
>
> * Убедитесь, что `"IBLOCK_ID" => "lab_blocks"` реально совпадает с ID вашего инфоблока (можно подсмотреть в админке).
> * В параметре `PROPERTY_CODES` перечисляются поля в том порядке, в котором вы их хотите видеть: сначала указываем обязательные, потом необязательные.

---

## 3. Вывод списка «блоков» и кнопка «Оформить»

Сейчас у вас, скорее всего, в `load_blocks.php` выбираются все блоки `->SELECT * FROM lab_blocks` и рендерятся в какой-то контейнер, а у каждого есть кнопка «Оформить». Мы заменим это на стандартный компонент `bitrix:news.list` или свой D7-компонент, который будет отрисовывать список элементов инфоблока.

### 3.1. Подключаем `bitrix:news.list`

В месте, где вы хотите вывести все блоки (например, это страница `/local/glab/index.php` под формой или отдельная `/local/glab/blocks.php`), вставьте:

```php
<?php
// Если вы находитесь в том же index.php под формой, то header уже подключён. 
$APPLICATION->IncludeComponent(
  "bitrix:news.list",
  "lab_blocks_template",  // имя вашего шаблона, который создадим
  [
    "IBLOCK_TYPE"            => "lab",
    "IBLOCK_ID"              => "lab_blocks",
    "NEWS_COUNT"             => "50",
    "SORT_BY1"               => "ID",
    "SORT_ORDER1"            => "DESC",
    "FIELD_CODE"             => [ "ID", "NAME", "PREVIEW_TEXT" ],
    "PROPERTY_CODE"          => [ /* здесь, если нужны какие-то доп. свойства */ ],
    "CHECK_DATES"            => "Y",
    "DETAIL_URL"             => "", 
    "AJAX_MODE"              => "Y",
    "AJAX_OPTION_JUMP"       => "N",
    "AJAX_OPTION_STYLE"      => "Y",
    "AJAX_OPTION_HISTORY"    => "N",
    "CACHE_TYPE"             => "A",
    "CACHE_TIME"             => "3600",
    "CACHE_FILTER"           => "N",
    "CACHE_GROUPS"           => "Y",
    "PREVIEW_TRUNCATE_LEN"   => "",
    "ACTIVE_DATE_FORMAT"     => "d.m.Y",
    "SET_TITLE"              => "N",
    "SET_BROWSER_TITLE"      => "N",
    "SET_META_KEYWORDS"      => "N",
    "SET_META_DESCRIPTION"   => "N",
    "SET_LAST_MODIFIED"      => "N",
    "INCLUDE_IBLOCK_INTO_CHAIN" => "N",
    "ADD_SECTIONS_CHAIN"     => "N",
    "HIDE_LINK_WHEN_NO_DETAIL"=> "N"
  ]
);
?>
```

Это выведет упрощённый HTML-список элементов. Но вам нужно, чтобы у каждого блока была кнопка «Оформить». Значит дальше вы должны создать собственный шаблон компонента (`lab_blocks_template`), где в цикле по `$arResult["ITEMS"]` напишите что-то вроде:

```php
<?php if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die(); ?>

<div class="lab-blocks-list">
  <?php foreach($arResult["ITEMS"] as $item): ?>
    <div class="lab-block">
      <h3><?=htmlspecialcharsbx($item["NAME"])?></h3>
      <p><?=htmlspecialcharsbx($item["PREVIEW_TEXT"])?></p>
      <!-- Любая разметка, поля из $item["PROPERTIES"] и т.д. -->
      <form method="POST" action="/local/glab/order.php">
        <input type="hidden" name="BLOCK_ID" value="<?= (int)$item["ID"] ?>" />
        <button type="submit">Оформить</button>
      </form>
    </div>
  <?php endforeach; ?>
</div>
```

* Здесь мы берём `$item["ID"]`, показываем название и текст.
* Дописываем форму с кнопкой «Оформить», которая отправляется на `order.php` (или куда угодно).
* В `order.php` уже будет ваша логика оформления: сохранить заявку, отправить письмо, поменять статус и т. д.

### 3.2. Как сделать AJAX-подгрузку (как в load\_blocks.php)

Если раньше вы делали `load_blocks.php`, который, при POST-запросе, возвращал фрагмент HTML, — то сейчас вы можете:

1. Либо оставить `load_blocks.php` на стороне PHP (но вместо прямого SQL/цикла использовать D7-ORM: `CIBlockElement::GetList` или `\Bitrix\Iblock\ElementTable::getList`).
2. Либо полностью переложить на AJAX-режим компонента: в настройках `bitrix:news.list` включить `AJAX_MODE => "Y"`, и тогда компонент автоматически подгрузит список при переходе по страницам, фильтрации и т. д.
3. Если нужна своя точка входа AJAX (POST), можете создать `ajax/get_blocks.php`, внутри которого написать что-то вроде:

   ```php
   <?php
   require_once($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
   CModule::IncludeModule("iblock");
   $arFilter = [ "IBLOCK_ID" => 10, "ACTIVE" => "Y" ];
   $rs = CIBlockElement::GetList(
     [ "ID" => "DESC" ],
     $arFilter,
     false,
     false,
     [ "ID", "NAME", "PREVIEW_TEXT" ]
   );
   while($el = $rs->GetNext()) {
     echo '<div class="lab-block">';
     echo '<h3>'.htmlspecialcharsbx($el["NAME"]).'</h3>';
     echo '<p>'.htmlspecialcharsbx($el["PREVIEW_TEXT"]).'</p>';
     echo '<button data-id="'.(int)$el["ID"].'">Оформить</button>';
     echo '</div>';
   }
   ?>
   ```

   А на фронтенде (в JS) делать AJAX на `/local/glab/ajax/get_blocks.php`, вставлять ответ внутрь нужного контейнера `.lab-blocks-list`.

---

## 4. Оформление («order.php»)

Сейчас у вас, вероятно, `add_block.php`, `remove_block.php` и т. д. В Битриксе логично вынести «оформление» заявки в свой компонент или в отдельный PHP-скрипт, подключающий `prolog_before.php` (чтобы можно было использовать API D7).

Например, создаём файл `/local/glab/order.php`:

```php
<?php
// Подключаем ядро, чтобы использовать CModule и CIBlockElement
require_once($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");
CModule::IncludeModule("iblock");

// получаем ID блока из POST
$arResult = [ "SUCCESS" => false, "ERROR" => "" ];
if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST["BLOCK_ID"])) {
    $blockId = (int)$_POST["BLOCK_ID"];
    // Допустим, элемент этого инфоблока – «блок», и мы сохраняем новый элемент в инфоблок "Заявки"
    $newOrder = [
        "IBLOCK_ID"   => 11, // инфоблок «Заявки» (создайте его заранее)
        "NAME"        => "Заявка на блок ".$blockId,
        "PROPERTY_VALUES" => [
             "BLOCK_REF" => $blockId,
             "USER_ID"   => $USER->GetID(),
             // любые другие свойства заявки
        ],
        "ACTIVE"      => "Y"
    ];
    $el = new CIBlockElement;
    if ($orderId = $el->Add($newOrder)) {
        $arResult["SUCCESS"] = true;
        $arResult["ORDER_ID"] = $orderId;
    } else {
        $arResult["ERROR"] = $el->LAST_ERROR;
    }
}

// Если AJAX-запрос, отдадим JSON
if (defined("BX_UTF")) {
    header('Content-Type: application/json; charset=UTF-8');
} else {
    header('Content-Type: application/json');
}
echo \Bitrix\Main\Web\Json::encode($arResult);
```

* Здесь мы принимаем `BLOCK_ID` и создаём элемент нового инфоблока «Заявки» (IBLOCK\_ID = 11).
* Возвращаем JSON, чтобы JS на фронте видел результат операции.

На стороне JS достаточно сделать:

```js
BX.ajax.post(
  '/local/glab/order.php',
  { BLOCK_ID: blockId },
  function(response) {
    var data = JSON.parse(response);
    if (data.SUCCESS) {
      alert("Заявка #" + data.ORDER_ID + " успешно создана");
    } else {
      alert("Ошибка: " + data.ERROR);
    }
  }
);
```

---

## 5. Итоговая схема

1. **Инфоблок «Блоки лаборатории»** (lab\_blocks) хранит все ваши «контейнеры блоков».
2. **Компонент `bitrix:iblock.element.add.form`** на `/local/glab/index.php` рисует форму «Добавить блок».
3. **Компонент `bitrix:news.list`** (или кастомный D7-компонент) рисует список уже добавленных блоков, каждый с кнопкой «Оформить».
4. **Скрипт `/local/glab/order.php`** обрабатывает нажатие «Оформить» и создаёт элемент в инфоблоке «Заявки».
5. **AJAX-обновление** списка блоков (если нужно) можно реализовать либо через AJAX\_MODE у компонента `news.list`, либо через свой `ajax/get_blocks.php`.

Таким образом вы замените все свои «ручные» POST-скрипты (`add_block.php`, `load_blocks.php`, `remove_block.php` и т. п.) на штатные инфоблок-компоненты и D7-ORM. Вся логика хранения и вывода будет идти через API Битрикс, и одновременно у вас останутся привычные «кнопки» и «блоки» (вёрстка из вашего шаблона).
