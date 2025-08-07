<?php
// …ваш существующий код profile.php до контента…
?>

<!-- Стили только для этой страницы -->
<style>
  /* Весь текст внутри #profile-page будет чёрным */
  #profile-page {
    color: #000 !important;
  }
  /* Если у вас есть таблицы или другие блоки — они тоже наследуют цвет */
  #profile-page table,
  #profile-page h1,
  #profile-page h2,
  #profile-page p {
    color: #000 !important;
  }
</style>

<!-- Обёртка профиля -->
<div id="profile-page">
  <h1><?= $APPLICATION->GetTitle() ?></h1>

  <table>
    <!-- …ваш контент профиля… -->
  </table>

  <hr>

  <h2>Заявки сотрудника</h2>
  <?php
  // …ваш вывод заявок…
  ?>
</div>

<?php
// …ваш существующий код footer…
?>
