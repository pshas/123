Ошибка `"Unexpected end of JSON input"` возникает, если функция `response.json()` вызывается на ответе, который не содержит корректные данные в формате JSON. Это может происходить по разным причинам. Рассмотрим способы устранения проблемы.

---

### Причины ошибки

1. **Пустой ответ от сервера**: 
   Сервер может не возвращать никаких данных.
   
2. **Ответ не является JSON**: 
   Сервер возвращает данные в другом формате (например, текст или HTML).
   
3. **Сервер возвращает ошибку (4xx или 5xx)**:
   В таких случаях `response.json()` не сработает, так как тело ответа может быть пустым или невалидным.

---

### Решения

#### 1. **Проверка статуса ответа**
Перед вызовом `response.json()`, проверьте статус ответа:

```javascript
fetch('suggestions.php?query=test')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.json();
    })
    .then(data => {
        console.log('Data:', data);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

Если сервер возвращает ошибочный статус (например, `404` или `500`), будет вызван блок `catch`.

---

#### 2. **Проверка на пустой ответ**
Если сервер может вернуть пустой ответ, обработайте это:

```javascript
fetch('suggestions.php?query=test')
    .then(response => response.text()) // Читаем как текст
    .then(text => {
        if (text.trim() === '') {
            throw new Error('Ответ сервера пуст');
        }
        return JSON.parse(text); // Преобразуем текст в JSON
    })
    .then(data => {
        console.log('Data:', data);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

---

#### 3. **Отладка на стороне сервера**
Убедитесь, что сервер возвращает валидный JSON. В `PHP` это можно сделать так:

```php
header('Content-Type: application/json');
echo json_encode($results);
```

Проверьте, не возникает ли ошибок до отправки данных, которые могут испортить JSON-ответ.

---

#### 4. **Проверка через отладчик (Network Tab)**
Используйте инструменты разработчика в браузере (вкладка **Network**), чтобы увидеть, что возвращает сервер. 
1. Проверьте, не содержит ли ответ HTML (например, сообщения об ошибке PHP).
2. Убедитесь, что Content-Type установлен как `application/json`.

---

#### 5. **Проверка на наличие ответа**
Если сервер возвращает `204 No Content` или ответ пуст, это также нужно обработать:

```javascript
fetch('suggestions.php?query=test')
    .then(response => {
        if (response.status === 204) {
            throw new Error('Нет содержимого');
        }
        return response.json();
    })
    .then(data => {
        console.log('Data:', data);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

---

#### 6. **Обновлённый пример с учётом всех ошибок**

Вот пример с обработкой всех описанных случаев:

```javascript
fetch('suggestions.php?query=test')
    .then(response => {
        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.text(); // Читаем как текст
    })
    .then(text => {
        if (text.trim() === '') {
            throw new Error('Ответ сервера пуст');
        }
        return JSON.parse(text); // Парсим JSON
    })
    .then(data => {
        console.log('Data:', data);
    })
    .catch(error => {
        console.error('Ошибка:', error);
    });
```

---

### Проверка на стороне PHP

Убедитесь, что PHP-скрипт возвращает валидный JSON:

1. **Обработка пустых данных**:
   ```php
   if (empty($results)) {
       echo json_encode([]);
       exit;
   }
   ```

2. **Проверка ошибок до вывода**:
   ```php
   try {
       // Ваш код
       echo json_encode($results);
   } catch (Exception $e) {
       http_response_code(500);
       echo json_encode(['error' => $e->getMessage()]);
   }
   ```

---

### Резюме

1. Убедитесь, что сервер возвращает корректный JSON.
2. Проверьте статус ответа перед вызовом `response.json()`.
3. Обрабатывайте пустой или невалидный ответ.
4. Используйте отладку в инструментах разработчика.

Эти шаги помогут устранить ошибку `"Unexpected end of JSON input"`.
