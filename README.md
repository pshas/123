В Битриксе для управления **группами пользователей** используется класс `CGroup`. Этот класс позволяет создавать, обновлять и удалять группы пользователей через API. 

---

### Пример добавления группы пользователей с использованием `CGroup`

```php
<?php

require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

if (!$USER->IsAdmin()) {
    die("Доступ запрещен");
}

// Подключение класса
use Bitrix\Main\Loader;

Loader::includeModule("main");

try {
    // Создаем объект класса CGroup
    $group = new CGroup;

    // Параметры новой группы
    $groupData = [
        "NAME" => "Название группы", // Название группы
        "DESCRIPTION" => "Описание группы", // Описание группы
        "ACTIVE" => "Y", // Активность: Y - активна, N - не активна
        "STRING_ID" => "custom_group_code", // Символьный идентификатор группы
    ];

    // Проверяем, есть ли уже такая группа
    $existingGroup = CGroup::GetList($by = "id", $order = "asc", ["STRING_ID" => $groupData["STRING_ID"]])->Fetch();

    if ($existingGroup) {
        echo "Группа с кодом '{$groupData["STRING_ID"]}' уже существует (ID: {$existingGroup["ID"]}).";
    } else {
        // Добавляем группу
        $newGroupId = $group->Add($groupData);

        if ($newGroupId > 0) {
            echo "Группа успешно создана! ID новой группы: " . $newGroupId;
        } else {
            echo "Ошибка создания группы: " . $group->LAST_ERROR;
        }
    }
} catch (\Exception $e) {
    echo "Произошла ошибка: " . $e->getMessage();
}
```

---

### Пояснение:

1. **Класс `CGroup`:**
   - Класс `CGroup` — стандартный инструмент для работы с группами пользователей.
   - Основные методы:
     - `Add($fields)`: добавляет новую группу.
     - `Update($id, $fields)`: обновляет данные группы.
     - `Delete($id)`: удаляет группу.

2. **Поля для создания группы (`$groupData`):**
   - `NAME`: Название группы.
   - `DESCRIPTION`: Описание группы.
   - `ACTIVE`: Статус активности (Y — активна, N — не активна).
   - `STRING_ID`: Символьный идентификатор группы (уникальный код для удобного поиска).

3. **Проверка существования группы:**
   - Метод `CGroup::GetList()` позволяет найти уже существующую группу по символьному идентификатору (`STRING_ID`).

---

### Дополнительно: Привязка прав доступа

После создания группы можно настроить права доступа. Например, дать права группе на доступ к модулю:

```php
$groupId = $newGroupId; // ID созданной группы
$moduleId = "main"; // Идентификатор модуля
$permission = "W"; // Права доступа (W — полный доступ)

CGroup::SetModulePermission($groupId, $moduleId, $permission);
echo "Права доступа для группы ID {$groupId} успешно установлены.";
```

---

### Получение списка всех групп:

Если нужно вывести все группы, можно использовать следующий код:

```php
$rsGroups = CGroup::GetList($by = "id", $order = "asc", ["ACTIVE" => "Y"]);
while ($group = $rsGroups->Fetch()) {
    echo "ID: {$group["ID"]}, Название: {$group["NAME"]}, Код: {$group["STRING_ID"]}" . PHP_EOL;
}
```

---

Если вам нужно выполнить что-то более сложное, например, массово добавить пользователей в группу или настраивать права для конкретных разделов, уточните!
