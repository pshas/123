Чтобы добавить указанные поля в модальное окно, необходимо обновить HTML форму в модальном окне, добавив новые поля для ввода. Каждый из этих полей будет принимать информацию от пользователя и затем отправлять её на сервер для обработки.

### Обновленный код модального окна с дополнительными полями:

```html
<!-- Модальное окно -->
<div class="modal fade" id="submitModal" tabindex="-1" role="dialog" aria-labelledby="submitModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="submitModalLabel">Оформить заявку</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <p>Заполните необходимые данные для оформления заявки:</p>
        
        <!-- Форма для отправки -->
        <form id="submitForm">
          <!-- Скрытое поле для ID шаблона -->
          <input type="hidden" name="template_id" id="templateId" value="">

          <!-- Объект исследования -->
          <div class="form-group">
            <label for="research_object">Объект исследования</label>
            <input type="text" class="form-control" name="research_object" id="research_object" required>
          </div>

          <!-- Инициатор -->
          <div class="form-group">
            <label for="initiator">Инициатор</label>
            <input type="text" class="form-control" name="initiator" id="initiator" required>
          </div>

          <!-- Дата создания (поле только для отображения) -->
          <div class="form-group">
            <label for="created_at">Создано</label>
            <input type="text" class="form-control" name="created_at" id="created_at" value="<?php echo date('Y-m-d H:i:s'); ?>" readonly>
          </div>

          <!-- Причина обращения -->
          <div class="form-group">
            <label for="reason">Причина обращения</label>
            <input type="text" class="form-control" name="reason" id="reason" required>
          </div>

          <!-- Цель испытаний и исследований -->
          <div class="form-group">
            <label for="goal">Цель испытаний и исследований</label>
            <textarea class="form-control" name="goal" id="goal" rows="3" required></textarea>
          </div>

          <!-- Статус -->
          <div class="form-group">
            <label for="status">Статус</label>
            <select class="form-control" name="status" id="status" required>
              <option value="Новый">Новый</option>
              <option value="В процессе">В процессе</option>
              <option value="Завершён">Завершён</option>
            </select>
          </div>

          <!-- Отчёт -->
          <div class="form-group">
            <label for="report">Отчёт</label>
            <textarea class="form-control" name="report" id="report" rows="3"></textarea>
          </div>

          <!-- Кнопка подтверждения -->
          <button type="submit" class="btn btn-success">Подтвердить оформление</button>
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Отмена</button>
      </div>
    </div>
  </div>
</div>

<!-- Уведомление об успешном оформлении -->
<div id="successMessage" class="alert alert-success" style="display:none;">
    Заявка успешно оформлена!
</div>
```

### Обновление JavaScript для отправки формы через AJAX

Теперь при отправке формы через AJAX необходимо передать все новые поля на сервер:

```html
<script>
// Функция для открытия модального окна и передачи ID шаблона
function openSubmitModal(templateId) {
    // Устанавливаем значение ID шаблона в скрытое поле формы
    document.getElementById('templateId').value = templateId;
    
    // Открываем модальное окно
    $('#submitModal').modal('show');
}

// Обработчик формы для отправки через AJAX
document.getElementById('submitForm').addEventListener('submit', function(e) {
    e.preventDefault(); // Отключаем стандартную отправку формы

    // Получаем данные формы
    var formData = new FormData(this);

    // Отправляем данные через AJAX
    fetch('submit_template.php', {
        method: 'POST',
        body: formData
    })
    .then(response => response.json()) // Предполагается, что сервер возвращает JSON
    .then(data => {
        if (data.success) {
            // Закрываем модальное окно
            $('#submitModal').modal('hide');

            // Показываем сообщение об успехе
            document.getElementById('successMessage').style.display = 'block';

            // Скрываем сообщение через несколько секунд
            setTimeout(() => {
                document.getElementById('successMessage').style.display = 'none';
            }, 3000);
        } else {
            alert('Ошибка: ' + data.error); // Выводим ошибку, если что-то пошло не так
        }
    })
    .catch(error => {
        console.error('Ошибка при отправке запроса:', error);
    });
});
</script>
```

### Обновление обработки на стороне сервера в `submit_template.php`

Теперь сервер должен обрабатывать и сохранять все новые поля в базе данных:

```php
<?php
// Проверка данных
if (isset($_POST['template_id']) && isset($_POST['research_object']) && isset($_POST['initiator']) && isset($_POST['reason']) && isset($_POST['goal']) && isset($_POST['status'])) {
    $template_id = $_POST['template_id'];
    $research_object = $_POST['research_object'];
    $initiator = $_POST['initiator'];
    $created_at = $_POST['created_at'];
    $reason = $_POST['reason'];
    $goal = $_POST['goal'];
    $status = $_POST['status'];
    $report = isset($_POST['report']) ? $_POST['report'] : ''; // Отчёт может быть пустым

    // Логика для сохранения данных в базе данных
    // Например, обновление статуса заявки в таблице

    $pdo = new PDO('mysql:host=localhost;dbname=your_database', 'username', 'password');
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    $stmt = $pdo->prepare("INSERT INTO user_templates (template_id, research_object, initiator, created_at, reason, goal, status, report) VALUES (:template_id, :research_object, :initiator, :created_at, :reason, :goal, :status, :report)");
    
    $stmt->execute([
        ':template_id' => $template_id,
        ':research_object' => $research_object,
        ':initiator' => $initiator,
        ':created_at' => $created_at,
        ':reason' => $reason,
        ':goal' => $goal,
        ':status' => $status,
        ':report' => $report
    ]);

    // Сообщаем об успешной операции
    $response = ['success' => true];
} else {
    // Сообщаем об ошибке
    $response = ['success' => false, 'error' => 'Неверные данные'];
}

// Отправляем ответ в формате JSON
header('Content-Type: application/json');
echo json_encode($response);
```

### Как это работает:

1. **Модальное окно**: При нажатии на кнопку "Оформить", модальное окно открывается, и пользователю предлагается заполнить дополнительные поля.
2. **AJAX отправка**: После заполнения формы данные отправляются на сервер через AJAX без перезагрузки страницы.
3. **Серверная обработка**: На сервере данные сохраняются в базе данных (например, в таблице `user_templates`).
4. **Ответ JSON**: Сервер отвечает JSON-ответом. Если операция успешна, выводится уведомление о том, что заявка оформлена.

Теперь все поля из модального окна будут передаваться на сервер без перезагрузки страницы, и данные будут сохраняться в базе данных.



Проблема с тем, что модальное окно открывается при загрузке страницы, скорее всего, связана с тем, что функция `openSubmitModal()` вызывается автоматически из-за некорректного вызова в PHP-коде.

### Причина
Если в вашем PHP-коде вызов функции выглядит так:

```php
<button class='btn btn-success btn-sm' onclick='openSubmitModal(<?php echo $row['id']; ?>)'>Оформить</button>
```

Это правильно. Но если вы случайно добавили круглые скобки к вызову функции, как:

```php
<button class='btn btn-success btn-sm' onclick='openSubmitModal(<?php echo $row['id']; ?>())'>Оформить</button>
```

То это приведет к тому, что функция выполнится сразу при рендеринге страницы, что и вызывает проблему.

### Исправление

1. **Проверьте правильность передачи аргумента в JavaScript**:
   Убедитесь, что в вашем PHP-коде правильно сформирована кнопка с передачей ID без лишних скобок:

   ```php
   <button class='btn btn-success btn-sm' onclick='openSubmitModal(<?php echo $row['id']; ?>)'>Оформить</button>
   ```

2. **JavaScript без немедленного вызова**:
   Убедитесь, что вызов `openSubmitModal()` происходит только внутри события, а не сразу на этапе рендеринга страницы.

   Правильный JavaScript:

   ```html
   <script>
   // Функция для открытия модального окна и передачи ID шаблона
   function openSubmitModal(templateId) {
       // Устанавливаем значение ID шаблона в скрытое поле формы
       document.getElementById('templateId').value = templateId;
       
       // Открываем модальное окно
       $('#submitModal').modal('show');
   }
   </script>
   ```

### Дополнительно: проверка логики работы с modals

Если проблема не решится, также попробуйте временно добавить лог, чтобы убедиться, что именно вызывает проблему:

```html
<script>
function openSubmitModal(templateId) {
    console.log("Modal открывается с ID:", templateId);  // Лог для проверки вызова
    document.getElementById('templateId').value = templateId;
    $('#submitModal').modal('show');
}
</script>
```

Это позволит отследить, в какой момент функция вызывается и с каким ID.

### Подведение итогов:
1. Убедитесь, что вызов JavaScript в PHP-коде кнопки правильный, и ID шаблона передается корректно: `onclick='openSubmitModal(<?php echo $row['id']; ?>)'`.
2. Добавьте логирование в JavaScript, чтобы отследить, когда именно вызывается функция открытия модального окна.
3. Проверьте кэш браузера, если старые данные всё ещё отображаются неправильно.

Теперь модальное окно должно открываться корректно только при нажатии кнопки "Оформить".


Ошибка `Uncaught TypeError: $().modal is not a function` обычно указывает на то, что jQuery не загружен или загружен некорректно, либо отсутствует подключение к Bootstrap JavaScript-файлам, которые предоставляют функциональность для модальных окон.

### Шаги для решения:

1. **Убедитесь, что jQuery подключен корректно:**

   Для работы модальных окон Bootstrap требуется подключение jQuery. Проверьте, что у вас есть корректное подключение jQuery перед подключением скриптов Bootstrap.

   Пример правильного подключения:

   ```html
   <!-- jQuery -->
   <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

   <!-- Bootstrap CSS -->
   <link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

   <!-- Bootstrap JS (зависит от jQuery) -->
   <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>
   ```

2. **Загрузите jQuery до Bootstrap:**

   Важно, чтобы jQuery загружался **до** подключения Bootstrap JS, так как Bootstrap использует функции jQuery. Если порядок неверный, это вызовет ошибку.

3. **Проверьте подключение версий:**

   Bootstrap 4 использует jQuery, но если вы используете Bootstrap 5, он больше не зависит от jQuery, и нужно подключать Bootstrap JavaScript-файлы отдельно. Вот пример для Bootstrap 5:

   ```html
   <!-- Bootstrap 5 CSS -->
   <link href="https://stackpath.bootstrapcdn.com/bootstrap/5.3.0/css/bootstrap.min.css" rel="stylesheet">

   <!-- Bootstrap 5 JS (без jQuery) -->
   <script src="https://stackpath.bootstrapcdn.com/bootstrap/5.3.0/js/bootstrap.bundle.min.js"></script>
   ```

4. **Проверка кода модального окна:**

   Убедитесь, что вызываете модальное окно корректно, используя правильный код для работы с Bootstrap:

   ```html
   <script>
   function openSubmitModal(templateId) {
       // Устанавливаем значение ID шаблона в скрытое поле формы
       document.getElementById('templateId').value = templateId;
       
       // Открываем модальное окно
       $('#submitModal').modal('show');  // Для Bootstrap 4
       // Для Bootstrap 5 (если без jQuery, то это вызов):
       // var modal = new bootstrap.Modal(document.getElementById('submitModal'));
       // modal.show();
   }
   </script>
   ```

   В Bootstrap 5 больше не используется jQuery для модальных окон, поэтому для Bootstrap 5 необходимо использовать JavaScript API напрямую.

### Заключение:

- Для **Bootstrap 4**: Проверьте, что jQuery подключен правильно, перед Bootstrap скриптами.
- Для **Bootstrap 5**: Используйте JavaScript API напрямую, без jQuery.

Попробуйте один из этих подходов в зависимости от версии Bootstrap, которую вы используете.
