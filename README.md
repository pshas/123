<?php require("../header.php"); ?>

<!-- Bootstrap CSS -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<!-- jQuery и Bootstrap JS -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>

<body>
	<div class="container">
		<h1 class="mt-4 mb-4">Задачи</h1>
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
			</thead>
			<tbody>
				<?php
				$dsn = 'mysql:host=localhost;dbname=test_db';
				$username = 'root';
				$password = 'root';
				$pdo = new PDO($dsn, $username, $password, [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);

				$stmt = $pdo->prepare("SELECT * FROM user_applications WHERE is_editable = 0");
				$stmt->execute();

				while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
					echo "<tr>";
					echo "<td>" . htmlspecialchars($row['id']) . "</td>";
					echo "<td>" . htmlspecialchars($row['created_at']) . "</td>";
					echo "<td>" . htmlspecialchars($row['full_name']) . "</td>";
					echo "<td>" . htmlspecialchars($row['work_department']) . "</td>";
					echo "<td>" . htmlspecialchars($row['NAME_IZD']) . "</td>";
					echo "<td>" . htmlspecialchars($row['ID_I']) . "</td>";
					echo "<td>" . htmlspecialchars($row['date_man'] ?? '—') . "</td>";
					echo "<td>" . htmlspecialchars($row['batch_number'] ?? '—') . "</td>";
					echo "<td>" . htmlspecialchars($row['task']) . "</td>";
					echo "<td>" . htmlspecialchars($row['status']) . "</td>";
					echo "<td><button class='btn btn-primary take-work' data-id='" . htmlspecialchars($row['id']) . "'>Взять в работу</button></td>";
					echo "</tr>";
				}
				?>
			</tbody>
		</table>
	</div>

	<!-- Модальное окно -->
	<div class="modal fade" id="workModal" tabindex="-1" aria-hidden="true">
		<div class="modal-dialog">
			<div class="modal-content">
				<div class="modal-header">
					<h5 class="modal-title">Взять в работу</h5>
					<button type="button" class="btn-close" data-bs-dismiss="modal"></button>
				</div>
				<div class="modal-body">
					<form id="workForm">
						<input type="hidden" id="task_id">
						<div class="mb-3"><label class="form-label">Инициатор</label><input type="text" id="initiator" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Цех/Отдел</label><input type="text" id="department" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Наименование</label><input type="text" id="name" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Номер детали</label><input type="text" id="detail" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Дата изготовления</label><input type="text" id="date" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Номер партии</label><input type="text" id="batch" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Несоответствие</label><input type="text" id="task" class="form-control" readonly></div>
						<div class="mb-3"><label class="form-label">Исполнитель</label><input type="text" id="executor" class="form-control"></div>
						<div class="mb-3"><label class="form-label">Результаты исследования</label><textarea id="results" class="form-control"></textarea></div>
						<div class="mb-3"><label class="form-label">Описание</label><textarea id="description" class="form-control"></textarea></div>
						<div class="mb-3"><label class="form-label">Вывод</label><textarea id="conclusion" class="form-control"></textarea></div>
						<button type="submit" class="btn btn-success">Сохранить</button>
					</form>
				</div>
			</div>
		</div>
	</div>

	<script>
		$(document).ready(function() {
			$(".take-work").click(function() {
				let taskId = $(this).data("id");

				$.ajax({
					url: "get_task.php",
					type: "POST",
					data: { id: taskId },
					dataType: "json",
					success: function(response) {
						$("#task_id").val(response.id);
						$("#initiator").val(response.full_name);
						$("#department").val(response.work_department);
						$("#name").val(response.NAME_IZD);
						$("#detail").val(response.ID_I);
						$("#date").val(response.date_man);
						$("#batch").val(response.batch_number);
						$("#task").val(response.task);
						$("#workModal").modal("show");
					}
				});
			});

			$("#workForm").submit(function(event) {
				event.preventDefault();

				let data = {
					task_id: $("#task_id").val(),
					executor: $("#executor").val(),
					results: $("#results").val(),
					description: $("#description").val(),
					conclusion: $("#conclusion").val()
				};

				console.log("Отправка данных:", data);
				$("#workModal").modal("hide");
			});
		});
	</script>
</body>
