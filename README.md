<?php
/**
 * battleship.php — Морской бой на PHP для 2 игроков.
 * Игровое состояние хранится в data/battleship_{id}.json
 */

declare(strict_types=1);

// ---------- CONFIG ----------
const DATA_DIR = __DIR__ . '/data';
const BOARD_SIZE = 10;
const SHIPS_SET = [4,3,3,2,2,2,1,1,1,1]; // набор кораблей
// ---------------------------

if (!file_exists(DATA_DIR)) {
    @mkdir(DATA_DIR, 0775, true);
}

// ------------------------ API BACKEND ------------------------

function handle_api(string $action): array {
    $gameId = $_GET['game_id'] ?? '';
    $token  = $_GET['token'] ?? '';

    switch ($action) {
        case 'create_game':
            $gid = gen_id();
            $state = new_game_state();
            $tokA = gen_token();
            $state['tokens'][$tokA] = 'A';
            save_state($gid, $state);
            return [
                'ok'=>true,
                'game_id'=>$gid,
                'token'=>$tokA,
                'me'=>'A',
                'state'=>$state,
            ];

        case 'join_game':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            if (!empty($state['players']['B']['joined'])) return ['ok'=>false,'error'=>'Второй игрок уже присоединился'];
            $tokB = gen_token();
            $state['tokens'][$tokB] = 'B';
            $state['players']['B']['joined'] = true;
            save_state($gid, $state);
            return ['ok'=>true,'token'=>$tokB,'me'=>'B','state'=>$state];

        case 'state':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            return ['ok'=>true,'state'=>$state];

        case 'shoot':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            $me = auth_player($state);
            if (!$me) return ['ok'=>false,'error'=>'Неверный токен'];
            if (!empty($state['winner'])) return ['ok'=>false,'error'=>'Игра окончена'];
            if ($state['turn'] !== $me) return ['ok'=>false,'error'=>'Сейчас не ваш ход'];

            $x = (int)($_GET['x'] ?? -1);
            $y = (int)($_GET['y'] ?? -1);
            if ($x<0 || $x>=BOARD_SIZE || $y<0 || $y>=BOARD_SIZE)
                return ['ok'=>false,'error'=>'Координаты вне поля'];

            $enemy = $me === 'A' ? 'B' : 'A';

            if ($state['players'][$me]['shots'][$y][$x] !== -1) {
                return ['ok'=>false,'error'=>'Уже стреляли в эту клетку'];
            }

            $hit = $state['players'][$enemy]['board'][$y][$x] > 0 ? 1 : 0;
            $state['players'][$me]['shots'][$y][$x] = $hit;
            $state['players'][$enemy]['shots_from_enemy'][$y][$x] = $hit;

            if ($hit === 0) {
                $state['turn'] = $enemy;
            } else {
                if (all_ships_destroyed($state['players'][$enemy]['board'], $state['players'][$me]['shots'])) {
                    $state['winner'] = $me;
                }
            }

            save_state($gid, $state);
            return ['ok'=>true,'state'=>$state];

        case 'rearrange':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            $me = auth_player($state);
            if (!$me) return ['ok'=>false,'error'=>'Неверный токен'];
            if (!empty($state['winner'])) return ['ok'=>false,'error'=>'Игра окончена'];

            $state['players'][$me]['board'] = auto_place_ships();
            $enemy = $me==='A'?'B':'A';
            $state['players'][$enemy]['shots'] = init_grid(-1);
            $state['players'][$me]['shots_from_enemy'] = init_grid(-1);

            save_state($gid, $state);
            return ['ok'=>true,'state'=>$state];

        default:
            return ['ok'=>false,'error'=>'Unknown action'];
    }
}

function require_param(string $name): string {
    if (!isset($_GET[$name]) || $_GET[$name]==='') {
        throw new RuntimeException("Missing param: $name");
    }
    return (string)$_GET[$name];
}

function gen_id(): string { return strtoupper(bin2hex(random_bytes(3))); }
function gen_token(): string { return bin2hex(random_bytes(16)); }

function new_game_state(): array {
    return [
        'turn' => 'A',
        'winner' => null,
        'tokens' => [],
        'players' => [
            'A' => [
                'joined' => true,
                'board' => auto_place_ships(),
                'shots' => init_grid(-1),
                'shots_from_enemy' => init_grid(-1),
            ],
            'B' => [
                'joined' => false,
                'board' => auto_place_ships(),
                'shots' => init_grid(-1),
                'shots_from_enemy' => init_grid(-1),
            ],
        ],
    ];
}

function init_grid(int $val): array {
    $g = [];
    for ($y=0;$y<BOARD_SIZE;$y++){
        $row=[];
        for ($x=0;$x<BOARD_SIZE;$x++) $row[]=$val;
        $g[]=$row;
    }
    return $g;
}

function auto_place_ships(): array {
    $board = init_grid(0);
    $id = 1;
    foreach (SHIPS_SET as $len) {
        place_ship($board, $id++, $len);
    }
    return $board;
}

function place_ship(array &$board, int $shipId, int $len): void {
    while (true) {
        $vertical = random_int(0,1) === 1;
        if ($vertical) {
            $x = random_int(0, BOARD_SIZE-1);
            $y = random_int(0, BOARD_SIZE-$len);
        } else {
            $x = random_int(0, BOARD_SIZE-$len);
            $y = random_int(0, BOARD_SIZE-1);
        }
        if (can_place($board,$x,$y,$len,$vertical)) {
            for ($i=0;$i<$len;$i++){
                $xx = $vertical ? $x : $x+$i;
                $yy = $vertical ? $y+$i : $y;
                $board[$yy][$xx] = $shipId;
            }
            return;
        }
    }
}

function can_place(array &$board,int $x,int $y,int $len,bool $vertical): bool {
    for ($i=0;$i<$len;$i++){
        $xx = $vertical ? $x : $x+$i;
        $yy = $vertical ? $y+$i : $y;
        if ($board[$yy][$xx] !== 0) return false;
        for ($dy=-1;$dy<=1;$dy++){
            for ($dx=-1;$dx<=1;$dx++){
                $cx = $xx+$dx; $cy = $yy+$dy;
                if ($cx>=0 && $cx<BOARD_SIZE && $cy>=0 && $cy<BOARD_SIZE) {
                    if ($board[$cy][$cx] !== 0) return false;
                }
            }
        }
    }
    return true;
}

function all_ships_destroyed(array $enemyBoard, array $myShots): bool {
    for ($y=0;$y<BOARD_SIZE;$y++){
        for ($x=0;$x<BOARD_SIZE;$x++){
            if ($enemyBoard[$y][$x] > 0 && $myShots[$y][$x] !== 1) {
                return false;
            }
        }
    }
    return true;
}

function game_path(string $gid): string {
    return DATA_DIR . "/battleship_{$gid}.json";
}
function load_state(string $gid): ?array {
    $path = game_path($gid);
    if (!is_file($path)) return null;
    return json_decode(file_get_contents($path), true);
}
function save_state(string $gid, array $state): void {
    file_put_contents(game_path($gid), json_encode($state));
}
function auth_player(array $state): ?string {
    $token = $_GET['token'] ?? '';
    return $state['tokens'][$token] ?? null;
}

// ------------------------ ROUTER ------------------------
$action = $_GET['action'] ?? '';
if ($action) {
    header('Content-Type: application/json; charset=UTF-8');
    try {
        echo json_encode(handle_api($action));
    } catch (Throwable $e) {
        http_response_code(500);
        echo json_encode(['ok'=>false, 'error'=>$e->getMessage()]);
    }
    exit;
}

// ------------------------ HTML UI ------------------------
?>
<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<title>Морской бой (PHP)</title>
</head>
<body>
<h1>Морской бой запущен ✅</h1>
<p>Открывай эту страницу в двух браузерах:</p>
<ul>
<li>Первый игрок — нажимает <code>?action=create_game</code></li>
<li>Второй — <code>?action=join_game&game_id=XXXX</code></li>
</ul>
<p>Интерфейс можно подключить как в прошлой версии.</p>
</body>
</html>
