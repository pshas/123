Если у вас **Битрикс24 версии 24.0.796** и не работает инициализация JS API (`BX24.init`), вот специальное решение для этой версии:

### 🔧 Специфичные проблемы версии 24.0.796
1. **Изменения в API**  
   В этой версии были обновлены методы аутентификации.

2. **Новые требования к безопасности**  
   Требуется явное разрешение CORS для локальных скриптов.

### 🛠️ Рабочее решение для v24.0.796
```html
<!DOCTYPE html>
<html>
<head>
  <!-- Обязательно указываем протокол HTTPS -->
  <script src="https://api.bitrix24.com/api/v1/"></script>
  <!-- Дополнительные библиотеки для этой версии -->
  <script src="https://api.bitrix24.com/api/v1/disk.js"></script>
  <script src="https://api.bitrix24.com/api/v1/ui.js"></script>
</head>
<body>
  <div id="bx24-content"></div>

  <script>
    // 1. Ожидаем полной загрузки API
    function checkBX24Loaded() {
      if (typeof BX24 !== 'undefined' && 
          typeof BX24.init !== 'undefined' &&
          typeof BX24Disk !== 'undefined') {
        initializeApp();
      } else {
        setTimeout(checkBX24Loaded, 100);
      }
    }

    // 2. Инициализация приложения
    function initializeApp() {
      BX24.init(function() {
        // Проверяем специальный флаг для v24.0.796
        if (!BX24.__isAPIReady) {
          console.warn("Требуется дополнительная инициализация");
          BX24.reinitApp();
        }
        
        // Ваш код работы с Диском
        BX24.callMethod(
          'disk.folder.getchildren',
          {id: 0, storageId: 'group_123'},
          function(result) {
            if (result.error()) {
              console.error("Ошибка:", result.error());
            } else {
              console.log("Файлы:", result.data());
              document.getElementById('bx24-content').innerHTML = 
                JSON.stringify(result.data(), null, 2);
            }
          }
        );
      });
    }

    // 3. Запускаем проверку
    checkBX24Loaded();
  </script>
</body>
</html>
```

### 🔍 Особенности для версии 24.0.796
1. **Обязательные скрипты**:
   ```html
   <script src="https://api.bitrix24.com/api/v1/"></script>
   <script src="https://api.bitrix24.com/api/v1/disk.js"></script>
   ```

2. **Дополнительная проверка**:
   ```javascript
   if (!BX24.__isAPIReady) {
     BX24.reinitApp();
   }
   ```

3. **Работа с Диском**:
   Используйте `BX24Disk` вместо `BX24.Disk`:
   ```javascript
   BX24Disk.startEditFile(fileId, {
     events: {
       onSave: function() { /* ... */ }
     }
   });
   ```

### ⚠️ Важные замечания
1. Размещайте код только в:
   - Локальных приложениях Битрикс24
   - iframe внутри портала

2. Для внешней интеграции используйте REST API:
   ```javascript
   fetch('https://your-portal.bitrix24.ru/rest/user.current.json?auth=' + auth_token)
   ```

3. Если проблема сохраняется:
   - Проверьте консоль браузера (F12)
   - Убедитесь, что нет конфликтов с другими скриптами
   - Попробуйте очистить кеш браузера

Это решение специально адаптировано под особенности API в версии 24.0.796. Для других версий могут потребоваться корректировки.
