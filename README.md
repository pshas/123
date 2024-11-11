Если запрос к `search_users.php` пустой, это означает, что значение параметра `query` из JavaScript не передается на сервер. Давайте еще раз проверим, правильно ли передается параметр, и исправим возможные ошибки.

### Проверка и исправление передачи параметра в JavaScript

1. **Убедитесь, что значение `query` передается в запросе AJAX**:
   В коде JavaScript проверьте, что переменная `query` действительно содержит текст из поля ввода и передается в запрос `BX.ajax`. 

2. **Измените обработчик `oninput` для тестирования параметра `query`**:
   Временно добавьте вывод переменной `query` в консоль сразу после её объявления:

   ```javascript
   document.getElementById('user_search').oninput = function() {
       const query = this.value.trim();
       console.log("Запрос:", query); // Выводим значение query в консоль для проверки

       if (query.length < 2) {
           document.getElementById('user_suggestions').style.display = 'none';
           return;
       }

       // Отправляем AJAX-запрос
       BX.ajax({
           url: '/local/ajax/search_users.php',
           method: 'POST',
           dataType: 'json',
           data: {
               query: query,
               sessid: BX.bitrix_sessid()
           },
           onsuccess: function(data) {
               console.log("Ответ от сервера:", data); // Выводим ответ для отладки
               let suggestions = document.getElementById('user_suggestions');
               suggestions.innerHTML = '';

               if (data && data.length > 0) {
                   data.forEach(function(user) {
                       let li = document.createElement('li');
                       li.textContent = user.NAME + ' (' + user.EMAIL + ')';
                       li.dataset.userId = user.ID;

                       li.onclick = function() {
                           document.getElementById('user_search').value = user.NAME;
                           suggestions.style.display = 'none';
                       };

                       suggestions.appendChild(li);
                   });

                   suggestions.style.display = 'block';
               } else {
                   suggestions.style.display = 'none';
               }
           },
           onfailure: function(error) {
               console.error('Ошибка при запросе к серверу', error);
           }
       });
   };
   ```

   - Откройте консоль браузера (обычно `F12`), чтобы убедиться, что значение `query` корректно выводится при вводе текста в поле.
   - Проверьте, что консоль выводит это значение и что оно передается в запрос `BX.ajax`.

3. **Проверьте код HTML, чтобы убедиться, что у поля ввода установлен правильный ID**:
   
   Код JavaScript ожидает, что поле поиска имеет `id="user_search"`. Проверьте ваш HTML, чтобы убедиться, что идентификатор совпадает:

   ```html
   <input type="text" id="user_search" placeholder="Введите имя пользователя">
   ```

4. **Проверка корректного использования `BX.ajax`**:
   Иногда `BX.ajax` может не отправлять данные, если не подключен ядро Bitrix. Убедитесь, что вызов `BX.ready()` присутствует и что код оборачивает всё внутри этой функции:

   ```javascript
   BX.ready(function() {
       document.getElementById('user_search').oninput = function() {
           const query = this.value.trim();
           console.log("Запрос:", query);

           if (query.length < 2) {
               document.getElementById('user_suggestions').style.display = 'none';
               return;
           }

           BX.ajax({
               url: '/local/ajax/search_users.php',
               method: 'POST',
               dataType: 'json',
               data: {
                   query: query,
                   sessid: BX.bitrix_sessid()
               },
               onsuccess: function(data) {
                   console.log("Ответ от сервера:", data);
                   let suggestions = document.getElementById('user_suggestions');
                   suggestions.innerHTML = '';

                   if (data && data.length > 0) {
                       data.forEach(function(user) {
                           let li = document.createElement('li');
                           li.textContent = user.NAME + ' (' + user.EMAIL + ')';
                           li.dataset.userId = user.ID;

                           li.onclick = function() {
                               document.getElementById('user_search').value = user.NAME;
                               suggestions.style.display = 'none';
                           };

                           suggestions.appendChild(li);
                       });

                       suggestions.style.display = 'block';
                   } else {
                       suggestions.style.display = 'none';
                   }
               },
               onfailure: function(error) {
                   console.error('Ошибка при запросе к серверу', error);
               }
           });
       };
   });
   ```

5. **Проверка работы через другие инструменты отладки**:
   Если `BX.ajax` всё равно не передает данные, попробуйте временно заменить его на обычный `fetch` для отладки. Это поможет выявить, в чем проблема:

   ```javascript
   document.getElementById('user_search').oninput = function() {
       const query = this.value.trim();
       console.log("Запрос:", query);

       if (query.length < 2) {
           document.getElementById('user_suggestions').style.display = 'none';
           return;
       }

       fetch('/local/ajax/search_users.php', {
           method: 'POST',
           headers: {
               'Content-Type': 'application/x-www-form-urlencoded'
           },
           body: `query=${encodeURIComponent(query)}&sessid=${BX.bitrix_sessid()}`
       })
       .then(response => response.json())
       .then(data => {
           console.log("Ответ от сервера:", data);
           let suggestions = document.getElementById('user_suggestions');
           suggestions.innerHTML = '';

           if (data && data.length > 0) {
               data.forEach(function(user) {
                   let li = document.createElement('li');
                   li.textContent = user.NAME + ' (' + user.EMAIL + ')';
                   li.dataset.userId = user.ID;

                   li.onclick = function() {
                       document.getElementById('user_search').value = user.NAME;
                       suggestions.style.display = 'none';
                   };

                   suggestions.appendChild(li);
               });

               suggestions.style.display = 'block';
           } else {
               suggestions.style.display = 'none';
           }
       })
       .catch(error => {
           console.error('Ошибка при запросе к серверу', error);
       });
   };
   ```

Использование `fetch` временно поможет понять, если проблема была в `BX.ajax`.
