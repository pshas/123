Давайте добавим функциональность для добавления выбранного пользователя в группу пользователей прямо из модального окна. Мы обновим код, чтобы включить кнопку для добавления пользователя и обработчик, который выполнит эту операцию на сервере.

### Полный пример с добавлением пользователя в группу

#### 1. HTML для модального окна

Добавим поле для выбора группы и кнопку для добавления пользователя в группу:

```html
<!-- Кнопка для открытия модального окна -->
<button id="openModal">Добавить пользователя в группу</button>

<!-- Модальное окно -->
<div id="userModal" style="display:none;">
    <div>
        <h2>Добавление пользователя в группу</h2>
        <label for="user_search">Пользователь:</label>
        <input type="text" id="user_search" name="user_search" autocomplete="off" placeholder="Введите имя пользователя...">
        <ul id="user_suggestions" style="display:none;"></ul> <!-- Список автоподсказок -->

        <label for="user_group">Группа:</label>
        <select id="user_group">
            <option value="">Выберите группу</option>
            <!-- Группы будут загружены с сервера -->
        </select>

        <button id="addUserButton">Добавить</button>
        <button id="closeModal">Закрыть</button>
    </div>
</div>
```

#### 2. JavaScript для работы с модальным окном и добавлением пользователя

Теперь добавим функциональность для обработки клика на кнопку "Добавить", чтобы отправить данные на сервер и добавить пользователя в группу:

```javascript
BX.ready(function() {
    // Открытие модального окна
    document.getElementById('openModal').onclick = function() {
        document.getElementById('userModal').style.display = 'block';
        loadUserGroups(); // Загружаем группы при открытии модального окна
    };

    // Закрытие модального окна
    document.getElementById('closeModal').onclick = function() {
        document.getElementById('userModal').style.display = 'none';
    };

    // Поиск пользователей
    document.getElementById('user_search').oninput = function() {
        const query = this.value.trim();

        if (query.length < 2) {
            document.getElementById('user_suggestions').style.display = 'none';
            return;
        }

        // AJAX-запрос для получения списка пользователей
        BX.ajax({
            url: '/local/ajax/search_users.php', // Ваш обработчик, который ищет пользователей
            method: 'POST',
            dataType: 'json',
            data: {
                query: query,
                sessid: BX.bitrix_sessid() // Обязательно передаем сессионный идентификатор
            },
            onsuccess: function(data) {
                let suggestions = document.getElementById('user_suggestions');
                suggestions.innerHTML = ''; // Очищаем предыдущие подсказки

                if (data.length > 0) {
                    data.forEach(function(user) {
                        let li = document.createElement('li');
                        li.textContent = user.NAME + ' (' + user.EMAIL + ')'; // Отображаем имя и почту пользователя
                        li.dataset.userId = user.ID;

                        // При выборе пользователя
                        li.onclick = function() {
                            document.getElementById('user_search').value = user.NAME; // Заполняем поле ввода
                            suggestions.style.display = 'none'; // Скрываем подсказки
                        };

                        suggestions.appendChild(li);
                    });

                    suggestions.style.display = 'block'; // Отображаем список подсказок
                } else {
                    suggestions.style.display = 'none'; // Скрываем, если данных нет
                }
            },
            onfailure: function(error) {
                console.error('Ошибка при получении данных', error);
            }
        });
    };

    // Функция для загрузки групп пользователей
    function loadUserGroups() {
        BX.ajax({
            url: '/local/ajax/get_user_groups.php', // Ваш обработчик для получения групп
            method: 'POST',
            dataType: 'json',
            data: {
                sessid: BX.bitrix_sessid() // Обязательно передаем сессионный идентификатор
            },
            onsuccess: function(data) {
                let userGroupSelect = document.getElementById('user_group');
                userGroupSelect.innerHTML = '<option value="">Выберите группу</option>'; // Сбрасываем предыдущие группы

                data.forEach(function(group) {
                    let option = document.createElement('option');
                    option.value = group.ID;
                    option.textContent = group.NAME;
                    userGroupSelect.appendChild(option);
                });
            },
            onfailure: function(error) {
                console.error('Ошибка при получении групп', error);
            }
        });
    }

    // Обработка нажатия кнопки "Добавить"
    document.getElementById('addUserButton').onclick = function() {
        const userName = document.getElementById('user_search').value;
        const userGroupId = document.getElementById('user_group').value;

        if (!userName || !userGroupId) {
            alert('Пожалуйста, выберите пользователя и группу.');
            return;
        }

        // AJAX-запрос для добавления пользователя в группу
        BX.ajax({
            url: '/local/ajax/add_user_to_group.php', // Ваш обработчик для добавления пользователя
            method: 'POST',
            dataType: 'json',
            data: {
                userName: userName,
                userGroupId: userGroupId,
                sessid: BX.bitrix_sessid() // Обязательно передаем сессионный идентификатор
            },
            onsuccess: function(data) {
                if (data.success) {
                    alert('Пользователь успешно добавлен в группу!');
                    document.getElementById('userModal').style.display = 'none'; // Закрываем модальное окно
                } else {
                    alert('Ошибка: ' + data.error);
                }
            },
            onfailure: function(error) {
                console.error('Ошибка при добавлении пользователя', error);
            }
        });
    };
});
```

#### 3. PHP обработчик для получения групп (`get_user_groups.php`)

Создайте файл `get_user_groups.php`, который будет возвращать список групп пользователей:

```php
<?php
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

// Проверка сессии
if (!check_bitrix_sessid()) {
    echo json_encode([]);
    die();
}

if (!$USER->IsAuthorized()) {
    echo json_encode([]);
    die();
}

// Подключение модуля
if (!CModule::IncludeModule('socialservices')) {
    echo json_encode([]);
    die();
}

// Получение групп пользователей
$arGroups = [];
$dbGroups = CGroup::GetList($by = "c_sort", $order = "asc", ['ACTIVE' => 'Y']);
while ($arGroup = $dbGroups->Fetch()) {
    $arGroups[] = [
        'ID' => $arGroup['ID'],
        'NAME' => $arGroup['NAME'],
    ];
}

echo json_encode($arGroups);
?>
```

#### 4. PHP обработчик для добавления пользователя в группу (`add_user_to_group.php`)

Создайте файл `add_user_to_group.php`, который будет обрабатывать добавление пользователя в группу:

```php
<?php
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

// Проверка сессии
if (!check_bitrix_sessid()) {
    echo json_encode(['success' => false, 'error' => 'Session expired']);
    die();
}

if (!$USER->IsAuthorized()) {
    echo json_encode(['success' => false, 'error' => 'User not authorized']);
    die();
}

// Подключение нужных модулей
if (!CModule::IncludeModule('main')) {
    echo json_encode(['success' => false, 'error' => 'Unable to include main module']);
    die();
}

$userName = htmlspecialcharsbx($_POST['userName']);
$userGroupId = intval($_POST['userGroupId']);

// Получаем ID пользователя по имени
$userFilter = ['NAME' => $userName, 'ACTIVE' => 'Y'];
$rsUsers = CUser::GetList(($by = "NAME"), ($order = "ASC"), $userFilter, ['FIELDS' => ['ID']]);
if ($arUser = $rsUsers->Fetch()) {
    // Добавляем пользователя в группу
    $userId = $arUser['ID'];
    $user = new CUser;
    $user->Update($userId, ['GROUP_ID' => array_merge((array)$user->GetUserGroup($userId), [$userGroupId])]);

    echo json_encode(['success' => true]);
} else {
    echo json_encode(['success' => false, 'error' => 'User not found']);
}
?>
```

### Заключение

Теперь модальное окно позволяет:

1. **Выбрать пользователя** из списка с автоподстановкой.
2. **Выбрать группу**, в которую будет добавлен пользователь.
3. **Добавить пользователя в группу** при нажатии на кнопку "Добавить".

При успешном добавлении пользователь получит сообщение об успешном добавлении, и модальное окно закроется. Если возникнет ошибка, отобразится соответствующее сообщение.

Если у вас есть дополнительные вопросы или требуется доработка, пожалуйста, дайте знать!
