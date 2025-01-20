<?
require("../header.php");
$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';

$pdo = new PDO($dsn, $username, $password);
$title = "";
$block_id = isset($_GET['block_id']) ? intval($_GET['block_id']) : 0;
$sql = $pdo->prepare("
SELECT id, title FROM blocks WHERE id = :block_id
");
$sql->execute([':block_id' => $block_id]);
$title = $sql->fetch(PDO::FETCH_ASSOC);

$user_id = $USER->GetId();
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['save_template'])) {
	$country = $_POST['country'];
	$name_izd = $_POST['name_izd'];
	$id_i = $_POST['id_i'];
	$work_department = $_POST['work_department'];
	$full_name = $_POST['full_name'];
	$personal_phone = $_POST['personal_phone'];
	$work_position = $_POST['work_position'];
	$work_phone = $_POST['work_phone'];
	$is_editable = isset($_POST['is_editable']) ? (int)$_POST['is_editable'] : 1;
	$task = $_POST['task'];
	$note = $_POST['note'];
	$status = "Сохранен";
	$stmt = $pdo->prepare("
	INSERT INTO user_applications (block_id, USER_ID, NAME_IZD, ID_I, COUNTRY, work_department, full_name, personal_phone, work_position, work_phone, is_editable, task, note, status)
VALUES (:block_id, :USER_ID, :NAME_IZD, :ID_I, :COUNTRY, :work_department, :full_name, :personal_phone, :work_position, :work_phone, :is_editable, :task, :note, :status)
	");
	$stmt->execute([
	':block_id' => $block_id,
	':USER_ID' => $user_id,
	':NAME_IZD' => $name_izd,
	':ID_I' => $id_i,
	':COUNTRY' => $country,
	':work_department' => $work_department,
	':full_name' => $full_name,
	':personal_phone' => $personal_phone,
	':work_position' => $work_position,
	':work_phone' => $work_phone,
	':is_editable' => $is_editable,
	':task' => $task,
	':note' => $note,
	':status' => $status
	]);
}
?>
<!-- jQuery -->

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

<!-- Bootstrap CSS -->
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">

<!-- Bootstrap JS (зависит от jQuery) -->
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>

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
<form method="POST">
 <button type="submit" name="save_template">Сохранить</button>
	<table class="form">
	<tbody>
	<tr>
		<td rowspan="2" colspan="2">
			 Заявка на измерение<br>
		</td>
		<td rowspan="1" colspan="2">
			 Дата
		</td>
		<td rowspan="1" colspan="2">
			 <? echo date("d.m.y"); ?> <br>
		</td>
	</tr>
	<tr>
		<td colspan="2">
			 Номер протокола
		</td>
		<td colspan="2">
			 <? echo $title['title']; ?>-<? echo date("Y"); ?>-id
		</td>
	</tr>
	<tr>
		<td colspan="2" style="width:100px">
			 Наименование
		</td>
		<td colspan="2">
			 Номер детали
		</td>
		<td colspan="2">
			 Дата предоставления на замер
		</td>
	</tr>
	<tr>
		<td colspan="2">
 <label for="NAME_IZD"></label> <input type="text" id="field2" name="name_izd">
	<div id="suggestions2" class="suggestions"></div>
		</td>
		<td colspan="2">
 <label for="id_i"></label> <input type="text" id="field1" name="id_i">
	<div id="suggestions1" class="suggestions"></div>
		</td>
		<td colspan="2">
			 <input type="date" id="date" name="date">
		</td>
	</tr>
	<tr>
		<td colspan="1">
			 Дата изготовления
		</td>
		<td rowspan="1">
			<input type="date" id="date" name="date">
		</td>
		<td rowspan="2" colspan="1">
			 Поставщик
		</td>
		<td rowspan="2">
 <label for="COUNTRY"></label> <input type="text" id="COUNTRY" name="country" readonly=""><br>
		</td>
		<td rowspan="2" colspan="1">
			 Количество
		</td>
		<td rowspan="2">
			 <label for="count"></label> <input type="text" id="count" name="count"> <br>
		</td>
	</tr>
	<tr>
		<td colspan="1">
			Дата поставки⁠
		</td>
		<td rowspan="1">
			<input type="date" id="date" name="date">
		</td>
	</tr>
	<tr>
		<td colspan="2">
			 Примечание
		</td>
		<td colspan="4">
			 Заполняется сотрудником лаборатории
		</td>
	</tr>
	<tr>
		<td colspan="6">
			 Протокол поставки
		</td>
	</tr>
	<tr>
		<td colspan="2">
			Дата предоставление детали
		</td>
		<td colspan="2">
			<input type="date" id="date" name="date">
		</td>
		<td colspan="1">
			 Цех/отдел
		</td>
		<td rowspan="1">
 <input type="text" id="WORK_DEPARTMENT" name="work_department" readonly="" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["WORK_DEPARTMENT"]; ?>">
		</td>
	</tr>
	<tr>
		<td colspan="2">
			 Лицо, предоставившее деталь на измерение
		</td>
		<td colspan="2" name="full_name">
 <input type="text" id="FULL_NAME" name="full_name" readonly="" value="<? echo $USER->GetFullName();?>">
		</td>
		<td colspan="1">
			 Тел.
		</td>
		<td colspan="1">
 <input type="text" id="PERSONAL_PHONE" name="personal_phone" readonly="" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["PERSONAL_PHONE"]; ?>">
		</td>
	</tr>
	<tr>
		<td colspan="2">
			 Должность
		</td>
		<td colspan="2">
 <input type="text" id="WORK_POSITION" name="work_position" readonly="" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["WORK_POSITION"]; ?>">
		</td>
		<td colspan="1">
			 Тел. рабочий
		</td>
		<td colspan="1">
			<input type="text" id="WORK_PHONE" name="work_phone" readonly="" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["WORK_PHONE"]; ?>"> 
		</td>
	</tr>
	<tr>
		<td colspan="6">
			 Предоставленные детали
		</td>
	</tr>
	<tr>
		<td colspan="4">
			 Номер детали
		</td>
		<td rowspan="1">
			 Наименование
		</td>
		<td rowspan="1">
			 VIN/Шасси (#CAD Model)
		</td>
	</tr>
	<tr>
		<td colspan="4">
			#id_i
		</td>
		<td colspan="1">
			#NAME_IZD
		</td>
		<td colspan="1">
 <br>
		</td>
	</tr>
	<tr>
		<td colspan="4">
		</td>
		<td colspan="1">
 <br>
		</td>
		<td colspan="1">
 <br>
		</td>
	</tr>
	<tr>
		<td colspan="4">
 <br>
		</td>
		<td colspan="1">
 <br>
		</td>
		<td colspan="1">
 <br>
		</td>
	</tr>
	<tr>
		<td colspan="6">
			 Задание для измерения
		</td>
	</tr>
	<tr>
		<td colspan="6">
			<input type="text" id="task" name="task"> <br>
		</td>
	</tr>
	<tr>
		<td colspan="6">
 <br>
		</td>
	</tr>
	<tr>
		<td colspan="6">
			 Обоснование/примечание
		</td>
	</tr>
	<tr>
		<td colspan="6">
			<input type="text" id="note" name="note"> <br>
		</td>
	</tr>
	<tr>
		<td colspan="6">
 <br>
		</td>
	</tr>
	<tr>
		<td colspan="6">
			 Прием/сдача
		</td>
	</tr>
	</tbody>
	</table>
</form>
