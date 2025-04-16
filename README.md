$APPLICATION->IncludeComponent(
    'bitrix:disk.file.list',
    '',
    [
        'STORAGE_ID' => 123, // ID хранилища
        'FOLDER_ID' => 456,  // ID папки (0 для корня)
        'RELATIVE_PATH' => '/', // Путь внутри диска
        'SHOW_FILTER' => 'Y',  // Показать фильтры
        'SHOW_NAVIGATION' => 'Y', // Хлебные крошки
    ]
);
