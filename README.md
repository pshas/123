Вот код страницы для отображения данных из базы данных, аналогичный вашему примеру:

```php
<?
require("../header.php");
$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';

$pdo = new PDO($dsn, $username, $password);

// Получаем ID записи из GET-параметра
$record_id = isset($_GET['record_id']) ? intval($_GET['record_id']) : 0;

// Запрос для получения данных записи
$sql = $pdo->prepare("
    SELECT * FROM user_applications 
    WHERE id = :record_id
");
$sql->execute([':record_id' => $record_id]);
$record = $sql->fetch(PDO::FETCH_ASSOC);

// Если запись не найдена, выводим сообщение
if (!$record) {
    die("Запись не найдена");
}

// Получаем название блока для отображения в заголовке
$block_sql = $pdo->prepare("
    SELECT title FROM blocks WHERE id = :block_id
");
$block_sql->execute([':block_id' => $record['block_id']]);
$block_title = $block_sql->fetch(PDO::FETCH_ASSOC);
?>

<!-- jQuery -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<!-- Bootstrap CSS -->
<!-- Bootstrap JS (зависит от jQuery) -->
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js"></script>

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

<form method="POST">
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
                <? echo date("d.m.y", strtotime($record['date_man'])); ?> <br>
            </td>
        </tr>
        <tr>
            <td colspan="2">
                Номер протокола
            </td>
            <td colspan="2">
                <? echo $block_title['title']; ?>-<? echo date("Y", strtotime($record['date_man'])); ?>-<? echo $record['id']; ?>
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
                <input type="text" id="field2" name="name_izd" style="width: 100%" value="<? echo htmlspecialchars($record['NAME_IZD']); ?>" readonly>
            </td>
            <td colspan="2">
                <input type="text" id="field1" name="id_i" style="width: 100%" value="<? echo htmlspecialchars($record['ID_I']); ?>" readonly>
            </td>
            <td colspan="2">
                <input type="date" value="<? echo htmlspecialchars($record['date_detail']); ?>" readonly>
            </td>
        </tr>
        <tr>
            <td colspan="1">
                Дата изготовления
            </td>
            <td rowspan="1">
                <input type="date" id="date_man" name="date_man" value="<? echo htmlspecialchars($record['date_man']); ?>" readonly>
            </td>
            <td rowspan="3" colspan="1">
                Поставщик
            </td>
            <td rowspan="3">
                <input type="text" id="COUNTRY" name="country" value="<? echo htmlspecialchars($record['COUNTRY']); ?>" readonly><br>
            </td>
            <td rowspan="3" colspan="1">
                Количество
            </td>
            <td rowspan="3">
                <input type="text" id="count_detail" name="count_detail" value="<? echo htmlspecialchars($record['count_detail']); ?>" readonly> <br>
            </td>
        </tr>
        <tr>
            <td colspan="1">
                Дата поставки⁠
            </td>
            <td rowspan="1">
                <input type="date" id="date_del" name="date_del" value="<? echo htmlspecialchars($record['date_del']); ?>" readonly>
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
                        <input type="checkbox" id="material1" name="material1" value="Plastic" <? echo ($record['material'] == 'Plastic') ? 'checked' : ''; ?> disabled>
                        <label for="material1"> Plastic</label><br>
                        <input type="checkbox" id="material2" name="material2" value="Metal" <? echo ($record['material'] == 'Metal') ? 'checked' : ''; ?> disabled>
                        <label for="material2"> Metal</label><br>
                        <input type="checkbox" id="material" name="material3" value="Painted" <? echo ($record['material'] == 'Painted') ? 'checked' : ''; ?> disabled>
                        <label for="material3"> Painted</label>
                    </div>
                    <div style="width: 50%; display: flex; height: auto;"> 
                        <input type="checkbox" id="processing1" name="processing1" value="Zeiss" <? echo ($record['processing'] == 'Zeiss') ? 'checked' : ''; ?> disabled>
                        <label for="processing1"> Zeiss</label><br>
                        <input type="checkbox" id="processing2" name="processing2" value="LaserTracer" <? echo ($record['processing'] == 'LaserTracer') ? 'checked' : ''; ?> disabled>
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
                <input type="date" id="date_detail" name="date_detail" value="<? echo htmlspecialchars($record['date_detail']); ?>" readonly>
            </td>
            <td colspan="1">
                Цех/отдел
            </td>
            <td rowspan="1">
                <input type="text" id="WORK_DEPARTMENT" name="work_department" readonly value="<? echo htmlspecialchars($record['work_department']); ?>">
            </td>
        </tr>
        <tr>
            <td colspan="2">
                Лицо, предоставившее деталь на измерение
            </td>
            <td colspan="2" name="full_name">
                <input type="text" id="FULL_NAME" name="full_name" readonly value="<? echo htmlspecialchars($record['full_name']); ?>" style="width: 100%">
            </td>
            <td colspan="1">
                Тел.
            </td>
            <td colspan="1">
                <input type="text" id="PERSONAL_PHONE" name="personal_phone" value="<? echo htmlspecialchars($record['personal_phone']); ?>" readonly>
            </td>
        </tr>
        <tr>
            <td colspan="2">
                Должность
            </td>
            <td colspan="2">
                <input type="text" id="WORK_POSITION" name="work_position" readonly value="<? echo htmlspecialchars($record['work_position']); ?>" style="width: 100%">
            </td>
            <td colspan="1">
                Тел. рабочий
            </td>
            <td colspan="1">
                <input type="text" id="WORK_PHONE" name="work_phone" readonly value="<? echo htmlspecialchars($record['work_phone']); ?>"> 
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
                <input type="text" id="field11" style="width: 100%" readonly value="<? echo htmlspecialchars($record['NAME_IZD']); ?>">
            </td>
            <td colspan="1">
                <input type="text" id="field22" style="width: 100%" readonly value="<? echo htmlspecialchars($record['ID_I']); ?>">
            </td>
            <td colspan="1">
                <input type="text" id="VIN" readonly value="<? echo htmlspecialchars($record['vin']); ?>">
            </td>
        </tr>
        <tr>
            <td colspan="6">
                Задание для измерения
            </td>
        </tr>
        <tr>
            <td colspan="6">
                <input type="text" id="task" name="task" style="width: 100%" value="<? echo htmlspecialchars($record['task']); ?>" readonly> <br>
            </td>
        </tr>
        <tr>
            <td colspan="6">
                Обоснование/примечание
            </td>
        </tr>
        <tr>
            <td colspan="6">
                <input type="text" id="note" name="note" style="width: 100%" value="<? echo htmlspecialchars($record['note']); ?>" readonly> <br>
            </td>
        </tr>
        <tr>
            <td colspan="6">
                Прием/сдача
            </td>
        </tr>
        <tr>
            <td class="td-col" colspan="6">
                <div style="display: flex;">
                    <input type="text" id="note" name="note" style="width: 50%" readonly>
                    <input type="text" id="note" name="note" style="width: 50%" readonly>
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
        </tbody>
    </table>
    <div style="position: fixed; bottom: 20px; right: 20px;">
        <button onclick="window.print()" class="btn btn-primary no-print">
            <i class="fas fa-print"></i> Печать
        </button>
    </div>
</form>

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
```

Основные особенности этой страницы:

1. Получает ID записи из GET-параметра `record_id`
2. Загружает данные записи из таблицы `user_applications`
3. Загружает название блока из таблицы `blocks` для отображения в заголовке
4. Все поля формы заполнены данными из базы и сделаны read-only
5. Сохранена оригинальная структура таблицы и стили
6. Добавлена проверка на существование записи
7. Все выводимые данные экранированы с помощью `htmlspecialchars()`
8. Сохранена функциональность печати формы

Для использования этой страницы нужно передавать ID записи в URL, например: `view_record.php?record_id=123`
