<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

// Подключаем стили шаблона, если файл существует
$cssFile = $templateFolder . "/style.css";
if (file_exists($_SERVER['DOCUMENT_ROOT'] . $cssFile)) {
    Bitrix\Main\Page\Asset::getInstance()->addCss($cssFile);
}
?>

<div id="companies" class="row">
    <?php foreach ($arResult["ITEMS"] as $item):
        // Собираем ссылку на детальную или на список, либо используем корневую
        $link = $item["DETAIL_PAGE_URL"] ?: $item["LIST_PAGE_URL"] ?: "/local/glab/";
        $link = htmlspecialcharsbx($link);

        // Текстовые данные
        $name    = htmlspecialcharsbx($item["NAME"]);
        $preview = htmlspecialcharsbx($item["PREVIEW_TEXT"]);

        // Картинка: предварительно проверяем PREVIEW_PICTURE, иначе - плейсхолдер
        if (!empty($item["PREVIEW_PICTURE"])) {
            $pictureUrl = CFile::GetPath($item["PREVIEW_PICTURE"]);
        } else {
            $pictureUrl = $templateFolder . "/img/company.png";
        }
        $pictureUrl = htmlspecialcharsbx($pictureUrl);
    ?>
        <a class="item col-md" href="<?= $link ?>">
            <div class="icon" style="background-image: url('<?= $pictureUrl ?>');"></div>
            <div class="title"><?= $name ?></div>
            <div class="text"><?= $preview ?></div>
        </a>
    <?php endforeach; ?>
</div>


#companies{
  margin:30px 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

#companies .item{  
  display: block;
  text-transform: none;
  font-weight: 400;
  font-size: 16px;
  line-height: normal;  
  background: #ffffff;
  width:268px;
  margin-right: 26px;
  margin-bottom: 26px;
  overflow: hidden;
  padding:15px;
  text-decoration: none;
  color: inherit;
  box-sizing: border-box;
}

#companies .item:nth-child(4n){
  margin-right: 0;
}

#companies .item .icon{
  height:75px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  margin-bottom: 10px;
}

#companies .item .title{
  margin:10px 0;
  height:50px;    
  font-weight: 600;
  font-size: 20px;  
  color: #000000;
}

#companies .item .text{
  margin-bottom:20px;
  height:160px;  
  font-size: 18px;  
  color: #555;
  overflow: hidden;
}

