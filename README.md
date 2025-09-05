<?php
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");
$APPLICATION->SetTitle("Хаб проектов");

CModule::IncludeModule("iblock");

// Получаем элементы инфоблока
$arFilter = ["IBLOCK_CODE" => "hub_projects", "ACTIVE" => "Y"];
$res = CIBlockElement::GetList(["SORT" => "ASC"], $arFilter, false, false, ["ID","NAME","PROPERTY_LINK"]);
?>

<div class="hub-container">
  <h1>Хаб проектов</h1>
  <div class="hub-grid">
    <?php while($item = $res->GetNextElement()):
        $fields = $item->GetFields();
        $link = $fields["PROPERTY_LINK_VALUE"];
        $name = $fields["NAME"];
    ?>
      <a href="<?=htmlspecialchars($link)?>" class="hub-card" target="_blank">
        <?=$name?>
      </a>
    <?php endwhile; ?>
  </div>
</div>

<style>
  .hub-container {
    max-width: 900px;
    margin: 0 auto;
    text-align: center;
  }
  .hub-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px,1fr));
    gap: 20px;
    margin-top: 30px;
  }
  .hub-card {
    display: block;
    padding: 20px;
    background: #f5f5f5;
    border-radius: 12px;
    text-decoration: none;
    color: #333;
    font-weight: bold;
    transition: 0.3s;
  }
  .hub-card:hover {
    background: #0070f3;
    color: #fff;
  }
</style>

<?php
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/footer.php");
?>
