Чтобы подставить значения в поля модального окна в зависимости от переданного ID (например, данных, которые хранятся в базе данных), вам нужно выполнить следующие шаги:

1. **Передача ID шаблона в функцию для открытия модального окна.**
2. **Выполнение AJAX-запроса к серверу для получения данных.**
3. **Заполнение полей модального окна полученными данными.**

### Шаги по реализации:

1. **HTML для модального окна:**

   Вот обновленная разметка для модального окна, где у вас будут поля для редактирования:

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

             <button type="submit" class="btn btn-primary">Сохранить изменения</button>
           </form>
         </div>
       </div>
     </div>
   </div>
   ```

2. **JavaScript для открытия модального окна и заполнения его данными:**

   Теперь нам нужно сделать AJAX-запрос к серверу для получения данных по указанному ID. Эти данные затем подставляются в поля формы.

   ```html
   <script>
   // Функция для открытия модального окна и подставления данных
   function openSubmitModal(templateId) {
       // Устанавливаем значение ID в скрытое поле формы
       document.getElementById('templateId').value = templateId;

       // Выполняем AJAX-запрос для получения данных о заявке
       fetch('get_template_data.php?id=' + templateId)
       .then(response => response.json()) // Ожидаем, что сервер вернет JSON
       .then(data => {
           if (data.success) {
               // Заполняем поля модального окна полученными данными
               document.getElementById('research_object').value = data.research_object;
               document.getElementById('initiator').value = data.initiator;
               document.getElementById('reason').value = data.reason;
               document.getElementById('goal').value = data.goal;
               document.getElementById('status').value = data.status;
               document.getElementById('report').value = data.report;

               // Открываем модальное окно
               $('#submitModal').modal('show');
           } else {
               alert('Не удалось загрузить данные шаблона.');
           }
       })
       .catch(error => {
           console.error('Ошибка при загрузке данных шаблона:', error);
       });
   }
   </script>
   ```

3. **PHP-файл для получения данных `get_template_data.php`:**

   Этот скрипт на сервере будет возвращать данные по шаблону с указанным ID. Он будет запрашивать данные из базы данных и возвращать их в формате JSON.

   ```php
   <?php
   // Подключение к базе данных
   $pdo = new PDO('mysql:host=localhost;dbname=your_database', 'username', 'password');
   $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

   if (isset($_GET['id'])) {
       $template_id = $_GET['id'];

       // Запрос для получения данных о шаблоне
       $stmt = $pdo->prepare("SELECT * FROM user_templates WHERE id = :id");
       $stmt->execute([':id' => $template_id]);

       $template = $stmt->fetch(PDO::FETCH_ASSOC);

       if ($template) {
           // Возвращаем данные в формате JSON
           echo json_encode([
               'success' => true,
               'research_object' => $template['research_object'],
               'initiator' => $template['initiator'],
               'reason' => $template['reason'],
               'goal' => $template['goal'],
               'status' => $template['status'],
               'report' => $template['report']
           ]);
       } else {
           echo json_encode(['success' => false, 'error' => 'Шаблон не найден.']);
       }
   } else {
       echo json_encode(['success' => false, 'error' => 'ID не передан.']);
   }
   ?>
   ```

4. **Кнопка для открытия модального окна:**

   Убедитесь, что кнопка для открытия модального окна передает корректный `template_id`:

   ```php
   echo "<td>
           <button class='btn btn-success btn-sm' onclick='openSubmitModal(" . $row['id'] . ")'>Оформить</button>
         </td>";
   ```

### Как это работает:

1. Когда пользователь нажимает на кнопку "Оформить", вызывается функция `openSubmitModal(templateId)`, в которой передается ID соответствующего шаблона.
2. Через AJAX-запрос данные этого шаблона подгружаются с сервера (из файла `get_template_data.php`), и поля модального окна заполняются этими данными.
3. После этого модальное окно открывается, и пользователь может редактировать данные.
4. При сохранении изменений данные отправляются на сервер для обработки, и шаблон обновляется.

Теперь модальное окно будет открываться с подставленными данными, соответствующими выбранному шаблону.
