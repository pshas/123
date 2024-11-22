В **Битрикс D7** создание группы выполняется через использование модулей платформы. В этом случае вы работаете непосредственно с API ядра Битрикса, что предоставляет больше возможностей для интеграции с другими модулями.

---

### Пример кода для создания группы на D7

```php
<?php

use Bitrix\Main\Loader;
use Bitrix\Socialnetwork\WorkgroupTable;

require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

if (!Loader::includeModule('socialnetwork')) {
    die('Модуль "Социальные сети" не установлен.');
}

try {
    // Параметры группы
    $groupData = [
        'NAME' => 'Название группы',
        'DESCRIPTION' => 'Описание группы',
        'VISIBLE' => 'Y', // Видимость: Y - публичная, N - приватная
        'OPENED' => 'Y', // Открытая группа
        'CLOSED' => 'N', // Закрытая группа
        'INITIATE_PERMS' => 'K', // Кто может приглашать: K - все члены, E - только владельцы
        'PROJECT' => 'N', // Проект: Y - проект, N - нет
        'OWNER_ID' => 1, // ID владельца группы (ID пользователя)
        'SUBJECT_ID' => 1, // ID темы группы
        'SITE_ID' => SITE_ID, // ID сайта (обычно 's1')
    ];

    // Создание группы
    $result = WorkgroupTable::add($groupData);

    if ($result->isSuccess()) {
        echo "Группа успешно создана! ID группы: " . $result->getId();
    } else {
        echo "Ошибка создания группы: " . implode(', ', $result->getErrorMessages());
    }
} catch (\Exception $e) {
    echo "Произошла ошибка: " . $e->getMessage();
}
```

---

### Пояснения:

1. **Подключение модулей:**
   - Для работы с группами через D7 необходимо подключить модуль `socialnetwork`.

2. **Поля:**
   - `NAME`: Название группы.
   - `DESCRIPTION`: Описание группы.
   - `VISIBLE`: Видимость группы на сайте.
   - `OPENED`: Указывает, открыта ли группа для всех.
   - `CLOSED`: Флаг закрытой группы.
   - `INITIATE_PERMS`: Права на приглашение (например, "K" — все участники).
   - `OWNER_ID`: Владелец группы (ID пользователя из таблицы пользователей).
   - `SUBJECT_ID`: ID темы группы (получите через `Bitrix\Socialnetwork\WorkgroupSubjectTable`).
   - `SITE_ID`: ID сайта, на котором создается группа.

3. **Тема группы (`SUBJECT_ID`):**
   Чтобы получить список доступных тем, выполните следующий запрос:

   ```php
   use Bitrix\Socialnetwork\WorkgroupSubjectTable;

   $subjects = WorkgroupSubjectTable::getList([
       'select' => ['ID', 'NAME'],
   ]);

   while ($subject = $subjects->fetch()) {
       echo "ID: {$subject['ID']} - Название: {$subject['NAME']}" . PHP_EOL;
   }
   ```

4. **Обработка ошибок:**
   - Если возникают ошибки, они будут возвращены в массиве `$result->getErrorMessages()`.

---

### Расширение функционала:
После создания группы вы можете добавить участников или настроить дополнительные параметры:
- Добавление участников: через API `Bitrix\Socialnetwork\UserToGroupTable`.
- Настройка прав: используя методы для работы с `CSocNetFeatures` и `CSocNetFeaturesPerms`.

Если вам нужно дополнить код конкретной логикой, напишите!
