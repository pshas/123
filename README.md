<?php
if(!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED!==true) die();
use Bitrix\Main\Page\Asset;

// Подключаем core.js через CJSCore::Init()
CJSCore::Init(array('ajax', 'popup', 'window', 'core'));
?>
<head>
    <?php
    // Вставляем заголовки сайта
    $APPLICATION->ShowHead();
    ?>
</head>

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


Для того чтобы AJAX-запросы в Bitrix работали корректно, важно подключить не только библиотеку `ajax`, но и некоторые другие библиотеки и настройки, которые обеспечат стабильную работу запросов, а также корректную работу модальных окон и элементов управления.

Вот что нужно подключить, чтобы запросы работали правильно:

### 1. **Подключение основных библиотек Bitrix**

#### a. **`ajax`** — основная библиотека для работы с AJAX-запросами в Bitrix.
#### b. **`popup`** — для работы с модальными окнами и всплывающими уведомлениями.
#### c. **`window`** — для работы с динамическими окнами в интерфейсе Bitrix.
#### d. **`jquery` (если используется)** — jQuery может потребоваться для некоторых компонентов или пользовательских скриптов, если они зависят от этой библиотеки.

Пример подключения:

```php
<?php
CJSCore::Init(array('ajax', 'popup', 'window', 'jquery'));
?>
```

### 2. **Корректное подключение API Bitrix для работы с AJAX**
Bitrix предоставляет специальные функции для работы с AJAX-запросами и взаимодействия с сервером. Нужно следить за тем, чтобы запросы отрабатывались корректно:

- **`CModule::IncludeModule()`** — если вы используете модули, такие как `socialnetwork` или `main`, убедитесь, что они подключены перед выполнением операций.

Пример подключения в PHP:

```php
if (!CModule::IncludeModule('main')) {
    die('Ошибка подключения модуля');
}
```

### 3. **Работа с сессиями и авторизацией**

#### a. **Проверка валидности сессии**
Чтобы запросы работали корректно, нужно убедиться, что сессия валидна. При каждом AJAX-запросе проверяйте валидность сессии, используя функцию `check_bitrix_sessid()`.

Пример проверки сессии в PHP:

```php
if (!check_bitrix_sessid()) {
    echo json_encode(['success' => false, 'message' => 'Сессия истекла']);
    die();
}
```

#### b. **Проверка прав доступа пользователя**
Если доступ к выполнению AJAX-запросов ограничен (например, только для администраторов или определенных групп), проверяйте права доступа пользователя через:

```php
if (!$USER->IsAuthorized()) {
    echo json_encode(['success' => false, 'message' => 'Пользователь не авторизован']);
    die();
}
```

### 4. **Ожидание загрузки страницы и библиотек**

Обязательно используйте функцию `BX.ready()`, чтобы быть уверенным, что код выполняется после загрузки всех библиотек и элементов страницы:

```javascript
BX.ready(function() {
    // Ваш JavaScript код для AJAX-запросов или модальных окон
});
```

### 5. **Корректная работа URL и маршрутизация**

Когда отправляете запросы через AJAX, следите за тем, чтобы правильно обрабатывался путь к вашему обработчику:

- **Относительный путь**: Используйте путь, начиная с корня сайта (`/local/ajax/your_handler.php`), чтобы избежать проблем при выполнении запросов из разных частей сайта.
- **Обработка 404 и других ошибок**: Убедитесь, что путь к обработчику запроса существует и возвращает корректные данные. Обрабатывайте ошибки сервера и клиента на стороне PHP и JavaScript.

### 6. **Обработка данных и тип запроса**

- **Тип запроса**: Используйте правильный тип HTTP-запроса для передачи данных. Обычно это `POST` для отправки данных через AJAX.
  
- **Формат передачи данных**: Убедитесь, что данные передаются в корректном формате (например, JSON), и что PHP-обработчик правильно их обрабатывает.

### Пример полного AJAX-запроса:

#### JS (с использованием `BX.ajax`):
```javascript
BX.ready(function() {
    // AJAX-запрос
    BX.ajax({
        url: '/local/ajax/your_handler.php',
        method: 'POST',
        dataType: 'json', // Тип данных, которые вы ожидаете от сервера
        data: {
            sessid: BX.bitrix_sessid(), // Обязательно передаем сессионный идентификатор
            user_id: 1,
            group_id: 2
        },
        onsuccess: function(response) {
            if(response.success) {
                alert('Запрос выполнен успешно');
            } else {
                alert('Ошибка: ' + response.message);
            }
        },
        onfailure: function(error) {
            console.error('Произошла ошибка', error);
        }
    });
});
```

#### PHP (обработчик AJAX запроса):

```php
<?php
require($_SERVER["DOCUMENT_ROOT"] . "/bitrix/modules/main/include/prolog_before.php");

if (!check_bitrix_sessid()) {
    echo json_encode(['success' => false, 'message' => 'Неверная сессия']);
    die();
}

if (!$USER->IsAuthorized()) {
    echo json_encode(['success' => false, 'message' => 'Пользователь не авторизован']);
    die();
}

// Подключаем модули
if (!CModule::IncludeModule("main")) {
    echo json_encode(['success' => false, 'message' => 'Ошибка подключения модуля']);
    die();
}

// Логика добавления пользователя в группу
$user_id = intval($_POST['user_id']);
$group_id = intval($_POST['group_id']);

// Ваша логика добавления пользователя в группу...

echo json_encode(['success' => true, 'message' => 'Пользователь успешно добавлен']);
?>
```

### Вывод:

1. **Подключите библиотеки `ajax`, `popup`, `window` и при необходимости `jquery` через `CJSCore::Init()`.**
2. **Проверьте валидность сессий через `check_bitrix_sessid()` в PHP обработчике.**
3. **Убедитесь, что пользователи авторизованы, если запрос требует прав доступа.**
4. **Используйте правильные пути для AJAX-обработчиков и проверяйте работу URL.**
5. **Грамотно обрабатывайте данные и ошибки как на стороне клиента (JavaScript), так и на стороне сервера (PHP).**

Эти шаги помогут вам настроить корректную работу AJAX-запросов в Bitrix.


Чтобы реализовать функциональность, позволяющую вводить пользователей вручную с автоматическим отображением подходящих вариантов из списка, можно использовать **поле с автодополнением**. В Bitrix такая функциональность может быть реализована с помощью компонента **`BX.ui`** или через популярные библиотеки автокомплита (например, jQuery UI Autocomplete или Select2).

Я покажу пример, как это можно сделать, используя стандартные возможности **Bitrix** и его **AJAX**. Ниже приведен пример с автокомплитом для поля выбора пользователей.

### Шаги для реализации автокомплита:

1. **Обновим HTML для поля выбора пользователей:**

Мы будем использовать обычное текстовое поле для ввода имени пользователя с автоподстановкой.

```html
<!-- Поле ввода с автокомплитом -->
<label for="user_search">Пользователь:</label>
<input type="text" id="user_search" name="user_search" autocomplete="off" placeholder="Введите имя пользователя...">
<ul id="user_suggestions" style="display:none;"></ul> <!-- Список автоподсказок -->
```

2. **JS код для автоподстановки с использованием AJAX**:

Нам нужно написать код, который будет отправлять запрос на сервер для поиска пользователей при вводе текста в поле. В ответе сервер должен возвращать список пользователей, которые соответствуют введенному значению.

```javascript
BX.ready(function() {
    // Отслеживание ввода текста в поле
    document.getElementById('user_search').oninput = function() {
        const query = this.value.trim();

        if (query.length < 2) { // Не отправляем запрос, если длина строки меньше 2 символов
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
                            document.getElementById('user_search').value = user.NAME;
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
});
```

3. **CSS для стилизации списка подсказок**:

Добавим простой стиль для выпадающего списка, чтобы он выглядел красиво и корректно отображался под полем ввода.

```css
#user_suggestions {
    position: absolute;
    background-color: white;
    border: 1px solid #ccc;
    max-height: 150px;
    overflow-y: auto;
    width: 300px;
    list-style: none;
    padding: 0;
    margin-top: 0;
    z-index: 1000;
}

#user_suggestions li {
    padding: 8px;
    cursor: pointer;
}

#user_suggestions li:hover {
    background-color: #f0f0f0;
}
```

4. **PHP обработчик (`search_users.php`)**:

Создадим обработчик, который будет принимать запрос на поиск пользователей и возвращать результаты в формате JSON.

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

$query = htmlspecialcharsbx($_POST['query']);

// Подключение нужных модулей
if (!CModule::IncludeModule('main')) {
    echo json_encode([]);
    die();
}

// Поиск пользователей по введенному запросу
$arFilter = [
    'ACTIVE' => 'Y',
    '?NAME' => $query, // Ищем по имени
    'LOGIC' => 'OR',
    '?LAST_NAME' => $query, // Ищем по фамилии
    '?EMAIL' => $query // Ищем по email
];
$rsUsers = CUser::GetList(($by = "NAME"), ($order = "ASC"), $arFilter, ['FIELDS' => ['ID', 'NAME', 'LAST_NAME', 'EMAIL']]);

$users = [];
while ($arUser = $rsUsers->Fetch()) {
    $users[] = [
        'ID' => $arUser['ID'],
        'NAME' => $arUser['NAME'] . ' ' . $arUser['LAST_NAME'],
        'EMAIL' => $arUser['EMAIL']
    ];
}

echo json_encode($users);
?>
```

### Пример работы:

1. Пользователь начинает вводить имя пользователя в текстовое поле.
2. Происходит AJAX-запрос на сервер (в обработчик `search_users.php`), который ищет пользователей по введенному запросу.
3. Сервер возвращает список подходящих пользователей.
4. В выпадающем списке отображаются результаты поиска.
5. Пользователь может кликнуть на нужного пользователя, и его данные будут вставлены в поле ввода.

### Заключение:

Этот код позволяет пользователю вводить текст и автоматически подсказывать возможные варианты пользователей на основе запроса. Такой механизм можно легко расширить, например, добавить сортировку по релевантности, расширить критерии поиска (по телефону, должности и т.д.) или стилизовать список вариантов для лучшего восприятия.
