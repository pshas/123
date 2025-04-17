Чтобы визуально выделить текущую страницу в навигационном меню, можно добавить активному элементу специальный CSS-класс (например, `active`) и стилизовать его. Вот как это можно реализовать:

### 1. Добавьте класс `active` к текущей ссылке
Например, если пользователь находится на странице "Мои заявки":
```html
<a href="/local/lab/my_application" class="ga-nav application active">Мои заявки</a>
```

### 2. Добавьте CSS-стили для активного элемента
```css
.navig a {
    /* Стили для обычных ссылок */
    color: #333;
    text-decoration: none;
    padding: 5px 10px;
    transition: all 0.3s ease;
}

.navig a.active {
    /* Стили для активной ссылки */
    color: #0066cc;
    font-weight: bold;
    border-bottom: 2px solid #0066cc;
}

.navig a:hover {
    /* Стили при наведении */
    color: #0066cc;
}
```

### 3. Динамическое добавление класса `active` (если используете JavaScript)
Если у вас SPA или вы можете использовать JavaScript:
```javascript
// Получаем текущий URL
const currentUrl = window.location.pathname;

// Находим все ссылки в навигации
const navLinks = document.querySelectorAll('#menu-container a');

// Перебираем ссылки и добавляем класс active к соответствующей
navLinks.forEach(link => {
    if (link.getAttribute('href') === currentUrl) {
        link.classList.add('active');
    }
});
```

### Альтернативный вариант - использовать разные стили для разных страниц
Если вы не можете изменять HTML, можно использовать атрибут `href` для стилизации:
```css
/* Для страницы "Мои заявки" */
a[href="/local/lab/my_application"] {
    color: #0066cc;
    font-weight: bold;
    border-bottom: 2px solid #0066cc;
}
```

Выберите подход, который лучше подходит для вашего проекта. Самый надежный - это добавление класса `active` к текущей ссылке на серверной стороне при генерации страницы.
