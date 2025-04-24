<?
	require("../header.php");
?>

<!-- Bootstrap CSS -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS (зависит от jQuery) -->
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>

<body>
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
            <label for="task">Причина обращения</label>
            <input type="text" class="form-control" name="task" id="task" required>
          </div>

          <!-- Цель испытаний и исследований -->
          <div class="form-group">
            <label for="note">Цель испытаний и исследований</label>
            <textarea class="form-control" name="note" id="note" rows="3" required></textarea>
          </div>

          <!-- Кнопка подтверждения -->
          <button type="button" class="btn btn-success" onclick="confirmSubmission()">Подтвердить оформление</button>
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
	<div class="container">
		<h1 class="mt-4 mb-4">Мои заявки</h1>
		<table class="table table-striped table-bordered">
		<thead>
			<tr>
				<th>Номер</th>
				<th>Создан</th>
				<th>Инициатор</th>
				<th>Цех/Отдел</th>
				<th>Наименование</th>
				<th>Номер детали</th>
				<th>Дата изготовления</th>
				<th>Номер партии</th>
				<th>Несоответствие</th>
				<th>Статус</th>
        <th>Действие</th>
			</tr>
		<thead>
		<tbody>
		<? $dsn = 'mysql:host=localhost;dbname=test_db';
		$username = 'root';
		$password = 'root';

		$pdo = new PDO($dsn, $username, $password);
		$user_id = $USER->GetId();
		$stmt = $pdo->prepare("SELECT * FROM user_applications a WHERE user_id =:user_id OR lab_id =:user_id");
		$stmt->execute([':user_id' => $user_id]);
		while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
			echo "<tr>";
				echo "<td>" . htmlspecialchars($row['id']) . "</td>";
				echo "<td>" . htmlspecialchars($row['created_at']) . "</td>";
				echo "<td>" . htmlspecialchars($row['full_name']) . "</td>";
				echo "<td>" . htmlspecialchars($row['work_department']) . "</td>";
				echo "<td>" . htmlspecialchars($row['NAME_IZD']) . "</td>";
				echo "<td>" . htmlspecialchars($row['ID_I']) . "</td>";
				echo "<td>" . htmlspecialchars($row['date_man']) . "</td>";
				echo "<td>" . htmlspecialchars($row['']) . "</td>";
				echo "<td>" . htmlspecialchars($row['task']) . "</td>";
				echo "<td>" . htmlspecialchars($row['status']) . "</td>";
        if ($row['is_editable'] === 1) {
				echo "<td> <button id='submitModal' class='btn btn-success btn-sm' onclick='openSubmitModal(" . $row['id'] . ")'>Оформить</button></td>";
        }
        if ($row['is_editable'] === 2 && $row['lab_id'] = $user_id) {
          echo "<td> <a href='report_form.php?id=" . $row['id'] . "'class='btn btn-info btn-sm''>Отчёт</td>";
          }
        
}
?>
		</tbody>
		</table>
	</div>
</body>

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
            document.getElementById('task').value = data.task;
            document.getElementById('note').value = data.goal;
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
            status: 'Ожидание',  
            is_editable: 0      
        }),
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            // Если успешно, закрываем модальное окно
            $('#submitModal').modal('hide');

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
