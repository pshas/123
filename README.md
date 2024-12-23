Давайте доработаем код, чтобы поддерживать два поля ввода (`input`). Каждое из них будет отправлять данные через метод `POST`, и сервер будет обрабатывать запросы для обоих полей.

---

### 1. **HTML с двумя полями ввода**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Подсказки для двух полей</title>
    <style>
        .suggestions {
            border: 1px solid #ccc;
            max-width: 300px;
            position: absolute;
            background: white;
            z-index: 10;
        }

        .suggestions div {
            padding: 5px;
            cursor: pointer;
        }

        .suggestions div:hover {
            background-color: #f0f0f0;
        }
    </style>
</head>
<body>
    <h1>Подсказки для двух полей ввода</h1>

    <input type="text" id="field1" placeholder="Введите что-то в поле 1">
    <div id="suggestions1" class="suggestions"></div>

    <input type="text" id="field2" placeholder="Введите что-то в поле 2">
    <div id="suggestions2" class="suggestions"></div>

    <script>
        // Функция для отправки данных через POST
        function fetchSuggestions(query, fieldId) {
            if (query.length === 0) {
                document.getElementById(`suggestions${fieldId}`).innerHTML = '';
                return;
            }

            fetch('suggestions.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: `query=${encodeURIComponent(query)}&fieldId=${fieldId}` // Передача данных
            })
            .then(response => response.json())
            .then(data => {
                const suggestionsBox = document.getElementById(`suggestions${fieldId}`);
                suggestionsBox.innerHTML = '';

                data.forEach(item => {
                    const div = document.createElement('div');
                    div.textContent = item;
                    div.onclick = () => {
                        document.getElementById(`field${fieldId}`).value = item;
                        suggestionsBox.innerHTML = '';
                    };
                    suggestionsBox.appendChild(div);
                });
            })
            .catch(error => console.error('Ошибка:', error));
        }

        // Обработчики ввода для двух полей
        document.getElementById('field1').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, 1);
        });

        document.getElementById('field2').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, 2);
        });
    </script>
</body>
</html>
```

---

### 2. **PHP-скрипт для обработки запросов**

В этом скрипте мы будем различать запросы от разных полей с помощью параметра `fieldId`.

```php
<?php
// Подключение к базе данных
$host = 'localhost';
$db = 'your_database';
$user = 'your_username';
$pass = 'your_password';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$db;charset=utf8", $user, $pass);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Ошибка подключения к базе данных: " . $e->getMessage());
}

// Проверяем, передан ли запрос методом POST
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Получаем данные из тела запроса
    $input = $_POST['query'] ?? '';
    $fieldId = $_POST['fieldId'] ?? '';

    // Проверяем, что запрос не пуст
    if (!empty($input) && !empty($fieldId)) {
        $column = $fieldId === '1' ? 'column1' : 'column2'; // Определяем колонку для поиска

        // SQL-запрос с использованием LIKE
        $stmt = $pdo->prepare("SELECT DISTINCT $column FROM your_table WHERE $column LIKE :query LIMIT 10");
        $stmt->execute(['query' => '%' . $input . '%']);

        // Получаем результаты
        $results = $stmt->fetchAll(PDO::FETCH_COLUMN);

        // Возвращаем результаты в формате JSON
        header('Content-Type: application/json');
        echo json_encode($results);
    } else {
        // Если запрос пустой, возвращаем пустой массив
        header('Content-Type: application/json');
        echo json_encode([]);
    }
} else {
    // Если метод не POST, возвращаем ошибку
    http_response_code(405);
    echo json_encode(['error' => 'Method not allowed']);
}
```

---

### Объяснение:

1. **HTML и JavaScript**:
   - Для каждого поля ввода (`field1` и `field2`) используется отдельный обработчик `input`.
   - При вводе данных отправляется запрос через метод `POST` с параметрами:
     - `query` — текст, введённый пользователем.
     - `fieldId` — идентификатор поля (1 для `field1` и 2 для `field2`).
   - Результаты отображаются в соответствующем блоке подсказок (`suggestions1` или `suggestions2`).

2. **PHP**:
   - В зависимости от `fieldId` выполняется поиск в соответствующей колонке (`column1` для `field1`, `column2` для `field2`).
   - Возвращается массив данных в формате JSON.

3. **SQL-запрос**:
   - Подставляем название колонки (`column1` или `column2`) для выполнения поиска с помощью оператора `LIKE`.

---

### Настройка:

1. Замените `your_table`, `column1`, и `column2` на реальные названия таблицы и колонок в вашей базе данных.
2. Убедитесь, что сервер возвращает данные корректно.

Теперь оба поля ввода (`field1` и `field2`) будут работать независимо, выполняя поиск в соответствующих колонках.
