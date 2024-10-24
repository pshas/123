Для того чтобы реализовать список пользователей и групп с интерактивным выбором, мы можем использовать стандартные компоненты Bitrix24, такие как BX.UI.Selector (селектор пользователей) и выбор групп через AJAX. Это улучшит UX, так как администраторы смогут легко выбирать пользователей и группы из выпадающих списков.

### 1. **Интерактивный выбор пользователей и групп**
Добавим динамические выпадающие списки для выбора пользователей и групп в модальном окне.

#### Модальное окно с выбором пользователей и групп:
```html
<!-- Кнопка для открытия модального окна -->
<button id="openModal">Добавить пользователя в группу</button>

<!-- Модальное окно -->
<div id="myModal" class="modal" style="display: none;">
    <div class="modal-content">
        <span class="close">&times;</span>
        <h2>Добавить пользователя в группу</h2>
        
        <!-- Селект для пользователей -->
        <label for="user_id">Пользователь:</label>
        <select id="user_id" name="user_id" required></select>
        
        <!-- Селект для групп -->
        <label for="group_id">Группа:</label>
        <select id="group_id" name="group_id" required></select><br><br>

        <button type="submit" id="submitForm">Добавить</button>
    </div>
</div>

<script>
document.getElementById('openModal').onclick = function() {
    document.getElementById('myModal').style.display = "block";
    loadUsers();  // Загрузка списка пользователей
    loadGroups(); // Загрузка списка групп
};

document.querySelector('.close').onclick = function() {
    document.getElementById('myModal').style.display = "none";
};

// Закрытие окна, если клик вне его
window.onclick = function(event) {
    if (event.target == document.getElementById('myModal')) {
        document.getElementById('myModal').style.display = "none";
    }
};

// Функция для загрузки списка пользователей через AJAX
function loadUsers() {
    BX.ajax({
        url: '/local/ajax/get_users.php',
        method: 'POST',
        dataType: 'json',
        onsuccess: function(data) {
            let userSelect = document.getElementById('user_id');
            userSelect.innerHTML = ''; // Очищаем предыдущие данные
            data.users.forEach(function(user) {
                let option = document.createElement('option');
                option.value = user.ID;
                option.text = user.NAME;
                userSelect.add(option);
            });
        }
    });
}

// Функция для загрузки списка групп через AJAX
function loadGroups() {
    BX.ajax({
        url: '/local/ajax/get_groups.php',
        method: 'POST',
        dataType: 'json',
        onsuccess: function(data) {
            let groupSelect = document.getElementById('group_id');
            groupSelect.innerHTML = ''; // Очищаем предыдущие данные
            data.groups.forEach(function(group) {
                let option = document.createElement('option');
                option.value = group.ID;
                option.text = group.NAME;
                groupSelect.add(option);
            });
        }
    });
}

// Обработка отправки формы
document.getElementById('submitForm').onclick = function(e) {
    e.preventDefault();
    
    const user_id = document.getElementById('user_id').value;
    const group_id = document.getElementById('group_id').value;
    
    // Вызов AJAX-запроса для добавления пользователя в группу
    BX.ajax({
        url: '/local/ajax/add_user_to_group.php',
        method: 'POST',
        data: {
            user_id: user_id,
            group_id: group_id
        },
        dataType: 'json',
        onsuccess: function(data) {
            if(data.success) {
                alert('Пользователь успешно добавлен в группу');
                document.getElementById('myModal').style.display = "none";
            } else {
                alert('Ошибка добавления пользователя: ' + data.message);
            }
        }
    });
};
</script>
```

### 2. **Создание обработчиков для получения списка пользователей и групп**

#### 2.1. Обработчик для получения списка пользователей (`/local/ajax/get_users.php`):
Используем API Bitrix для получения списка всех пользователей.

```php
<?php
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

if (!CModule::IncludeModule("main")) {
    echo json_encode(['success' => false, 'message' => 'Ошибка подключения модуля']);
    die();
}

$users = [];
$rsUsers = CUser::GetList(($by = "id"), ($order = "asc"), ['ACTIVE' => 'Y']);
while ($user = $rsUsers->Fetch()) {
    $users[] = [
        'ID' => $user['ID'],
        'NAME' => $user['NAME'] . ' ' . $user['LAST_NAME']
    ];
}

echo json_encode(['success' => true, 'users' => $users]);
?>
```

#### 2.2. Обработчик для получения списка групп (`/local/ajax/get_groups.php`):
Используем API для получения списка всех групп.

```php
<?php
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

if (!CModule::IncludeModule("socialnetwork")) {
    echo json_encode(['success' => false, 'message' => 'Ошибка подключения модуля']);
    die();
}

$groups = [];
$rsGroups = CSocNetGroup::GetList(
    ['NAME' => 'ASC'],
    ['ACTIVE' => 'Y'],
    false,
    false,
    ['ID', 'NAME']
);
while ($group = $rsGroups->Fetch()) {
    $groups[] = [
        'ID' => $group['ID'],
        'NAME' => $group['NAME']
    ];
}

echo json_encode(['success' => true, 'groups' => $groups]);
?>
```

### 3. **Добавление пользователя в группу**
Файл `/local/ajax/add_user_to_group.php` уже был приведен в предыдущем шаге. Необходимо только убедиться, что обработчики возвращают корректные данные, и все AJAX-запросы работают правильно.

### 4. **Включение AJAX и стилей в шаблон**
Не забудьте включить поддержку AJAX и стили для модального окна в шаблон:

```php
CJSCore::Init(array('ajax'));
```

### Результат
Теперь при нажатии на кнопку откроется модальное окно, в котором можно будет выбрать пользователя и группу из интерактивных выпадающих списков. Это значительно улучшит удобство использования, так как пользователи смогут быстро находить нужные группы и пользователей без ввода ID вручную.
