Если необходимо искать данные по обоим полям в одной таблице и автозаполнять соответствующее поле ввода при выборе результата, то вот обновленный пример:

---

### 1. Файл `suggestions.php` (поиск по двум столбцам):
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

// Проверка входных данных
if (isset($_GET['query'])) {
    $input = $_GET['query'];

    // Указываем таблицу и поля для поиска
    $table = 'your_table';
    $columns = ['column1', 'column2']; // Поля для поиска

    // SQL-запрос с использованием LIKE
    $query = "SELECT DISTINCT column1, column2 
              FROM $table 
              WHERE column1 LIKE :query OR column2 LIKE :query 
              LIMIT 10";

    $stmt = $pdo->prepare($query);
    $stmt->execute(['query' => '%' . $input . '%']);
    $results = $stmt->fetchAll(PDO::FETCH_ASSOC);

    // Возвращаем результат в формате JSON
    header('Content-Type: application/json');
    echo json_encode($results);
} else {
    // Если параметры отсутствуют
    http_response_code(400);
    echo json_encode(['error' => 'Invalid parameters']);
}
?>
```

---

### 2. HTML-страница (с автозаполнением):
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Подсказки из базы данных</title>
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
    <h1>Подсказки для полей ввода</h1>
    <input type="text" id="field1" placeholder="Введите что-то (поле 1)">
    <div id="suggestions1" class="suggestions"></div>

    <input type="text" id="field2" placeholder="Введите что-то (поле 2)">
    <div id="suggestions2" class="suggestions"></div>

    <script>
        // Функция для получения подсказок
        function fetchSuggestions(query, fieldId) {
            if (query.length === 0) {
                document.getElementById(`suggestions${fieldId}`).innerHTML = '';
                return;
            }

            fetch(`suggestions.php?query=${encodeURIComponent(query)}`)
                .then(response => response.json())
                .then(data => {
                    const suggestionsBox = document.getElementById(`suggestions${fieldId}`);
                    suggestionsBox.innerHTML = '';

                    data.forEach(item => {
                        const div = document.createElement('div');
                        div.textContent = fieldId === '1' ? item.column1 : item.column2;

                        div.onclick = () => {
                            document.getElementById(`field${fieldId}`).value = div.textContent;
                            suggestionsBox.innerHTML = '';
                        };

                        suggestionsBox.appendChild(div);
                    });
                })
                .catch(error => console.error('Ошибка:', error));
        }

        // Обработчики ввода для каждого поля
        document.getElementById('field1').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, '1');
        });

        document.getElementById('field2').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, '2');
        });
    </script>
</body>
</html>
```

---

### Объяснение:
1. **PHP**:
   - SQL-запрос ищет совпадения в обоих столбцах (`column1` и `column2`) с помощью оператора `LIKE`.
   - Возвращаются оба столбца (`column1` и `column2`) для подсказок.

2. **HTML и JS**:
   - Для каждого поля ввода (`field1` и `field2`) отображаются соответствующие подсказки.
   - При выборе подсказки текст автоматически вставляется в поле, откуда производился ввод.

3. **Особенности**:
   - Поле `field1` отображает результаты из столбца `column1`.
   - Поле `field2` отображает результаты из столбца `column2`.
   - Поиск происходит одновременно по обоим столбцам.

---

### Настройка:
- Замените `your_table`, `column1`, и `column2` на реальные названия таблицы и колонок.
- При необходимости можно добавить дополнительные столбцы или усложнить логику подсказок.

Этот подход обеспечивает поиск по обоим столбцам и корректное автозаполнение.
