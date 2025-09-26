<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Калькулятор ежедневного вклада</title>
  <style>
    :root{
      --bg:#0f172a;      /* slate-900 */
      --card:#111827;    /* gray-900 */
      --muted:#94a3b8;   /* slate-400 */
      --text:#e5e7eb;    /* gray-200 */
      --accent:#22d3ee;  /* cyan-400 */
      --ok:#34d399;      /* emerald-400 */
      --warn:#f59e0b;    /* amber-500 */
      --err:#ef4444;     /* red-500 */
      --ring: rgba(34,211,238,.35);
    }
    *{box-sizing:border-box}
    html,body{height:100%}
    body{
      margin:0; font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,Helvetica,Arial,"Apple Color Emoji","Segoe UI Emoji";
      background: radial-gradient(1200px 600px at 80% -10%, rgba(34,211,238,.12), transparent 70%),
                  radial-gradient(900px 500px at -10% 20%, rgba(52,211,153,.10), transparent 70%),
                  var(--bg);
      color:var(--text); line-height:1.5;
    }
    .wrap{max-width:1100px; margin:32px auto; padding:0 16px}
    header{display:flex; align-items:center; justify-content:space-between; gap:16px; margin-bottom:18px}
    h1{margin:0; font-size:clamp(20px,3vw,28px); font-weight:800}
    .subtitle{color:var(--muted); font-size:.95rem}

    .grid{display:grid; grid-template-columns:1.2fr 1fr; gap:18px}
    @media (max-width:920px){ .grid{grid-template-columns:1fr} }

    .card{background:linear-gradient(180deg, rgba(255,255,255,.05), rgba(255,255,255,.02));
          border:1px solid rgba(148,163,184,.15); border-radius:16px; padding:18px; box-shadow:0 10px 30px rgba(0,0,0,.2)}
    .card h2{margin:0 0 8px; font-size:1.05rem}

    .row{display:grid; grid-template-columns:1fr 1fr; gap:12px}
    @media (max-width:560px){ .row{grid-template-columns:1fr} }

    label{display:block; font-size:.85rem; color:var(--muted); margin-bottom:6px}
    input, select{
      width:100%; padding:12px 12px; border-radius:12px; border:1px solid rgba(148,163,184,.2);
      background:#0b1220; color:var(--text); outline:none; transition:.15s ease;
    }
    input:focus, select:focus{border-color:var(--accent); box-shadow:0 0 0 4px var(--ring)}

    .hint{font-size:.8rem; color:var(--muted)}

    .actions{display:flex; flex-wrap:wrap; gap:10px; margin-top:12px}
    button{
      padding:12px 16px; border-radius:12px; border:1px solid rgba(148,163,184,.25);
      background:#0b1324; color:var(--text); cursor:pointer; font-weight:600; transition:.15s ease;
    }
    button.primary{background:linear-gradient(180deg, rgba(34,211,238,.25), rgba(34,211,238,.1)); border-color:rgba(34,211,238,.45)}
    button:hover{transform:translateY(-1px); filter:brightness(1.05)}

    .kpi{display:grid; grid-template-columns:repeat(4,1fr); gap:12px}
    @media (max-width:920px){ .kpi{grid-template-columns:repeat(2,1fr)} }
    @media (max-width:460px){ .kpi{grid-template-columns:1fr} }
    .kpi .item{background:#0b1220; border:1px solid rgba(148,163,184,.15); border-radius:14px; padding:14px}
    .kpi .label{font-size:.8rem; color:var(--muted)}
    .kpi .value{font-size:1.15rem; font-weight:800; margin-top:4px}

    table{width:100%; border-collapse:separate; border-spacing:0; margin-top:10px}
    th, td{padding:10px 12px; border-bottom:1px dashed rgba(148,163,184,.2); text-align:right}
    th:first-child, td:first-child{text-align:left}
    thead th{font-size:.8rem; color:var(--muted)}

    .note{margin-top:10px; color:var(--muted); font-size:.85rem}
    .footer{margin-top:18px; color:var(--muted); font-size:.8rem}
    .switch{display:flex; align-items:center; gap:10px; margin-top:6px}
    .err{color:var(--err)}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Калькулятор дохода с ежедневной капитализацией</h1>
        <div class="subtitle">Рассчитайте итоговую сумму, начисленные проценты и эффективную доходность при ежедневном начислении процентов и пополнениях.</div>
      </div>
    </header>

    <div class="grid">
      <section class="card" aria-labelledby="inputs">
        <h2 id="inputs">Параметры вклада</h2>
        <div class="row">
          <div>
            <label for="amount">Начальная сумма (₽)</label>
            <input id="amount" type="number" min="0" step="0.01" placeholder="Например, 100000" value="100000" />
          </div>
          <div>
            <label for="rate">Годовая ставка, % (номинальная)</label>
            <input id="rate" type="number" min="0" step="0.01" placeholder="Например, 12" value="12" />
          </div>
        </div>
        <div class="row">
          <div>
            <label for="days">Срок, дней</label>
            <input id="days" type="number" min="1" step="1" placeholder="Например, 365" value="365" />
          </div>
          <div>
            <label for="topup">Ежедневное пополнение (₽) — опционально</label>
            <input id="topup" type="number" min="0" step="0.01" placeholder="0" value="0" />
          </div>
        </div>
        <div class="row">
          <div>
            <label for="tax">Налог на проценты, % (если применяется)</label>
            <input id="tax" type="number" min="0" step="0.01" placeholder="Напр., 13" value="0" />
            <div class="hint">Укажите 0, если налог не удерживается банком.</div>
          </div>
          <div>
            <label for="dcc">День-год (day count)</label>
            <select id="dcc">
              <option value="365" selected>Actual/365 (фикс. 365)</option>
              <option value="366">Actual/366 (для високосных лет)</option>
              <option value="act">Actual/Actual (авто по сроку)</option>
            </select>
          </div>
        </div>
        <div class="switch">
          <input type="checkbox" id="showTable" />
          <label for="showTable">Показать помесячную/подробную разбивку</label>
        </div>
        <div class="actions">
          <button class="primary" id="calcBtn">Рассчитать</button>
          <button id="resetBtn" title="Сбросить значения">Сброс</button>
        </div>
        <div class="note">Формула: ежедневная капитализация с эффективной дневной ставкой <code>r_d = r_г/БД</code>, где БД — выбранная база дней в году. Пополнения добавляются в конце каждого дня <em>после</em> начисления процентов.</div>
      </section>

      <section class="card">
        <h2>Итоги</h2>
        <div class="kpi" id="kpi">
          <div class="item"><div class="label">Итог на конец срока</div><div class="value" id="outEnd">—</div></div>
          <div class="item"><div class="label">Начисленные проценты (до налога)</div><div class="value" id="outIntGross">—</div></div>
          <div class="item"><div class="label">Налог на проценты</div><div class="value" id="outTax">—</div></div>
          <div class="item"><div class="label">Чистые проценты</div><div class="value" id="outIntNet">—</div></div>
        </div>
        <div class="kpi" style="margin-top:10px">
          <div class="item"><div class="label">Всего пополнений</div><div class="value" id="outTopups">—</div></div>
          <div class="item"><div class="label">Собственные средства</div><div class="value" id="outOwn">—</div></div>
          <div class="item"><div class="label">Эффективная доходность (годовая, ≈)</div><div class="value" id="outEffAPR">—</div></div>
          <div class="item"><div class="label">Доходность за период</div><div class="value" id="outROR">—</div></div>
        </div>
      </section>
    </div>

    <section class="card" id="tableCard" style="margin-top:18px; display:none">
      <h2>Разбивка</h2>
      <div id="tableWrap"></div>
      <div class="note">Если срок &gt; 400 дней, для производительности показываем только помесячную сводку.</div>
    </section>

    <div class="footer">⚠️ Это упрощённая модель. Фактическое начисление процентов и налогообложение может отличаться у конкретного банка.</div>
  </div>

  <script>
    const rub = new Intl.NumberFormat('ru-RU', { style:'currency', currency:'RUB', maximumFractionDigits:2 });
    const pct = new Intl.NumberFormat('ru-RU', { style:'percent', maximumFractionDigits:2 });

    const $ = id => document.getElementById(id);

    function getDayBase(days, mode){
      if(mode === '365') return 365;
      if(mode === '366') return 366;
      // act/act: грубо возьмём 365 или 366 по длине периода
      return days >= 366 ? 366 : 365;
    }

    function calc(){
      const amount = Math.max(0, Number($("amount").value || 0));
      const rate = Math.max(0, Number($("rate").value || 0)) / 100; // годовая
      const days = Math.max(1, Math.floor(Number($("days").value || 1)));
      const topup = Math.max(0, Number($("topup").value || 0));
      const taxRate = Math.max(0, Number($("tax").value || 0)) / 100;
      const base = getDayBase(days, $("dcc").value);

      const rd = rate / base; // дневная ставка

      let balance = amount;
      let interestAccrued = 0;
      let interestTax = 0;
      let totalTopups = 0;

      const dailyRows = [];
      const monthlyRows = [];

      let monthIndex = 1; // агрегируем каждые 30/31? Для простоты возьмём блоками по календарным ~30.5д.
      let daysInAgg = 0, intAgg = 0, topAgg = 0, startBalAgg = balance;

      for(let d=1; d<=days; d++){
        const interest = balance * rd;
        interestAccrued += interest;
        balance += interest;
        // пополнение в конце дня
        if(topup > 0){ balance += topup; totalTopups += topup; topAgg += topup; }
        
        // детализация — копим в массив если дней немного
        if(days <= 400 && $("showTable").checked){
          dailyRows.push({day:d, interest, balance, topup: topup});
        }
        
        // агрегирование по месяцу ≈ каждые 30.5 дня
        daysInAgg++;
        intAgg += interest;
        if(daysInAgg >= 30.5 || d === days){
          monthlyRows.push({month: monthIndex++, days: Math.round(daysInAgg), start: startBalAgg, interest:intAgg, topups:topAgg, end: balance});
          startBalAgg = balance; daysInAgg = 0; intAgg = 0; topAgg = 0;
        }
      }

      // налог (упрощённо: на всю сумму процентов)
      interestTax = interestAccrued * taxRate;
      const netInterest = interestAccrued - interestTax;
      const endBalance = amount + totalTopups + netInterest;

      // метрики доходности
      const ownFunds = amount + totalTopups;
      const ror = ownFunds > 0 ? (endBalance - ownFunds) / ownFunds : 0; // доходность за период на вложенные средства
      const effApr = Math.pow(1 + rd, base) - 1; // эффективная годовая при ежедневной капитализации

      $("outEnd").textContent = rub.format(endBalance);
      $("outIntGross").textContent = rub.format(interestAccrued);
      $("outTax").textContent = rub.format(interestTax);
      $("outIntNet").textContent = rub.format(netInterest);
      $("outTopups").textContent = rub.format(totalTopups);
      $("outOwn").textContent = rub.format(ownFunds);
      $("outEffAPR").textContent = pct.format(effApr);
      $("outROR").textContent = pct.format(ror);

      // таблица
      const tableWrap = $("tableWrap");
      tableWrap.innerHTML = '';
      const show = $("showTable").checked;
      const tableCard = $("tableCard");

      if(!show){ tableCard.style.display='none'; return; }

      tableCard.style.display = 'block';

      if(days <= 400){
        // ежедневная таблица
        const table = document.createElement('table');
        table.innerHTML = `<thead><tr><th>День</th><th>Начисленные % за день</th><th>Пополнение</th><th>Баланс на конец дня</th></tr></thead>`;
        const tbody = document.createElement('tbody');
        dailyRows.forEach(r=>{
          const tr = document.createElement('tr');
          tr.innerHTML = `<td>${r.day}</td><td>${rub.format(r.interest)}</td><td>${rub.format(r.topup)}</td><td>${rub.format(r.balance)}</td>`;
          tbody.appendChild(tr);
        });
        table.appendChild(tbody);
        tableWrap.appendChild(table);
      } else {
        // помесячная сводка
        const table = document.createElement('table');
        table.innerHTML = `<thead><tr><th>Период</th><th>Дней</th><th>Баланс на начало</th><th>Начисленные %</th><th>Пополнения</th><th>Баланс на конец</th></tr></thead>`;
        const tbody = document.createElement('tbody');
        monthlyRows.forEach(m=>{
          const tr = document.createElement('tr');
          tr.innerHTML = `<td>${m.month}</td><td>${m.days}</td><td>${rub.format(m.start)}</td><td>${rub.format(m.interest)}</td><td>${rub.format(m.topups)}</td><td>${rub.format(m.end)}</td>`;
          tbody.appendChild(tr);
        });
        table.appendChild(tbody);
        tableWrap.appendChild(table);
      }
    }

    $("calcBtn").addEventListener('click', calc);
    $("showTable").addEventListener('change', calc);
    ["amount","rate","days","topup","tax","dcc"].forEach(id=> $(id).addEventListener('change', ()=>{}));

    $("resetBtn").addEventListener('click', ()=>{
      $("amount").value = 100000;
      $("rate").value = 12;
      $("days").value = 365;
      $("topup").value = 0;
      $("tax").value = 0;
      $("dcc").value = '365';
      $("showTable").checked = false;
      $("tableCard").style.display='none';
      ["outEnd","outIntGross","outTax","outIntNet","outTopups","outOwn","outEffAPR","outROR"].forEach(id=> $(id).textContent='—');
    });

    // первый расчёт
    calc();
  </script>
</body>
</html>
