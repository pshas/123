<?php
$dsn = "pgsql:host=localhost;port=5432;dbname=sitemanager;";
$user = "bitrix";
$pass = "supersecret";

try {
    $pdo = new PDO($dsn, $user, $pass, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_NUM,
    ]);

    // Базовые версии PostgreSQL
    [$serverVersion] = $pdo->query("SHOW server_version")->fetch();
    [$serverVersionNum] = $pdo->query("SHOW server_version_num")->fetch();

    // Параметры Postgres Pro (возвращают NULL, если это не Pro)
    [$pgproVersion] = $pdo->query("SELECT current_setting('pgpro_version', true)")->fetch();
    [$pgproEdition] = $pdo->query("SELECT current_setting('pgpro_edition', true)")->fetch();
    [$pgproBuild]   = $pdo->query("SELECT current_setting('pgpro_build',   true)")->fetch();

    // Фолбэк: текстовая строка version()
    [$verString] = $pdo->query("SELECT version()")->fetch();

    $isPro = !empty($pgproVersion) || !empty($pgproEdition);

    header('Content-Type: text/plain; charset=utf-8');
    echo "server_version:      {$serverVersion}\n";
    echo "server_version_num:  {$serverVersionNum}\n";
    echo "pgpro_version:       " . ($pgproVersion ?: "NULL") . "\n";
    echo "pgpro_edition:       " . ($pgproEdition ?: "NULL") . "\n";
    echo "pgpro_build:         " . ($pgproBuild   ?: "NULL") . "\n";
    echo "version():           {$verString}\n";
    echo "Detected:            " . ($isPro ? "Postgres Pro ({$pgproEdition})" : "vanilla PostgreSQL") . "\n";

} catch (PDOException $e) {
    http_response_code(500);
    echo "DB error: " . $e->getMessage();
}
