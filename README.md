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
	$count_detail = $_POST['count_detail'];
	$stmt = $pdo->prepare("
	INSERT INTO user_applications (block_id, USER_ID, NAME_IZD, ID_I, COUNTRY, work_department, full_name, personal_phone, work_position, work_phone, is_editable, task, note, status, count_detail, date_man, date_del, date_detail)
VALUES (:block_id, :USER_ID, :NAME_IZD, :ID_I, :COUNTRY, :work_department, :full_name, :personal_phone, :work_position, :work_phone, :is_editable, :task, :note, :status, :count_detail, :date_man, :date_del, :date_detail)
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
	':status' => $status,
	':count_detail' => $count_detail,
	':date_man' => $date_man,
	':date_del' => $date_del,
	':date_detail' => $date_detail
	]);
}
?> <!-- jQuery --> <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script> <!-- Bootstrap CSS --> <!-- Bootstrap JS (зависит от jQuery) --> <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script> <style>
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
		<td colspan="2">
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
  			<input type="text" id="field2" name="name_izd" style="width: 100%">
	<div id="suggestions2" class="suggestions"></div>
		</td>
		<td colspan="2">
 			<input type="text" id="field1" name="id_i" style="width: 100%">
	<div id="suggestions1" class="suggestions"></div>
		</td>
		<td colspan="2">

		</td>
	</tr>
	<tr>
		<td colspan="1">
			 Дата изготовления
		</td>
		<td rowspan="1">
			<input type="date" id="date_man" name="date_man">
		</td>
		<td rowspan="3" colspan="1">
			 Поставщик
		</td>
		<td rowspan="3">
			<input type="text" id="COUNTRY" name="country"><br>
		</td>
		<td rowspan="3" colspan="1">
			 Количество
		</td>
		<td rowspan="3">
			 <label for="count"></label> <input type="text" id="count_detail" name="count_detail"> <br>
		</td>
	</tr>
	<tr>
		<td colspan="1">
			Дата поставки⁠
		</td>
		<td rowspan="1">
			<input type="date" id="date_del" name="date_del">
		</td>
	</tr>
	<tr>
		<td>
			Номер партии
		</td>
		<td>
			&nbsp;
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
		<td colspan="2">
			&nbsp;
		</td>
		<td colspan="4" style="padding: 0px;">
			<div style="display: flex; margin: 0px;">
				<div style="width: 50%; display: flex; height: auto;"> 
					<input type="checkbox" id="material1" name="material1" value="Plastic">
  					<label for="material1"> Plastic</label><br>
  					<input type="checkbox" id="material2" name="material2" value="Metal">
  					<label for="material2"> Metal</label><br>
  					<input type="checkbox" id="material" name="material3" value="Painted">
  					<label for="material3"> Painted</label>
				</div>
				<div style="width: 50%; display: flex; height: auto;"> 
					<input type="checkbox" id="processing1" name="processing1" value="Zeiss">
  					<label for="processing1"> Zeiss</label><br>
  					<input type="checkbox" id="processing2" name="processing2" value="LaserTracer">
  					<label for="processing2"> LaserTracer</label><br>
				</div>
			</div>
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
			<input type="date" id="date_detail" name="date_detail">
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
 <input type="text" id="FULL_NAME" name="full_name" readonly="" value="<? echo $USER->GetFullName();?>" style="width: 100%">
		</td>
		<td colspan="1">
			 Тел.
		</td>
		<td colspan="1">
 <input type="text" id="PERSONAL_PHONE" name="personal_phone" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["PERSONAL_PHONE"]; ?>">
		</td>
	</tr>
	<tr>
		<td colspan="2">
			 Должность
		</td>
		<td colspan="2">
 <input type="text" id="WORK_POSITION" name="work_position" readonly="" value="<? $arUser = CUser::GetByID($GLOBALS["USER"]->GetId())->GetNext(); echo $arUser["WORK_POSITION"]; ?>" style="width: 100%">
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
			 Наименование
		</td>
		<td rowspan="1">
			 Номер детали
		</td>
		<td rowspan="1">
			 VIN/Шасси (#CAD Model)
		</td>
	</tr>
	<tr>
		<td colspan="4">
			<input type="text" id="field11" style="width: 100%" readonly="">
		</td>
		<td colspan="1">
			<input type="text" id="field22" style="width: 100%" readonly="">
		</td>
		<td colspan="1">
			<input type="text" id="VIN">
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
			<input type="text" id="task" name="task" style="width: 100%"> <br>
		</td>
	</tr>
	<tr>
	</tr>
	<tr>
		<td colspan="6">
			 Обоснование/примечание
		</td>
	</tr>
	<tr>
		<td colspan="6">
			<input type="text" id="note" name="note" style="width: 100%"> <br>
		</td>
	</tr>
	<tr>
	</tr>
	<tr>
		<td colspan="6">
			 Прием/сдача
		</td>
	</tr>
	<tr>
		<td class="td-col" colspan="6">
			<div style="display: flex;">
				<input type="text" id="note" name="note" style="width: 50%">
				<input type="text" id="note" name="note" style="width: 50%">
			</div>
		</td>
	</tr>
	<tr>
		<td colspan="3">
			Время измерения
		</td>
		<td rowspan="1" colspan="2">
			Сотрудник ОИиУК
		</td>
		<td colspan="1">
			Подпись
		</td>
	</tr>
	<tr>
		<td colspan="3">
			&nbsp;
		</td>
		<td rowspan="1" colspan="2">
			&nbsp; &nbsp;
		</td>
		<td colspan="1">
			&nbsp;
		</td>
	</tr>
	</tbody>
	</table>
	<div style="position: fixed; bottom: 20px; right: 20px;">
    <button onclick="window.print()" class="btn btn-primary no-print">
        <i class="fas fa-print"></i> Печать
    </button>
</div>
</form>
<script src="form.js"></script>

<style> 
        .form, .form * {
            visibility: visible;
        }
        .form {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
        }
        
        /* Улучшение отображения таблицы при печати */
        table.form {
            border-collapse: collapse;
            width: 35%;
			height: 40%;
        }
        table.form td, table.form th {
            border: 1px solid #000;
            padding: 4px;
        }

    @media print {
        body * {
            visibility: hidden;
        }
        .form, .form * {
            visibility: visible;
        }
        .form {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
        }
        .no-print {
            display: none !important;
        }
        
        /* Улучшение отображения таблицы при печати */
        table.form {
            border-collapse: collapse;
            width: 100%;
        }
        table.form td, table.form th {
            border: 1px solid #000;
            padding: 8px;
        }
    }
    
    /* Стиль для кнопки печати */
    .print-btn {
        position: fixed;
        bottom: 20px;
        right: 20px;
        z-index: 1000;
    }
</style>

<script>
function printForm() {
    // Скрыть лишние элементы
    document.querySelectorAll('.no-print').forEach(el => {
        el.style.display = 'none';
    });
    
    window.print();
    
    // Показать обратно после печати
    setTimeout(() => {
        document.querySelectorAll('.no-print').forEach(el => {
            el.style.display = '';
        });
    }, 500);
}
</script>
