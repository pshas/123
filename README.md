$carStmt = $pdo->prepare("SELECT name_car, nom_car, type_car FROM qr_schema.cars WHERE id_car = ?");
    $carStmt->execute([$car_id]);
    $car = $carStmt->fetch(PDO::FETCH_ASSOC);

    if ($car && !empty($car['name_car'])) {
        $car_name = "{$car['name_car']} {$car['nom_car']} ({$car['type_car']})";
    } else {
        $car_name = "Автомобиль #{$car_id}";
    }
