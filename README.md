Для изменения поля `status` и поля `is_activate` при нажатии кнопки "Подтвердить оформление" в модальном окне, вам нужно реализовать следующую логику:

1. **Добавить кнопку "Подтвердить оформление" в модальное окно.**
2. **Обновить статус и активность шаблона через AJAX-запрос к серверу.**
3. **После успешного обновления — закрыть модальное окно.**

### Шаги по реализации:

1. **Изменение HTML модального окна:**
   
   Добавьте кнопку "Подтвердить оформление" рядом с кнопкой "Сохранить изменения":

   ```html
   <!-- Модальное окно -->
   <div class="modal fade" id="submitModal" tabindex="-1" role="dialog" aria-labelledby="submitModalLabel" aria-hidden="true">
     <div class="modal-dialog" role="document">
       <div class="modal-content">
         <div class="modal-header">
           <h5 class="modal-title" id="submitModalLabel">Редактировать заявку</h5>
           <button type="button" class="close" data-dismiss="modal" aria-label="Close">
             <span aria-hidden="true">&times;</span>
           </button>
         </div>
         <div class="modal-body">
           <!-- Форма для редактирования -->
           <form id="submitForm">
             <input type="hidden" name="template_id" id="templateId" value="">

             <!-- Поля формы (пример) -->
             <div class="form-group">
               <label for="research_object">Объект исследования</label>
               <input type="text" class="form-control" name="research_object" id="research_object" required>
             </div>

             <!-- Другие поля... -->

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

             <div class="modal-footer">
               <!-- Кнопка "Сохранить изменения" -->
               <button type="submit" class="btn btn-primary">Сохранить изменения</button>
               
               <!-- Кнопка "Подтвердить оформление" -->
               <button type="button" class="btn btn-success" onclick="confirmSubmission()">Подтвердить оформление</button>
             </div>
           </form>
         </div>
       </div>
     </div>
   </div>
   ```

2. **JavaScript для обработки кнопки "Подтвердить оформление":**

   Теперь мы реализуем функцию `confirmSubmission()`, которая будет менять поле `status` и `is_activate` через AJAX-запрос.

   ```html
   <script>
   // Функция для подтверждения оформления
   function confirmSubmission() {
       // Получаем ID шаблона из скрытого поля
       const templateId = document.getElementById('templateId').value;

       // Отправляем AJAX-запрос на сервер для обновления статуса и активации
       fetch('confirm_submission.php', {
           method: 'POST',
           headers: {
               'Content-Type': 'application/json',
           },
           body: JSON.stringify({
               id: templateId,
               status: 'Завершён',  // Меняем статус на "Завершён"
               is_activate: 0       // Меняем активность на "Деактивировано"
           }),
       })
       .then(response => response.json())
       .then(data => {
           if (data.success) {
               // Если успешно, закрываем модальное окно
               $('#submitModal').modal('hide');

               // Обновляем интерфейс, если нужно
               alert('Заявка успешно оформлена!');
           } else {
               alert('Ошибка при оформлении заявки.');
           }
       })
       .catch(error => {
           console.error('Ошибка при оформлении:', error);
       });
   }
   </script>
   ```

3. **PHP-скрипт `confirm_submission.php` для обновления полей:**

   На стороне сервера создайте файл `confirm_submission.php`, который будет обрабатывать запрос на изменение полей `status` и `is_activate`.

   ```php
   <?php
   // Подключение к базе данных
   $pdo = new PDO('mysql:host=localhost;dbname=your_database', 'username', 'password');
   $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

   // Получаем данные из POST-запроса
   $data = json_decode(file_get_contents('php://input'), true);

   if (isset($data['id'])) {
       $template_id = $data['id'];
       $status = $data['status'];
       $is_activate = $data['is_activate'];

       // Запрос для обновления статуса и активации шаблона
       $stmt = $pdo->prepare("UPDATE user_templates SET status = :status, is_activate = :is_activate WHERE id = :id");
       $result = $stmt->execute([
           ':status' => $status,
           ':is_activate' => $is_activate,
           ':id' => $template_id
       ]);

       if ($result) {
           // Возвращаем успешный ответ
           echo json_encode(['success' => true]);
       } else {
           echo json_encode(['success' => false, 'error' => 'Не удалось обновить шаблон.']);
       }
   } else {
       echo json_encode(['success' => false, 'error' => 'ID не передан.']);
   }
   ?>
   ```

### Как это работает:

1. Пользователь нажимает на кнопку "Подтвердить оформление".
2. Вызов функции `confirmSubmission()` отправляет AJAX-запрос на сервер с ID шаблона и новыми значениями для полей `status` и `is_activate`.
3. PHP-скрипт на сервере (`confirm_submission.php`) обновляет соответствующую запись в базе данных.
4. После успешного обновления, модальное окно закрывается, и можно показать сообщение о успешном оформлении.

### Поля:
- `status`: Меняется на "Завершён" при оформлении.
- `is_activate`: Меняется на `0` (деактивировано) при оформлении.

Этот процесс происходит без перезагрузки страницы и обновляет только нужные данные в базе.
