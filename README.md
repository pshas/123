Ошибка **"Unable to initialize Bitrix24 JS library"** возникает, когда JavaScript не может подключиться к API Битрикс24. Давайте разберём решение по шагам:

### 🔹 Основные причины и решения

1. **Скрипт выполняется вне среды Битрикс24**
   - ✅ *Решение*: Код должен работать только внутри портала Битрикс24 (в iframe или как локальное приложение)
   - ❌ Не будет работать: На внешнем сайте, в локальном HTML-файле, в консоли браузера

2. **Библиотека не загружена**
   ```html
   <!-- Правильное подключение -->
   <script src="//api.bitrix24.com/api/v1/"></script>
   ```
   - Проверьте:
     - Нет ли ошибок в консоли (F12 → Console)
     - Не блокирует ли AdBlock скрипт
     - Корректно ли работает интернет-соединение

3. **Попытка инициализации до загрузки библиотеки**
   ```javascript
   // Правильный подход:
   function waitForBX() {
     if (typeof BX24 !== 'undefined') {
       BX24.init(myCallback);
     } else {
       setTimeout(waitForBX, 100);
     }
   }
   waitForBX();
   ```

### 🔹 Рабочий пример кода
```html
<!DOCTYPE html>
<html>
<head>
  <title>BX24 Integration</title>
  <script src="//api.bitrix24.com/api/v1/"></script>
</head>
<body>
  <div id="status">Инициализация...</div>
  
  <script>
    function initApp() {
      if (typeof BX24 === 'undefined') {
        document.getElementById('status').textContent = 
          'Ошибка: Библиотека BX24 не загружена';
        return;
      }
      
      BX24.init(function() {
        if (BX24.getAuth()) {
          document.getElementById('status').textContent = 
            'Успешно подключено! Пользователь: ' + BX24.getUser().NAME;
          
          // Ваш код работы с API
          BX24.callMethod("disk.folder.getchildren", {id: 0}, function(res) {
            console.log("Содержимое диска:", res.data());
          });
          
        } else {
          document.getElementById('status').textContent = 
            'Ошибка авторизации';
        }
      });
    }
    
    // Ждём полной загрузки страницы
    window.addEventListener('load', initApp);
  </script>
</body>
</html>
```

### 🔹 Дополнительные проверки
1. **Где размещать код?**
   - В разделе "Маркетплейс" → "Локальные приложения"
   - В iframe внутри виджета Битрикс24

2. **Проверьте права:**
   - У пользователя должны быть права на использование API
   - Приложение должно быть зарегистрировано в разделе "Разработчикам"

3. **Альтернативы:**
   ```javascript
   // Если BX24 недоступен, попробуйте:
   if (typeof BX !== 'undefined') {
     BX.ready(function() {
       // Работа через BX вместо BX24
     });
   }
   ```

### 🔹 Если ничего не помогает
1. Проверьте консоль браузера (F12) на наличие ошибок
2. Убедитесь, что:
   - Вы используете актуальную версию портала
   - Нет конфликтов с другими скриптами
   - Корпоративный firewall не блокирует API

Для внешней интеграции рассмотрите использование REST API с OAuth 2.0 вместо JS-библиотеки.
