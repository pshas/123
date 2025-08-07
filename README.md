1. В файле вывода заявок (`orders.php`), замените фрагмент с ФИО на ссылку на новую страницу профиля:

   ```diff
   <!-- Вместо -->
   - <td><?= htmlspecialcharsbx($el['PROPERTY_FULL_NAME_VALUE']) ?></td>
   <!-- Сделайте так -->
   <td>
     <?php
       $uid = (int)$el['PROPERTY_USER_ID_VALUE'];
       $fio = htmlspecialcharsbx($el['PROPERTY_FULL_NAME_VALUE']);
       $url = "/local/glab/user.php?user_id={$uid}";
     ?>
     <a href="<?= $url ?>"><?= $fio ?></a>
   </td>
   ```

2. Создайте страницу `/local/glab/user.php`, куда будет передаваться `user_id`, и в ней подключите профиль пользователя через готовый компонент:

   ```php
   <?php
   // /local/glab/user.php
   require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/header.php';
   use Bitrix\Main\Loader;

   global $USER;
   Loader::includeModule('intranet'); // или 'main', если нужно
   $userId = isset($_GET['user_id']) ? (int)$_GET['user_id'] : 0;
   if ($userId <= 0) {
       ShowError("Пользователь не найден");
       require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
       exit;
   }

   // Устанавливаем заголовок
   $res = CUser::GetByID($userId);
   if ($arUser = $res->Fetch()) {
       $APPLICATION->SetTitle($arUser['LAST_NAME'].' '.$arUser['NAME']);
   } else {
       ShowError("Пользователь не найден");
       require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
       exit;
   }

   // Подключаем компонент просмотра профиля
   $APPLICATION->IncludeComponent(
       "bitrix:intranet.user.profile",
       "",
       [
           "USER_PROPERTY"      => [],           // вывод всех свойств
           "USER_PROPERTY_NAME" => "",           
           "PATH_TO_USER"       => "",           
           "AJAX_MODE"          => "N",
           "SET_TITLE"          => "N",
           "USER_VAR"           => "user_id",
           "USER_ID"            => $userId,
       ]
   );

   require_once $_SERVER['DOCUMENT_ROOT'].'/bitrix/footer.php';
   ```

3. Проверьте, чтобы у вас был модуль «Интранет» (для `bitrix:intranet.user.profile`) или замените на `bitrix:main.profile.view`, если используете стандартный:

   ```php
   $APPLICATION->IncludeComponent(
     "bitrix:main.profile.view",
     "",
     ["USER_ID"=>$userId,"PATH_TO_LIST"=>"/local/glab/application/orders.php","SET_TITLE"=>"N"],
     false
   );
   ```

Теперь в таблице «Мои заявки» ФИО — это ссылка на страницу `/local/glab/user.php?user_id=…`, где компонент автоматически выведет всю доступную информацию о пользователе из Битрикс (имя, e-mail, отдел, телефон, фото и прочее).
