<?
$dsn = 'mysql:host=localhost;dbname=test_db';
$username = 'root';
$password = 'root';

try {
    $pdo = new PDO($dsn, $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Ошибка подключения к базе данных: " . $e->getMessage());
}

// Проверка входных данных
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Получаем данные из тела запроса
    $input = $_POST['query'] ?? '';
    $fieldId = $_POST['fieldId'] ?? '';

    // Проверяем, что запрос не пуст
    if (!empty($input) && !empty($fieldId)) {
        $column = $fieldId === '1' ? 'MODEL' : 'NAME_IZD'; // Определяем колонку для поиска

        // SQL-запрос с использованием LIKE
        $stmt = $pdo->prepare("SELECT * FROM test WHERE $column LIKE :query LIMIT 100");
        $stmt->execute(['query' => '%' . $input . '%']);

        // Получаем результаты
        $results = $stmt->fetchAll(PDO::FETCH_ASSOC);

        // Возвращаем результаты в формате JSON
        header('Content-Type: application/json');
        echo json_encode($results);
    } else {
        // Если запрос пустой, возвращаем пустой массив
        header('Content-Type: application/json');
        echo json_encode([]);
    }
} else {
    // Если метод не POST, возвращаем ошибку
    http_response_code(405);
    echo json_encode(['error' => 'Method not allowed']);
}
?>
