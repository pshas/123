<style>
/* Центрируем таблицу */
form {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #000; /* весь текст формы черный */
}

/* Таблица */
table.form {
  border-collapse: collapse;
  margin: 20px auto;
  width: 90%;
  max-width: 1200px;
  background: #fff;
  color: #000; /* текст внутри таблицы черный */
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

/* Инпуты и текстовые поля */
table.form input,
table.form textarea {
  border: none;
  outline: none;
  width: 100%;
  color: #000;
  font-family: inherit;
  font-size: 14px;
  background: transparent;
}

/* Кнопка "Сохранить" */
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

/* Кнопка "Печать" */
.no-print {
  background: #248fca;
  border: none;
  color: #fff;
  cursor: pointer;
  border-radius: 4px;
  padding: 8px 20px;
}

.no-print:hover {
  background: #1c6da0;
}
</style>
