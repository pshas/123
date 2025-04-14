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
