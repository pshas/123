Окей, «Оформить» не сработало, потому что `STATUS` — это свойство-список, а в `SetPropertyValuesEx` ему нужен **ID варианта** (или хотя бы `XML_ID`), а не произвольная строка. Ниже — рабочий патч для `index.php`, который:

1. добавляет ссылку «Оформить» в колонку «Действие»
2. при клике на неё находит нужный enum `STATUS` по `XML_ID` (`NEW`, `IN_WORK`, `DONE`) **или** по текстовому значению (на случай русских названий)
3. меняет статус и делает redirect по PRG-паттерну (чтобы не повторять действие при F5)

---

### 1) Обработчик наверху `index.php` (до вывода таблицы)

```php
<?php
use Bitrix\Main\Loader;

Loader::includeModule('iblock');

// Получаем ID инфоблока по коду (если у тебя уже есть $ordersIblockId — используй его)
$iblockCode = 'lab_orders';
$rsI = CIBlock::GetList([], ['CODE'=>$iblockCode,'ACTIVE'=>'Y']);
if (!$ib = $rsI->Fetch()) {
    ShowError("Инфоблок заявок не найден");
} else {
    $ordersIblockId = (int)$ib['ID'];
}

// Хелпер: получить ID варианта списка свойства по XML_ID или VALUE
function getListEnumId($iblockId, $propCode, $wanted)
{
    $wanted = (array)$wanted; // можно передать массив с синонимами
    $enumId = null;

    // 1) пробуем по XML_ID
    $by = "SORT"; $order = "ASC";
    $rs = CIBlockPropertyEnum::GetList([$by=>$order], ["IBLOCK_ID"=>$iblockId, "CODE"=>$propCode]);
    while ($e = $rs->Fetch()) {
        if ($e['XML_ID'] && in_array($e['XML_ID'], $wanted, true)) {
            $enumId = (int)$e['ID'];
            break;
        }
    }
    if ($enumId) return $enumId;

    // 2) пробуем по VALUE (регистр игнорим)
    $rs = CIBlockPropertyEnum::GetList([$by=>$order], ["IBLOCK_ID"=>$iblockId, "CODE"=>$propCode]);
    while ($e = $rs->Fetch()) {
        foreach ($wanted as $w) {
            if (mb_strtolower($e['VALUE']) === mb_strtolower($w)) {
                return (int)$e['ID'];
            }
        }
    }
    return null;
}

// Обработка действия "оформить"
if (isset($_GET['action'], $_GET['id']) && $_GET['action'] === 'approve' && check_bitrix_sessid()) {
    global $APPLICATION;

    $elementId = (int)$_GET['id'];

    // Ищем enum ID для статуса "IN_WORK"
    // Настрой: поставь у вариантов STATUS XML_ID: NEW / IN_WORK / DONE.
    // Если у тебя русские значения, мы подстрахуемся синонимами.
    $targetEnumId = getListEnumId($ordersIblockId, 'STATUS', ['IN_WORK', 'В работе', 'В обработке']);

    if ($targetEnumId) {
        CIBlockElement::SetPropertyValuesEx($elementId, $ordersIblockId, ['STATUS' => $targetEnumId]);
        // PRG-паттерн: убираем action/id/sessid из URL
        LocalRedirect($APPLICATION->GetCurPageParam('ok=approve', ['action','id','sessid']));
    } else {
        ShowError('Не найден вариант свойства STATUS для "IN_WORK". Задай XML_ID у варианта или проверь названия.');
    }
}

// Сообщение об успехе (после редиректа)
if (isset($_GET['ok']) && $_GET['ok'] === 'approve') {
    echo '<div class="ok">Статус заявки обновлён.</div>';
}
```

---

### 2) Кнопка «Оформить» в колонке «Действие» при выводе строк

(фрагмент в цикле вывода таблицы)

```php
<td class="actions">
    <a href="edit.php?id=<?= (int)$arItem['ID'] ?>">Редактировать</a>
    &nbsp;|&nbsp;
    <a href="index.php?action=approve&id=<?= (int)$arItem['ID'] ?>&<?= bitrix_sessid_get() ?>">Оформить</a>
</td>
```

> При желании можно показывать «Оформить» только для NEW. Для этого получай текущее значение `STATUS` и скрывай ссылку, если оно уже `IN_WORK`/`DONE`.

---

### Важно проверить у себя

* В инфоблоке `lab_orders` у свойства **STATUS** должны быть варианты. Рекомендую каждому варианту задать **XML\_ID**:

  * NEW
  * IN\_WORK
  * DONE

  Тогда код найдёт их по `XML_ID` без двусмысленностей. Если оставить русские значения, хелпер всё равно попробует сопоставить по `VALUE`.

* Переменная `$ordersIblockId` доступна в обработчике (вставляй блок из п.1 **после** получения этого ID или используй свой уже существующий код, где она у тебя есть).

* Ссылка добавляет `<?= bitrix_sessid_get() ?>`, поэтому `check_bitrix_sessid()` пройдёт.

---

Если хочешь, сделаю ещё кнопку «Закрыть» → `DONE` (аналогично, только меняем целевой `XML_ID`/`VALUE`).
