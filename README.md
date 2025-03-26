$stmt = $pdo->prepare("UPDATE user_applications SET is_editable = 2, lab_id = ?, status = 'В работе' ");
