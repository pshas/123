<head>
    <?php
    // Вставляем заголовки сайта
    $APPLICATION->ShowHead();
    ?>
</head>
<header>
	<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" rel="stylesheet">
	<script src="/local/lab/jquery-3.6.0.min.js"></script>
	<link rel="stylesheet" href="/local/lab/form.css">
<div>

	<div id='content'>
		
			<div id='topline' class='dis-flex'>
				<div id='timer'><?=date( 'd.m.Y H:i' )?></div>
				<div id='auth'>	
					<div id='login'>
						<a class='user'><? echo $USER->GetFullName();?></a>
					</div>
				</div>
			</div>
			
			<div id='header' class='dis-flex'>
				<div id='logo'>Laboratory</div>
				<div id='search'></div>
			</div>
	<div id="navigation" class="navig">
		<div id="menu-container">
			<a href="/local/lab/" class="ga-nav main active">Главная</a>
			<a href="/local/lab/my_application" class="ga-nav application">Мои заявки</a>
			<a href="/local/lab/tasks" class="ga-nav tasks">В работе</a>
			<a href="/local/lab/" class="ga-nav reports">Отчёты</a>
			<a href="/local/lab/" class="ga-nav archive">Архив</a>
			<a href="/local/lab/contacts/" class="ga-nav contacts">Контакты</a>
		</div>
	</div>
</div>
</header>
