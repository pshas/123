<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	<link rel="stylesheet" href="/local/qr-opros/src/styles.css">
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-sRIl4kxILFvY47J16cr9ZwB07vP4J8+LH7qKQnuqkuIAvNWLzeN8tE5YBujZqJLB" crossorigin="anonymous">
    <title>Document</title>
</head>

<?php
//require($_SERVER["DOCUMENT_ROOT"]."/bitrix/header.php");
include "/local/qr-opros/phpqrcode/qrlib.php";

?>
<script src="/local/qr-opros/jquery-3.6.0.min.js"></script>
<script src="/local/qr-opros/load_blocks.php"></script>

<body>
<div>

</div>
<div id='content'>
<div id="blocks-container">
</div>
	<button id="addBlock">Добавить автомобиль</button>
		<div id="modal" style="display: none;">
			<h3>Добавить автомобиль</h3>
 			<label for="nameCar">Марка транспортного средства</label> <input type="text" id="nameCar"><br>
 			<br>
 			<label for="typeCar">Тип автомобиля</label> <input type="text" id="typeCar"><br>
 			<br>
 			<label for="nomCar">Регистрационный знак автомобиля</label> <input type="text" id="nomCar"><br>
 			<br>
 			<button id="saveBlock">Сохранить</button> <button id="cancel">Отмена</button>
		</div>
		<script>
			$(document).ready(function() {
				function loadBlocks() {
					$.ajax({
						url: 'load_blocks.php',
						method: 'GET',
						success: function(data) {
							$('#blocks-container').html(data);
						}
					});
				}

			loadBlocks();

		document.getElementById("print-btn").addEventListener("click", function() {
		document.body.style.visibility = "hidden";
		let printBtn = document.getElementById("printarea");
		printBtn.style.visibility = "visible";
		window.print();
		document.body.style.visibility = "visible";
	});



			$('#addBlock').click(function() {
				$('#modal').show();
				$(this).hide();
			});

			$('#saveBlock').click(function() {
				var nameCar =$('#nameCar').val();
				var typeCar =$('#typeCar').val();
				var nomCar =$('#nomCar').val();

			$.ajax({
				url: 'add_block.php',
				method: 'POST',
				data: { nameCar: nameCar, typeCar: typeCar, nomCar: nomCar},
				success: function(blockData) {
					console.log(blockData);
					var block = JSON.parse(blockData);
					$('#blocks-container').append(`
						<div class="block" car-id="${block.id}"> 
							<h3>${block.id}</h3>
							<h4>${block.nameCar}</h4>
							<p>${block.typeCar}</p>
							<p>${block.nomCar}</p>
							<a href="application/index.php?block_id=${block.id}" class="order-btn">Скачать qr</a><br />
							<a href="application/index.php?block_id=${block.id}" class="order-btn">Тест отзыв</a><br />
							<div class="remove-btn">Деактивировать</div>
						</div>
					`);
					$('#modal').hide();
					$('#blockTitle').val('');
					$('#blockDescription').val('');
					$('#addBlock').show();
				}
			});
		});

		$('#cancel').click(function() {
			$('#modal').hide();
			$('#blockTitle').val('');
			$('#blockDescription').val('');
			$('#addBlock').show();
		});

		$(document).on('click', '.remove-btn', function() {
			const block = $(this).closest('.block');
			const blockId = block.data('id');

			$.ajax({
				url: 'remove_block.php',
				method: 'POST',
				data: { id: blockId },
				success: function() {
					block.remove();
				}
			});
		});
	});
	</script>

<?php
$blockId = isset($_GET['block_id']) ? intval($_GET['block_id']) : 0;
		if ($blockId <= 0) {
			return;
		}


/*
$APPLICATION->IncludeComponent(
		"qr_opros", // Папка /local/components/qr_opros
		"",
		[
			"BLOCK_ID" => $blockId;
			"FORM_ID" => "5678"
		],
		false
	);
*/
?>
