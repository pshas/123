Понял 👍
У тебя уже есть таблица заявок, в ней в колонке «Действие» есть кнопка (например «Редактировать» или «Подробнее»). Теперь тебе нужно добавить ещё одну — **«Оформить»**, и при её нажатии менять статус заявки (например, из `NEW` → `IN_WORK` или `DONE`).

---

### 1. В index.php при выводе таблицы заявок

Допустим, сейчас у тебя вывод цикла заявок выглядит так:

```php
<?php while ($arItem = $rsItems->GetNext()): ?>
<tr>
    <td><?= $arItem['ID'] ?></td>
    <td><?= htmlspecialcharsbx($arItem['NAME']) ?></td>
    <td><?= $arItem['PROPERTY_STATUS_VALUE'] ?></td>
    <td>
        <a href="edit.php?id=<?= $arItem['ID'] ?>">Редактировать</a>
    </td>
</tr>
<?php endwhile; ?>
```

Добавим туда кнопку «Оформить»:

```php
<td>
    <a href="edit.php?id=<?= $arItem['ID'] ?>">Редактировать</a> |
    <a href="index.php?action=approve&id=<?= $arItem['ID'] ?>&<?= bitrix_sessid_get() ?>">Оформить</a>
</td>
```

---

### 2. Обработка кнопки «Оформить»

В начале `index.php` (до вывода таблицы) добавим обработчик:

```php
<?php
use Bitrix\Main\Loader;
Loader::includeModule("iblock");

if ($_GET['action'] === 'approve' 
    && check_bitrix_sessid() 
    && !empty($_GET['id'])) 
{
    $el = new CIBlockElement;
    $id = (int)$_GET['id'];

    // Меняем свойство STATUS
    $res = $el->SetPropertyValuesEx($id, $ordersIblockId, [
        'STATUS' => 'IN_WORK' // или DONE — зависит от твоей логики
    ]);

    if ($res) {
        echo '<div class="ok">Заявка №'.$id.' оформлена.</div>';
    } else {
        echo '<div class="errors">Ошибка при изменении статуса заявки.</div>';
    }
}
```

---

### 3. Что получится

В таблице заявок появится колонка «Действие» с двумя ссылками:

* «Редактировать» (как раньше),
* «Оформить» (меняет статус заявки).

После клика статус у элемента инфоблока обновится.

---

❓Уточни: тебе при «Оформить» нужно переводить заявку строго в `IN_WORK` или сразу в `DONE`?
(я могу сразу сделать универсальный обработчик — например, «Оформить» → `IN_WORK`, а «Закрыть» → `DONE`).
