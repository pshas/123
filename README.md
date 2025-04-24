Вот модифицированный код, который позволяет изменять статус заявки на "приостановлено", "выполнено", "отклонено" и "отменено":

```php
<?php
require("../header.php");
?>

<!-- Bootstrap CSS -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS (зависит от jQuery) -->
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>

<body>
<!-- Модальное окно для оформления заявки -->
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
        <form id="submitForm">
          <input type="hidden" name="template_id" id="templateId" value="">
          <div class="form-group">
            <label for="research_object">Объект исследования</label>
            <input type="text" class="form-control" name="research_object" id="research_object" required>
          </div>
          <div class="form-group">
            <label for="initiator">Инициатор</label>
            <input type="text" class="form-control" name="initiator" id="initiator" required>
          </div>
          <div class="form-group">
            <label for="created_at">Создано</label>
            <input type="text" class="form-control" name="created_at" id="created_at" value="<?php echo date('Y-m-d H:i:s'); ?>" readonly>
          </div>
          <div class="form-group">
            <label for="task">Причина обращения</label>
            <input type="text" class="form-control" name="task" id="task" required>
          </div>
          <div class="form-group">
            <label for="note">Цель испытаний и исследований</label>
            <textarea class="form-control" name="note" id="note" rows="3" required></textarea>
          </div>
          <button type="button" class="btn btn-success" onclick="confirmSubmission()">Подтвердить оформление</button>
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Отмена</button>
      </div>
    </div>
  </div>
</div>

<!-- Модальное окно для изменения статуса -->
<div class="modal fade" id="statusModal" tabindex="-1" role="dialog" aria-labelledby="statusModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="statusModalLabel">Изменить статус заявки</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <form id="statusForm">
          <input type="hidden" name="application_id" id="applicationId" value="">
          <div class="form-group">
            <label for="status">Новый статус</label>
            <select class="form-control" name="status" id="statusSelect" required>
              <option value="приостановлено">Приостановлено</option>
              <option value="выполнено">Выполнено</option>
              <option value="отклонено">Отклонено</option>
              <option value="отменено">Отменено</option>
            </select>
          </div>
          <div class="form-group">
            <label for="status_note">Комментарий (необязательно)</label>
            <textarea class="form-control" name="status_note" id="status_note" rows="3"></textarea>
          </div>
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Отмена</button>
        <button type="button" class="btn btn-primary" onclick="updateStatus()">Сохранить</button>
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
		<? 
		$dsn = 'mysql:host=localhost;dbname=test_db';
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
        echo "<td>";
        if ($row['is_editable'] === 1) {
          echo "<button class='btn btn-success btn-sm' onclick='openSubmitModal(" . $row['id'] . ")'>Оформить</button>";
        }
        if ($row['is_editable'] === 2 && $row['lab_id'] == $user_id) {
          echo "<a href='report_form.php?id=" . $row['id'] . "' class='btn btn-info btn-sm'>Отчёт</a>";
          echo "<button class='btn btn-warning btn-sm ml-1' onclick='openStatusModal(" . $row['id'] . ")'>Изменить статус</button>";
        }
        echo "</td>";
			echo "</tr>";
		}
		?>
		</tbody>
		</table>
	</div>
</body>

<script>
// Функция для открытия модального окна оформления заявки
function openSubmitModal(templateId) {
    document.getElementById('templateId').value = templateId;
    fetch('get_template_data.php?id=' + templateId)
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            document.getElementById('research_object').value = data.research_object;
            document.getElementById('initiator').value = data.initiator;
            document.getElementById('task').value = data.task;
            document.getElementById('note').value = data.goal;
            $('#submitModal').modal('show');
        } else {
            alert('Не удалось загрузить данные шаблона.');
        }
    })
    .catch(error => {
        console.error('Ошибка при загрузке данных шаблона:', error);
    });
}

// Функция для открытия модального окна изменения статуса
function openStatusModal(applicationId) {
    document.getElementById('applicationId').value = applicationId;
    $('#statusModal').modal('show');
}

// Функция для подтверждения оформления заявки
function confirmSubmission() {
    const templateId = document.getElementById('templateId').value;
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
            $('#submitModal').modal('hide');
            alert('Заявка успешно оформлена!');
            location.reload();
        } else {
            alert('Ошибка при оформлении заявки.');
        }
    })
    .catch(error => {
        console.error('Ошибка при оформлении:', error);
    });
}

// Функция для обновления статуса заявки
function updateStatus() {
    const applicationId = document.getElementById('applicationId').value;
    const status = document.getElementById('statusSelect').value;
    const note = document.getElementById('status_note').value;
    
    fetch('update_status.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            id: applicationId,
            status: status,
            note: note
        }),
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            $('#statusModal').modal('hide');
            alert('Статус успешно обновлен!');
            location.reload();
        } else {
            alert('Ошибка при обновлении статуса.');
        }
    })
    .catch(error => {
        console.error('Ошибка при обновлении статуса:', error);
    });
}
</script>
```

Вам также понадобятся два новых PHP-файла:

1. `update_status.php` (для обработки изменения статуса):
```php
<?php
header('Content-Type: application/json');

$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';

try {
    $pdo = new PDO($dsn, $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    $data = json_decode(file_get_contents('php://input'), true);
    
    $stmt = $pdo->prepare("UPDATE user_applications SET status = :status, status_note = :note WHERE id = :id");
    $stmt->execute([
        ':status' => $data['status'],
        ':note' => $data['note'],
        ':id' => $data['id']
    ]);
    
    echo json_encode(['success' => true]);
} catch (PDOException $e) {
    echo json_encode(['success' => false, 'error' => $e->getMessage()]);
}
?>
```

2. `get_template_data.php` (если его еще нет):
```php
<?php
header('Content-Type: application/json');

$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';

try {
    $pdo = new PDO($dsn, $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    $id = $_GET['id'];
    
    $stmt = $pdo->prepare("SELECT * FROM user_applications WHERE id = :id");
    $stmt->execute([':id' => $id]);
    $data = $stmt->fetch(PDO::FETCH_ASSOC);
    
    if ($data) {
        echo json_encode([
            'success' => true,
            '
