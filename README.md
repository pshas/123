use Bitrix\Main\Loader;
use Bitrix\Disk\Driver;
use Bitrix\Disk\Security\SecurityContext;

Loader::includeModule('disk');

// 1. Получаем хранилище (например, группы)
$groupId = 123; // ID группы
$storage = Driver::getInstance()->getStorageByGroupId($groupId);
if (!$storage) {
    die('Хранилище не найдено');
}

// 2. Получаем корневую папку
$rootFolder = $storage->getRootObject();
if (!$rootFolder) {
    die('Корневая папка не найдена');
}

// 3. Создаем контекст безопасности (для текущего пользователя)
$userId = \Bitrix\Main\Engine\CurrentUser::get()->getId();
$securityContext = $storage->getSecurityContext($userId);

// 4. Получаем содержимое папки с учетом прав
$children = $rootFolder->getChildren($securityContext); // Теперь аргумент передан

foreach ($children as $item) {
    if ($item instanceof \Bitrix\Disk\File) {
        echo 'Файл: ' . $item->getName() . '<br>';
    } elseif ($item instanceof \Bitrix\Disk\Folder) {
        echo 'Папка: ' . $item->getName() . '<br>';
    }
}
