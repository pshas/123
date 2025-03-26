Вот как добавить кнопку "Печать" в правом нижнем углу формы:

1. Добавьте кнопку в HTML (перед закрывающим тегом `</form>`):
```html
<div style="position: fixed; bottom: 20px; right: 20px;">
    <button onclick="window.print()" class="btn btn-primary no-print">
        <i class="fas fa-print"></i> Печать
    </button>
</div>
```

2. Добавьте стили для печати (в `<head>` или в `<style>`):
```html
<style>
    @media print {
        body * {
            visibility: hidden;
        }
        .form, .form * {
            visibility: visible;
        }
        .form {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
        }
        .no-print {
            display: none !important;
        }
        
        /* Улучшение отображения таблицы при печати */
        table.form {
            border-collapse: collapse;
            width: 100%;
        }
        table.form td, table.form th {
            border: 1px solid #000;
            padding: 8px;
        }
    }
    
    /* Стиль для кнопки печати */
    .print-btn {
        position: fixed;
        bottom: 20px;
        right: 20px;
        z-index: 1000;
    }
</style>
```

3. Добавьте FontAwesome для иконки принтера (в `<head>`):
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.4/css/all.min.css">
```

4. Обновите класс таблицы (если нужно):
```html
<table class="table form">
```

Полный код кнопки с улучшениями:
```html
<div class="print-btn no-print">
    <button onclick="printForm()" class="btn btn-primary">
        <i class="fas fa-print"></i> Печать формы
    </button>
</div>

<script>
function printForm() {
    // Можно добавить дополнительные действия перед печатью
    window.print();
}
</script>
```

Это решение:
1. Размещает кнопку в правом нижнем углу экрана
2. Скрывает кнопку при печати
3. Обеспечивает печать только таблицы с классом `form`
4. Добавляет иконку принтера для лучшего UX
5. Сохраняет все стили таблицы при печати

Дополнительно можно добавить обработку перед печатью, например, скрытие ненужных элементов:
```javascript
function printForm() {
    // Скрыть лишние элементы
    document.querySelectorAll('.no-print').forEach(el => {
        el.style.display = 'none';
    });
    
    window.print();
    
    // Показать обратно после печати
    setTimeout(() => {
        document.querySelectorAll('.no-print').forEach(el => {
            el.style.display = '';
        });
    }, 500);
}
```
