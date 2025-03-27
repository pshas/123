Вот стилизованная версия страницы `report_form.php` с современным дизайном:

```php
<?
require("../header.php");

$application_id = intval($_GET['id']);

$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';
$pdo = new PDO($dsn, $username, $password);

$stmt = $pdo->prepare("SELECT * FROM user_applications WHERE id = ?");
$stmt->execute([$application_id]);
$application = $stmt->fetch(PDO::FETCH_ASSOC);

if (!$application) {
    die("Заявка не найдена");
}
?>

<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Форма отчёта - Заявка №<?= $application['id'] ?></title>
    
    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    
    <!-- Custom Styles -->
    <style>
        :root {
            --primary-color: #3498db;
            --secondary-color: #2980b9;
            --accent-color: #e74c3c;
            --light-gray: #f8f9fa;
            --dark-gray: #343a40;
        }
        
        body {
            background-color: #f5f5f5;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        .report-container {
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
            padding: 30px;
            margin-top: 30px;
            margin-bottom: 50px;
        }
        
        .report-header {
            border-bottom: 2px solid var(--primary-color);
            padding-bottom: 15px;
            margin-bottom: 30px;
        }
        
        .report-title {
            color: var(--dark-gray);
            font-weight: 600;
        }
        
        .application-number {
            color: var(--accent-color);
            font-weight: bold;
        }
        
        .form-section {
            margin-bottom: 25px;
            padding: 20px;
            background-color: var(--light-gray);
            border-radius: 8px;
        }
        
        .section-title {
            color: var(--secondary-color);
            margin-bottom: 15px;
            font-size: 1.1rem;
            font-weight: 500;
        }
        
        .readonly-field {
            background-color: #e9ecef;
            cursor: not-allowed;
        }
        
        textarea {
            min-height: 100px;
            resize: vertical;
        }
        
        .btn-submit {
            background-color: var(--primary-color);
            border: none;
            padding: 10px 25px;
            font-weight: 500;
            transition: all 0.3s;
        }
        
        .btn-submit:hover {
            background-color: var(--secondary-color);
            transform: translateY(-2px);
        }
        
        .info-badge {
            background-color: var(--primary-color);
            color: white;
            padding: 5px 10px;
            border-radius: 5px;
            font-size: 0.8rem;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="report-container">
            <div class="report-header">
                <h1 class="report-title">Форма отчёта <span class="application-number">№<?= htmlspecialchars($application['id']) ?></span></h1>
                <div class="d-flex justify-content-between align-items-center">
                    <div>
                        <span class="info-badge">Статус: <?= htmlspecialchars($application['status']) ?></span>
                    </div>
                    <div>
                        <span class="text-muted">Дата создания: <?= htmlspecialchars($application['created_at']) ?></span>
                    </div>
                </div>
            </div>
            
            <form method="POST" action="save_report.php">
                <input type="hidden" name="application_id" value="<?= $application['id'] ?>">
                
                <!-- Основная информация -->
                <div class="form-section">
                    <h3 class="section-title">Основная информация</h3>
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Объект исследования</label>
                            <input type="text" class="form-control readonly-field" 
                                   value="<?= htmlspecialchars($application['NAME_IZD']) ?>" readonly>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Инициатор</label>
                            <input type="text" class="form-control readonly-field" 
                                   value="<?= htmlspecialchars($application['full_name']) ?>" readonly>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Цех/Отдел</label>
                            <input type="text" class="form-control readonly-field" 
                                   value="<?= htmlspecialchars($application['work_department']) ?>" readonly>
                        </div>
                        <div class="col-md-6 mb-3">
                            <label class="form-label">Несоответствие</label>
                            <input type="text" class="form-control readonly-field" 
                                   value="<?= htmlspecialchars($application['task']) ?>" readonly>
                        </div>
                    </div>
                </div>
                
                <!-- Цель испытания -->
                <div class="form-section">
                    <h3 class="section-title">Цель испытания</h3>
                    <textarea class="form-control readonly-field" readonly><?= htmlspecialchars($application['note']) ?></textarea>
                </div>
                
                <!-- Результаты исследования -->
                <div class="form-section">
                    <h3 class="section-title">Результаты исследования</h3>
                    <textarea class="form-control" name="results" id="results" rows="5" required
                              placeholder="Опишите подробно результаты проведённых исследований..."></textarea>
                </div>
                
                <!-- Описание -->
                <div class="form-section">
                    <h3 class="section-title">Описание</h3>
                    <textarea class="form-control" name="description" id="description" rows="5" required
                              placeholder="Детальное описание процесса исследования..."></textarea>
                </div>
                
                <!-- Выводы -->
                <div class="form-section">
                    <h3 class="section-title">Выводы</h3>
                    <textarea class="form-control" name="conclusion" id="conclusion" rows="5" required
                              placeholder="Сформулируйте основные выводы по результатам исследования..."></textarea>
                </div>
                
                <div class="d-flex justify-content-end mt-4">
                    <a href="applications.php" class="btn btn-outline-secondary me-3">Назад</a>
                    <button type="submit" class="btn btn-submit">
                        <i class="bi bi-check-circle"></i> Сохранить отчёт
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Bootstrap 5 JS Bundle with Popper -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    
    <!-- Bootstrap Icons -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
</body>
</html>
```

### Ключевые особенности стилей:

1. **Современный дизайн**:
   - Плавные тени и скругленные углы
   - Четкое разделение на секции
   - Акцентные цвета для важных элементов

2. **Улучшенная типографика**:
   - Четкие заголовки секций
   - Хорошая читаемость текста
   - Правильные отступы

3. **Адаптивная верстка**:
   - Корректное отображение на всех устройствах
   - Гибкая сетка (колонки на больших экранах, стекирование на мобильных)

4. **Интерактивные элементы**:
   - Плавные hover-эффекты для кнопок
   - Визуальная обратная связь при взаимодействии

5. **Семантическое оформление**:
   - Четкое разделение информации на логические блоки
   - Визуальное отличие редактируемых и readonly полей

6. **Дополнительные визуальные элементы**:
   - Бейджи для статусов
   - Иконки для кнопок
   - Подсветка важной информации

Для полной функциональности вам также понадобится:
1. Файл `save_report.php` для обработки формы
2. Возможно, JavaScript для валидации перед отправкой
3. Настроенные права доступа к странице

Хотите, добавлю какие-нибудь дополнительные элементы дизайна или функциональности?
