<?php
use Bitrix\Main\Mail\Mail;

require($_SERVER['DOCUMENT_ROOT'].'/bitrix/modules/main/include/prolog_before.php');

// Тема письма
$subject = "Тестовое письмо из Битрикс24";

// Кому отправляем
$to = "user@example.com";

// Тело письма (можно HTML)
$message = "
    <h2>Привет!</h2>
    <p>Это тестовое письмо, отправленное из Битрикс24 через Mail::send().</p>
";

// Отправка письма
$result = Mail::send([
    'TO' => $to,
    'SUBJECT' => $subject,
    'BODY' => $message,
    'HEADER' => [
        'From' => 'no-reply@your-domain.ru'
    ],
    'CHARSET' => 'UTF-8',
    'CONTENT_TYPE' => 'html', // или 'text'
    'MESSAGE_ID' => uniqid('', true),
    'SITE_ID' => SITE_ID,
]);

if ($result) {
    echo "Письмо отправлено!";
} else {
    echo "Ошибка при отправке письма.";
}
