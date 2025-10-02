Тогда вот улучшенный вариант 👇
Он не только вытащит параметры подключения, но и сделает тестовый запрос `SELECT version();`, чтобы проверить, что соединение реально работает:

```php
<?php
require($_SERVER["DOCUMENT_ROOT"]."/bitrix/modules/main/include/prolog_before.php");

use Bitrix\Main\Application;

$connection = Application::getConnection();
$config = $connection->getConfiguration();

echo "<h3>🔑 Данные подключения к БД:</h3>";
echo "Тип: " . htmlspecialchars($connection->getType()) . "<br>";
echo "Хост: " . htmlspecialchars($config['host']) . "<br>";
echo "База: " . htmlspecialchars($config['database']) . "<br>";
echo "Логин: " . htmlspecialchars($config['login']) . "<br>";
echo "Пароль: " . htmlspecialchars($config['password']) . "<br><br>";

try {
    $result = $connection->query("SELECT version();");
    $row = $result->fetch();
    echo "<h3>✅ Соединение успешно!</h3>";
    echo "Версия сервера БД: " . htmlspecialchars($row['version']) . "<br>";
} catch (\Exception $e) {
    echo "<h3>❌ Ошибка подключения:</h3>";
    echo htmlspecialchars($e->getMessage());
}
```

---

### 🔹 Как использовать

1. Сохрани файл, например `check_db.php`, в корень проекта.
2. Открой в браузере: `https://твой-домен/check_db.php`.
3. Ты увидишь хост, базу, логин, пароль + версию PostgreSQL Pro (или MySQL, если у тебя не Postgres).
4. После проверки удали файл — чтобы никто левый не смог его открыть.

---

Хочешь, я сделаю ещё маленький вариант этого скрипта, который просто возвращает JSON (удобно для быстрого дебага через API, без HTML-вывода)?
