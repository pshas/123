В вашем шаблоне `header.php` (в `/bitrix/templates/lab/header.php`) сделайте две вещи:

1. **Выведите ФИО авторизованного пользователя как ссылку** на страницу профиля.
2. **Создайте файл** `/local/glab/application/profile.php`, который по `user_id` покажет данные сотрудника и его заявки.

---

## 1. Правим `header.php`

Найдите в начале файла, сразу после `<?php if(!defined("B_PROLOG_INCLUDED") …)` блок, где выводится админ-панель и лого, и вставьте туда:

```diff
  <?php $APPLICATION->ShowPanel(); ?>
  <header class="site-header">
    <div class="wrapper">
      <div class="logo">
        <a href="/local/glab/">GLab</a>
      </div>

+     <?php
+     global $USER;
+     if($USER->IsAuthorized()):
+         // Ссылка на профиль
+         $uid = $USER->GetID();
+         $fio = htmlspecialcharsbx($USER->GetFullName());
+     ?>
+       <nav id="user-nav">
+         <a href="/local/glab/application/profile.php?user_id=<?= $uid ?>"
+            class="user-link"><?= $fio ?></a>
+       </nav>
+     <?php endif; ?>

      <?php
      $APPLICATION->IncludeComponent(
        "bitrix:menu","top",[/*…*/],false
      );
      ?>
    </div>
  </header>
```

Добавьте в ваш CSS (рядом со стилями меню) например:

```css
#user-nav {
  display: inline-block;
  margin-left: 20px;
}
.user-link {
  color: #fff;
  text-decoration: none;
  padding: 5px 10px;
  transition: background .2s;
}
.user-link:hover {
  background: rgba(255,255,255,0.1);
}
```

---

## 2. Создаём `profile.php`

Создайте файл `/local/glab/application/profile.php`:

```php
<?php
// /local/glab/application/profile.php
require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php';
use Bitrix\Main\Loader;
global $USER;

// 1) Получаем user_id из GET
$profileId = (int)($_GET['user_id'] ?? 0);
if($profileId <= 0) {
    ShowError("Не указан сотрудник");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}

// 2) Проверяем права: лаборанты (группа 8) видят любого, остальные — только свой
Loader::includeModule('main');
$groups = $USER->GetUserGroupArray();
if(!in_array(8, $groups, true) && $USER->GetID() !== $profileId) {
    ShowError("Доступ запрещён");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}

// 3) Получаем данные пользователя
$rsUser = CUser::GetByID($profileId);
$arUser = $rsUser->Fetch();
if(!$arUser) {
    ShowError("Сотрудник не найден");
    require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
    exit;
}

// 4) Устанавливаем заголовок
$APPLICATION->SetTitle("Профиль: ".$arUser['NAME'].' '.$arUser['LAST_NAME']);

?>
<h1><?= $APPLICATION->GetTitle() ?></h1>

<table>
  <tr><th>ФИО</th><td><?= htmlspecialcharsbx($arUser['NAME'].' '.$arUser['LAST_NAME']) ?></td></tr>
  <tr><th>Логин</th><td><?= htmlspecialcharsbx($arUser['LOGIN']) ?></td></tr>
  <tr><th>Отдел</th><td><?= htmlspecialcharsbx($arUser['WORK_DEPARTMENT']) ?></td></tr>
  <tr><th>Должность</th><td><?= htmlspecialcharsbx($arUser['WORK_POSITION']) ?></td></tr>
  <tr><th>Телефон</th><td><?= htmlspecialcharsbx($arUser['PERSONAL_PHONE']) ?></td></tr>
  <!-- сюда при необходимости другие поля профиля -->
</table>

<hr>

<h2>Заявки сотрудника</h2>
<?php
// 5) Подключаем iblock и выводим его заявки
Loader::includeModule('iblock');
$iblockCode = 'lab_orders';
$rsI = CIBlock::GetList([], ['CODE'=>$iblockCode,'ACTIVE'=>'Y']);
$ib = $rsI->Fetch();
$iblockId = (int)$ib['ID'];

// Получаем заявки по свойству USER_ID
$rs = CIBlockElement::GetList(
    ['DATE_CREATE'=>'DESC'],
    [
      'IBLOCK_ID'=> $iblockId,
      'ACTIVE'=>'Y',
      'PROPERTY_USER_ID' => $profileId
    ],
    false,false,
    ['ID','NAME','DATE_CREATE','PROPERTY_STATUS']
);

if($rs->SelectedRowsCount()==0) {
    echo "<p>Заявок нет.</p>";
} else {
    echo '<table border="1" cellpadding="5"><thead>
      <tr><th>ID</th><th>Дата</th><th>Статус</th><th>Просмотр</th></tr>
    </thead><tbody>';
    while($el = $rs->Fetch()) {
        $view = "/local/glab/application/index.php?block_id=".$el['IBLOCK_SECTION_ID']
              ."&order_id=".$el['ID'];
        echo "<tr>
          <td>{$el['ID']}</td>
          <td>{$el['DATE_CREATE']}</td>
          <td>".htmlspecialcharsbx($el['PROPERTY_STATUS_VALUE'])."</td>
          <td><a href='{$view}'>Открыть</a></td>
        </tr>";
    }
    echo '</tbody></table>';
}
?>

<?php
require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
?>
```

---

**Что получится:**

* В шапке у авторизованного пользователя вместо обычного меню появится его ФИО — ссылка на `/profile.php?user_id=…`.
* На странице профиля отображаются данные сотрудника и список всех заявок, где свойство `USER_ID` равно этому `user_id`.
* Доступ проверяется:

  * **Лаборанты** (группа 8) смотрят **любого** пользователя;
  * **Обычные** — только **свои** заявки.
  * 
