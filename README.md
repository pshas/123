<?php
require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_before.php');

if (!check_bitrix_sessid()) {
    die(json_encode(['error' => 'Invalid sessid']));
}

global $USER;
$userId = $USER->GetID();

if ($_SERVER['REQUEST_METHOD'] === 'POST' && $userId > 0) {
    $taskId = (int)$_POST['id'];
    
    if ($taskId > 0) {
        $dsn = 'mysql:host=localhost;dbname=test_db';
        $username = 'root';
        $password = 'root';
        
        try {
            $pdo = new PDO($dsn, $username, $password, [PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION]);
            
            $stmt = $pdo->prepare("UPDATE user_applications SET is_editable = 2, lab_id = ? WHERE id = ?");
            $stmt->execute([$userId, $taskId]);
            
            echo json_encode(['success' => true]);
        } catch (PDOException $e) {
            echo json_encode(['error' => 'Database error: ' . $e->getMessage()]);
        }
    } else {
        echo json_encode(['error' => 'Invalid task ID']);
    }
} else {
    echo json_encode(['error' => 'Access denied']);
}

require_once($_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/epilog_after.php');
?>
