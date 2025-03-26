<?php 
require("../header.php"); 
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_before.php');

// Получаем ID текущего пользователя Битрикс
global $USER;
$currentUserId = $USER->GetID();
?>

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

    <script>
        $(document).ready(function() {
            $(".take-work").click(function() {
                let taskId = $(this).data("id");
                let button = $(this);
                
                // Блокируем кнопку на время выполнения запроса
                button.prop('disabled', true);
                
                $.ajax({
                    url: "update_task.php",
                    type: "POST",
                    data: { 
                        id: taskId,
                        sessid: BX.bitrix_sessid() // Добавляем sessid для проверки в Битриксе
                    },
                    dataType: "json",
                    success: function(response) {
                        if (response.success) {
                            // Меняем кнопку на "В работе" и делаем ее неактивной
                            button.text('В работе');
                            button.removeClass('btn-primary').addClass('btn-secondary');
                            
                            // Можно обновить строку или всю таблицу
                            // location.reload(); // если нужно обновить всю страницу
                        } else {
                            alert('Ошибка: ' + (response.error || 'Не удалось обновить задачу'));
                            button.prop('disabled', false);
                        }
                    },
                    error: function() {
                        alert('Ошибка соединения с сервером');
                        button.prop('disabled', false);
                    }
                });
            });
        });
    </script>
</body>
