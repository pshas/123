<?php while ($sub = $rsSubs->Fetch()):
    $subId = (int)$sub['ID'];
    // вот здесь формируем новую ссылку
    $link = "/local/glab/application/index.php?block_id=".$subId;
?>
  <a class="item col-md" href="<?= htmlspecialcharsbx($link) ?>">
    <div class="icon" …></div>
    <div class="title"><?= htmlspecialcharsbx($sub['NAME']) ?></div>
    <div class="text"><?= nl2br(htmlspecialcharsbx($sub['DESCRIPTION'])) ?></div>
  </a>
<?php endwhile; ?>
