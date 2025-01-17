<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Форма заявки</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
        }
        header {
            background-color: #0073e6;
            color: white;
            padding: 15px;
            text-align: center;
        }
        .container {
            width: 90%;
            margin: 20px auto;
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        table, th, td {
            border: 1px solid #ddd;
        }
        th, td {
            padding: 10px;
            text-align: left;
        }
        .buttons {
            text-align: right;
            margin-bottom: 20px;
        }
        .buttons button {
            padding: 10px 20px;
            margin-left: 10px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .buttons .save {
            background-color: #4CAF50;
            color: white;
        }
        .buttons .submit {
            background-color: #0073e6;
            color: white;
        }
        .highlight {
            background-color: #f9f9f9;
        }
    </style>
</head>
<body>
    <header>
        <h1>Форма заявки</h1>
    </header>
    <div class="container">
        <div class="buttons">
            <button class="save">Сохранить</button>
            <button class="submit">Оформить</button>
        </div>
        <form>
            <table>
                <tr>
                    <th colspan="2">Заявка на измерение</th>
                </tr>
                <tr>
                    <td>Дата</td>
                    <td><input type="date" name="date"></td>
                </tr>
                <tr>
                    <td>Номер протокола</td>
                    <td>CMM-2023-001</td>
                </tr>
                <tr>
                    <td>Название детали</td>
                    <td>Шланг</td>
                </tr>
                <tr>
                    <td>Номер детали</td>
                    <td>3506045</td>
                </tr>
                <tr>
                    <td>Дата изготовления</td>
                    <td><input type="date" name="manufacture_date"></td>
                </tr>
                <tr>
                    <td>Поставщик</td>
                    <td>
                        <select name="supplier">
                            <option>Китай</option>
                            <option>Россия</option>
                            <option>Германия</option>
                        </select>
                    </td>
                </tr>
                <tr>
                    <td>Количество</td>
                    <td><input type="number" name="quantity"></td>
                </tr>
                <tr class="highlight">
                    <td>Материал</td>
                    <td>
                        <label><input type="checkbox" name="material" value="Plastic"> Plastic</label>
                        <label><input type="checkbox" name="material" value="Metal"> Metal</label>
                        <label><input type="checkbox" name="material" value="Painted"> Painted</label>
                    </td>
                </tr>
                <tr>
                    <td>Дата предоставления</td>
                    <td><input type="date" name="submission_date"></td>
                </tr>
                <tr>
                    <td>Задание для измерения</td>
                    <td>
                        <textarea name="task" rows="3" placeholder="Введите задание..."></textarea>
                    </td>
                </tr>
                <tr>
                    <td>Дефект</td>
                    <td><input type="text" name="defect" placeholder="Описание дефекта"></td>
                </tr>
                <tr>
                    <td>Примечания</td>
                    <td><textarea name="notes" rows="3"></textarea></td>
                </tr>
                <tr>
                    <td colspan="2" style="text-align: right;">
                        <button type="submit" class="submit">Сохранить</button>
                    </td>
                </tr>
            </table>
        </form>
    </div>
</body>
</html>
