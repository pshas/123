<?$APPLICATION->IncludeComponent(
    "company:disk.manualdrive",
    "",
    [
        "STORAGE_MODE" => "COMMON",          // тип хранилища (общий диск)
        "COMMON_CODE"  => "shared_files_s1", // код общего диска
        "FOLDER_PATH"  => "/Проекты/2025",   // путь к нужной папке
        "CREATE_IF_NOT_EXISTS" => "Y",       // создавать путь при отсутствии
        "ALLOW_UPLOAD" => "Y",               // разрешить загрузку
        "ALLOW_CREATE_FOLDER" => "Y"         // разрешить создание подпапок
    ]
);?>
