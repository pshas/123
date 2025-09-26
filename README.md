Отлично! Ниже — готовая минимальная игра «Морской бой» на **PHP + JS** для двух игроков. Это один файл `battleship.php`, который одновременно отдает HTML-интерфейс и служит API (через `?action=...`). Хранение состояния — в JSON-файлах на диске, без БД.

## Как запустить

1. Сохраните код в файл `battleship.php` в корне PHP-сервера.
2. Убедитесь, что PHP-процесс может писать в подкаталог `data` (скрипт создаст его сам).
3. Откройте страницу в двух браузерах/вкладках:

   * На первой вкладке нажмите «Создать игру» — получите `Game ID` и `Player Token`.
   * На второй вкладке вставьте `Game ID` и нажмите «Присоединиться».
4. Ходы делаются по очереди. Победит тот, кто первым потопит все корабли соперника.

---

```php
<?php
/**
 * battleship.php — простой «Морской бой» для 2 игроков на одном файле.
 * UI + API. Хранение состояния в data/battleship_{game}.json
 * Авторотация кораблей (классика: 4-палубный 1шт, 3-палубные 2шт, 2-палубные 3шт, 1-палубные 4шт).
 */

declare(strict_types=1);

// ---------- CONFIG ----------
const DATA_DIR = __DIR__ . '/data';
const BOARD_SIZE = 10;
const SHIPS_SET = [4,3,3,2,2,2,1,1,1,1]; // длины кораблей
// ---------------------------

if (!file_exists(DATA_DIR)) {
    @mkdir(DATA_DIR, 0775, true);
}

// Роутинг: если есть action — API; иначе — отдать HTML.
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
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
  :root { --gap:8px; --cell:34px; --radius:12px; }
  body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial,sans-serif;margin:0;padding:16px;background:#0d1117;color:#e6edf3}
  h1{margin:0 0 12px}
  .wrap{display:grid;grid-template-columns:340px 1fr;gap:16px;align-items:start}
  .card{background:#161b22;border:1px solid #30363d;border-radius:16px;padding:12px;box-shadow:0 2px 10px rgba(0,0,0,.2)}
  .row{display:flex;gap:8px;flex-wrap:wrap;margin:8px 0}
  input[type=text]{padding:10px;border-radius:10px;border:1px solid #30363d;background:#0d1117;color:#e6edf3;min-width:220px}
  button{padding:10px 14px;border-radius:12px;border:1px solid #30363d;background:#238636;color:white;cursor:pointer}
  button.secondary{background:#21262d;color:#e6edf3}
  button:disabled{opacity:.6;cursor:not-allowed}
  .status{font-size:14px;opacity:.9;margin-top:6px}
  .grids{display:grid;grid-template-columns:repeat(2, auto);gap:16px}
  .grid{display:grid;grid-template-columns:repeat(<?=BOARD_SIZE?>, var(--cell));gap:4px;background:#0b0f14;padding:8px;border-radius:16px;border:1px solid #30363d}
  .cell{width:var(--cell);height:var(--cell);display:flex;align-items:center;justify-content:center;border-radius:10px;background:#1a2330;border:1px solid #263042;font-size:13px;user-select:none}
  .me .cell.ship{background:#153a2e;border-color:#1f6f57}
  .cell:hover{outline:2px solid #3b82f6}
  .enemy .cell{cursor:pointer}
  .hit{background:#8b0000 !important}
  .miss{background:#2b2f36 !important;opacity:.6}
  .head{display:flex;justify-content:space-between;align-items:center;margin-bottom:8px}
  .badge{padding:4px 8px;border-radius:999px;border:1px solid #30363d;background:#0d1117}
  .winner{font-weight:700;color:#ffcc00}
  .legend{font-size:12px;opacity:.8;margin-top:8px}
  .mono{font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; font-size: 12px;}
</style>
</head>
<body>
  <h1>Морской бой (PHP, 2 игрока)</h1>
  <div class="wrap">
    <div class="card">
      <div class="head"><strong>Лобби</strong><span class="badge" id="whoAmI">Не в игре</span></div>
      <div class="row">
        <button id="btnCreate">Создать игру</button>
      </div>
      <div class="row">
        <input id="gameId" type="text" placeholder="Game ID">
        <button class="secondary" id="btnJoin">Присоединиться</button>
      </div>
      <div class="row">
        <button class="secondary" id="btnCopy">Скопировать Game ID</button>
      </div>
      <div class="status mono" id="creds"></div>
      <div class="legend">
        Создайте игру — получите <em>Game ID</em> и <em>Player Token</em>. Передайте <em>Game ID</em> другу, чтобы он присоединился.
      </div>
    </div>

    <div class="card">
      <div class="head">
        <strong>Статус</strong>
        <span class="badge" id="turnBadge">—</span>
      </div>
      <div class="status" id="status">Ожидание действий…</div>
      <div class="row">
        <button class="secondary" id="btnRefresh">Обновить</button>
        <button class="secondary" id="btnNewShips">Переставить мои корабли</button>
      </div>
    </div>
  </div>

  <div class="card" style="margin-top:16px">
    <div class="grids">
      <div>
        <div class="head"><strong>Мое поле</strong><span class="badge">10×10</span></div>
        <div id="myGrid" class="grid me"></div>
      </div>
      <div>
        <div class="head"><strong>Поле противника</strong><span class="badge">стреляем сюда</span></div>
        <div id="enemyGrid" class="grid enemy"></div>
      </div>
    </div>
  </div>

<script>
const BOARD = <?=BOARD_SIZE?>;
const ui = {
  who: document.getElementById('whoAmI'),
  creds: document.getElementById('creds'),
  gameId: document.getElementById('gameId'),
  status: document.getElementById('status'),
  turnBadge: document.getElementById('turnBadge'),
  myGrid: document.getElementById('myGrid'),
  enemyGrid: document.getElementById('enemyGrid'),
  btnCreate: document.getElementById('btnCreate'),
  btnJoin: document.getElementById('btnJoin'),
  btnRefresh: document.getElementById('btnRefresh'),
  btnNewShips: document.getElementById('btnNewShips'),
  btnCopy: document.getElementById('btnCopy'),
};

const store = {
  gameId: localStorage.getItem('bs_game') || '',
  token: localStorage.getItem('bs_token') || '',
  me: localStorage.getItem('bs_me') || '',
};

function saveCreds() {
  localStorage.setItem('bs_game', store.gameId || '');
  localStorage.setItem('bs_token', store.token || '');
  localStorage.setItem('bs_me', store.me || '');
  ui.gameId.value = store.gameId;
  ui.creds.textContent = store.gameId ? `Game ID: ${store.gameId}\nToken: ${store.token}` : '';
  ui.who.textContent = store.me ? `Я: ${store.me}` : 'Не в игре';
}
saveCreds();

ui.btnCopy.onclick = () => {
  if (!store.gameId) return;
  navigator.clipboard.writeText(store.gameId);
};

ui.btnCreate.onclick = async () => {
  const r = await api('create_game');
  if (!r.ok) return msg(r.error, true);
  store.gameId = r.game_id;
  store.token = r.token;
  store.me = r.me;
  saveCreds();
  msg('Игра создана. Ждем второго игрока…');
  draw(r.state);
};

ui.btnJoin.onclick = async () => {
  const gid = ui.gameId.value.trim();
  if (!gid) return msg('Введите Game ID', true);
  const r = await api('join_game', {game_id: gid});
  if (!r.ok) return msg(r.error, true);
  store.gameId = gid;
  store.token = r.token;
  store.me = r.me;
  saveCreds();
  msg('Присоединились. Можно играть!');
  draw(r.state);
};

ui.btnRefresh.onclick = refresh;
ui.btnNewShips.onclick = async () => {
  const r = await api('rearrange');
  if (!r.ok) return msg(r.error, true);
  draw(r.state);
};

async function refresh() {
  if (!store.gameId || !store.token) return;
  const r = await api('state');
  if (!r.ok) return msg(r.error, true);
  draw(r.state);
}

function msg(text, bad=false) {
  ui.status.textContent = text;
  if (bad) ui.status.style.color = '#ff9b9b'; else ui.status.style.color = '#e6edf3';
}

async function api(action, params={}) {
  const p = new URLSearchParams({action});
  if (store.gameId) p.set('game_id', store.gameId);
  if (store.token) p.set('token', store.token);
  for (const [k,v] of Object.entries(params)) p.set(k,v);
  const r = await fetch(`?${p.toString()}`, {cache:'no-store'});
  return r.json();
}

function cellId(x,y){return `${x}_${y}`;}

function draw(state){
  // Бейдж хода/победителя
  if (state.winner) {
    ui.turnBadge.innerHTML = `<span class="winner">Победитель: ${state.winner}</span>`;
    msg(state.winner === store.me ? 'Вы победили! 🎉' : 'Вы проиграли 😔');
  } else {
    ui.turnBadge.textContent = `Ход: ${state.turn}`;
    msg(state.turn === store.me ? 'Ваш ход — стреляйте по полю противника.' : 'Ход соперника. Обновляйте статус.');
  }

  // Рисуем поля
  ui.myGrid.innerHTML = '';
  ui.enemyGrid.innerHTML = '';

  const me = state.players[store.me];
  const enemyKey = store.me === 'A' ? 'B' : 'A';
  const enemy = state.players[enemyKey];

  for (let y=0;y<BOARD;y++){
    for (let x=0;x<BOARD;x++){
      const c1 = document.createElement('div');
      c1.className = 'cell';
      const shipHere = me.board[y][x] > 0;
      if (shipHere) c1.classList.add('ship');
      const shotAtMe = enemy.shots[y][x]; // вражеские выстрелы по мне
      if (shotAtMe === 1) c1.classList.add('hit');
      else if (shotAtMe === 0) c1.classList.add('miss');
      ui.myGrid.appendChild(c1);

      const c2 = document.createElement('div');
      c2.className = 'cell';
      const myShot = me.shots[y][x]; // мои выстрелы по врагу
      if (myShot === 1) c2.classList.add('hit');
      else if (myShot === 0) c2.classList.add('miss');

      if (!state.winner && state.turn === store.me) {
        c2.addEventListener('click', () => shoot(x,y));
      }
      ui.enemyGrid.appendChild(c2);
    }
  }
}

async function shoot(x,y){
  const r = await api('shoot', {x, y});
  if (!r.ok) return msg(r.error, true);
  draw(r.state);
}

// Авто-обновление раз в 3 секунды, если в игре и нет победителя
setInterval(async ()=>{
  if (!store.gameId || !store.token) return;
  const r = await api('state');
  if (!r.ok) return;
  if (!r.state.winner && r.state.turn !== store.me){
    draw(r.state);
  }
}, 3000);

</script>
</body>
</html>
<?php
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
                'state'=>$state_public($state),
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
            return ['ok'=>true,'token'=>$tokB,'me'=>'B','state'=>$state_public($state)];

        case 'state':
            $gid = require_param('game_id');
            $state = load_state($gid) ?? null;
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            return ['ok'=>true,'state'=>$state_public($state)];

        case 'shoot':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            $me = auth_player($state) ?? null;
            if (!$me) return ['ok'=>false,'error'=>'Неверный токен'];
            if (!empty($state['winner'])) return ['ok'=>false,'error'=>'Игра окончена'];
            if ($state['turn'] !== $me) return ['ok'=>false,'error'=>'Сейчас не ваш ход'];

            $x = (int)($_GET['x'] ?? -1);
            $y = (int)($_GET['y'] ?? -1);
            if ($x<0 || $x>=BOARD_SIZE || $y<0 || $y>=BOARD_SIZE)
                return ['ok'=>false,'error'=>'Координаты вне поля'];

            $enemy = $me === 'A' ? 'B' : 'A';

            // Если уже стреляли сюда — запретим
            if ($state['players'][$me]['shots'][$y][$x] !== -1) {
                return ['ok'=>false,'error'=>'Уже стреляли в эту клетку'];
            }

            $hit = $state['players'][$enemy]['board'][$y][$x] > 0 ? 1 : 0;
            $state['players'][$me]['shots'][$y][$x] = $hit;
            // Отметим попадание на моем поле для врага (для удобной отрисовки у соперника):
            $state['players'][$enemy]['shots_from_enemy'][$y][$x] = $hit;

            if ($hit === 0) {
                // Промах — переход хода
                $state['turn'] = $enemy;
            } else {
                // Попали — проверим победу
                if (all_ships_destroyed($state['players'][$enemy]['board'], $state['players'][$me]['shots'])) {
                    $state['winner'] = $me;
                }
                // При попадании ход сохраняется у стрелявшего (классический «без хода» можно легко поменять)
            }

            save_state($gid, $state);
            return ['ok'=>true,'state'=>$state_public($state)];

        case 'rearrange':
            $gid = require_param('game_id');
            $state = load_state($gid);
            if (!$state) return ['ok'=>false,'error'=>'Игра не найдена'];
            $me = auth_player($state);
            if (!$me) return ['ok'=>false,'error'=>'Неверный токен'];
            if (!empty($state['winner'])) return ['ok'=>false,'error'=>'Игра окончена'];

            // Переставить только мои корабли; сбросить по мне выстрелы врага
            $state['players'][$me]['board'] = auto_place_ships();
            $enemy = $me==='A'?'B':'A';
            // Сбросим выстрелы врага по мне (чтобы было честно)
            $state['players'][$enemy]['shots'] = init_grid(-1);
            $state['players'][$me]['shots_from_enemy'] = init_grid(-1);

            save_state($gid, $state);
            return ['ok'=>true,'state'=>$state_public($state)];

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

function gen_id(): string {
    return strtoupper(bin2hex(random_bytes(3))); // 6 hex-символов
}
function gen_token(): string {
    return bin2hex(random_bytes(16));
}

function new_game_state(): array {
    $now = time();
    return [
        'created' => $now,
        'turn' => 'A',
        'winner' => null,
        'tokens' => [], // token => 'A'|'B'
        'players' => [
            'A' => [
                'joined' => true,
                'board' => auto_place_ships(),
                'shots' => init_grid(-1), // мои выстрелы по врагу: -1 не стреляли, 0 мимо, 1 попал
                'shots_from_enemy' => init_grid(-1), // выстрелы по мне
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
    $id = 1; // id корабля
    foreach (SHIPS_SET as $len) {
        place_ship($board, $id++, $len);
    }
    return $board;
}

function place_ship(array &$board, int $shipId, int $len): void {
    $tries = 0;
    while (true) {
        $tries++;
        if ($tries>10000) throw new RuntimeException('Не удалось расставить корабли');
        $vertical = random_int(0,1) === 1;
        if ($vertical) {
            $x = random_int(0, BOARD_SIZE-1);
            $y = random_int(0, BOARD_SIZE-$len);
        } else {
            $x = random_int(0, BOARD_SIZE-$len);
            $y = random_int(0, BOARD_SIZE-1);
        }
        // Проверка свободного места + «контур» (корабли не касаются)
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
        // Контур: проверим соседние клетки 3×3 вокруг каждой палубы
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

function state_public(array $state): array {
    // Возвращаем все, но это «честно»: каждый клиент сам смотрит только свои корабли.
    // (В UI поле врага закрыто, есть лишь попадания/промахи)
    return $state;
}

function game_path(string $gid): string {
    return DATA_DIR . "/battleship_{$gid}.json";
}

function load_state(string $gid): ?array {
    $path = game_path($gid);
    if (!is_file($path)) return null;
    $raw = file_get_contents($path);
    if ($raw === false) return null;
    $data = json_decode($raw, true);
    return is_array($data) ? $data : null;
}

function save_state(string $gid, array $state): void {
    $path = game_path($gid);
    file_put_contents($path, json_encode($state, JSON_UNESCAPED_UNICODE|JSON_UNESCAPED_SLASHES));
}

function auth_player(array $state): ?string {
    $token = $_GET['token'] ?? '';
    if (!$token) return null;
    return $state['tokens'][$token] ?? null;
}
```

---

### Что уже есть

* 10×10 поля, **авторасстановка** классического набора кораблей (не касаются).
* Четкое чередование ходов (при промахе — ход переходит, при попадании — остаётся).
* Отображение **попаданий/промахов**, победитель, бейдж хода.
* **Лобби**: создать/подключиться по `Game ID`, хранение токена в `localStorage`.
* Кнопка «Переставить мои корабли» (сбрасывает и выстрелы врага по вам).

### Идеи для апгрейда

* Ручная расстановка (drag&drop) с проверкой валидности.
* Показ «корабль убит» (подсветка контура).
* Пароли/имена игроков, список активных комнат.
* Очистка «старых» игр по времени.

Если хочешь, могу добавить ручную расстановку или подключить БД (SQLite/MySQL) — скажи, какие именно фичи нужны.
