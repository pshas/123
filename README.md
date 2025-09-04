Чтобы таблица с формой выглядела аккуратно, была по центру экрана и имела границы, достаточно добавить стили.
Например — прямо в `profile.php` или в общий `style.css` шаблона:

```html
<style>
/* Центрируем таблицу */
form {
  display: flex;
  justify-content: center;
}

/* Таблица */
table.form {
  border-collapse: collapse;
  margin: 20px auto;
  width: 90%;             /* можно задать фикс ширину, например 1000px */
  max-width: 1200px;
  background: #fff;
}

/* Ячейки */
table.form td {
  border: 1px solid #000;
  padding: 8px;
  vertical-align: middle;
}

/* Заголовочные строки */
table.form tr:first-child td,
table.form tr:nth-child(2) td {
  font-weight: 600;
  background: #f5f5f5;
  text-align: center;
}

/* Кнопка сохранения сверху */
form > button[name="save_template"] {
  display: block;
  margin: 10px auto 20px;
  padding: 8px 20px;
  background: #248fca;
  border: none;
  color: #fff;
  cursor: pointer;
  border-radius: 4px;
}

form > button[name="save_template"]:hover {
  background: #1c6da0;
}
</style>
```

👉 Что это даст:

* Таблица будет по центру (`margin:auto` + `display:flex; justify-content:center;`).
* Все ячейки будут с чёрной рамкой (`border:1px solid #000`).
* Верхние строки («Заявка на измерение», «Номер протокола») подсветятся серым фоном.
* Кнопка «Сохранить» будет выровнена по центру и аккуратно оформлена.

Хотите, я добавлю стили ещё и для кнопки **«Печать»** снизу, чтобы она выглядела так же?
