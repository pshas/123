use Bitrix\Iblock\SectionTable;
use Bitrix\Main\Type\DateTime;

// ID вашего инфоблока
$iblockId = 10;

// Данные для новой компании
$fields = [
    'IBLOCK_ID'      => $iblockId,
    'NAME'           => 'Первая компания',
    'ACTIVE'         => 'Y',
    'SORT'           => 500,
    'DESCRIPTION'    => 'Описание первой компании',
    // 'CODE'        => 'pervaya_kompaniya',
    // 'PICTURE'     => \CFile::MakeFileArray($_SERVER['DOCUMENT_ROOT'].'/local/images/logo.png'),
    // Не указываем SECTION_ID — значит корневой раздел
];

$result = SectionTable::add($fields);
if ($result->isSuccess()) {
    $newSectionId = $result->getId();
    echo "Создан раздел ID={$newSectionId}\n";
} else {
    echo "Ошибка: " . implode("; ", $result->getErrorMessages());
}
