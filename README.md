Для работы **JS API Битрикс24** (включая встраивание Диска, как в предыдущем примере) необходимо выполнить несколько условий. Вот полный список требований:

---

### 🔹 **1. Подключение JS API Битрикс24**
#### Обязательно:
```html
<!-- Основная библиотека API -->
<script src="//api.bitrix24.com/api/v1/"></script>

<!-- Инициализация API -->
<script>
    BX24.init(function() {
        console.log("API готово к работе!");
    });
</script>
```
- **Где размещать**: В `<head>` или перед закрывающим `</body>`.
- **Требования**:  
  - Страница должна открываться **в рамках Битрикс24** (портал или iframe).  
  - Пользователь должен быть авторизован в Битрикс24.

---

### 🔹 **2. Авторизация и права доступа**
#### Проверьте:
1. **Токен доступа**:  
   - JS API автоматически использует токен текущего пользователя.  
   - Для внешних сайтов потребуется [OAuth 2.0](https://training.bitrix24.com/rest_help/oauth/).

2. **Права пользователя**:  
   - Для работы с Диском нужны права:  
     - `disk` — доступ к файлам.  
     - `sonet_group` — доступ к группам (если используется Диск группы).

---

### 🔹 **3. Дополнительные библиотеки (по необходимости)**
Для сложных функций (например, drag-and-drop загрузки) подключите:
```html
<!-- UI-виджеты Битрикс24 -->
<script src="//api.bitrix24.com/api/v1/"></script>
<script src="//api.bitrix24.com/api/v1/disk.js"></script>
<script src="//api.bitrix24.com/api/v1/ui.js"></script>
```

---

### 🔹 **4. Пример полной настройки**
#### HTML:
```html
<div id="disk-container"></div>
<button id="upload-btn">Загрузить файл</button>
```

#### JavaScript:
```javascript
BX24.init(function() {
    // 1. Проверка доступности модуля Диска
    if (!BX24.Disk) {
        console.error("Модуль Диска не загружен!");
        return;
    }

    // 2. Отображение Диска группы
    BX24.DiskFolderViewer.start({
        targetNode: document.getElementById('disk-container'),
        storageId: 'group_123', // ID группы
        folderId: 0,            // Корневая папка
        mode: 'grid'            // Вид: 'list' или 'grid'
    });

    // 3. Загрузка файлов по кнопке
    document.getElementById('upload-btn').addEventListener('click', function() {
        BX24.DiskFileUpload.start({
            targetNode: document.getElementById('disk-container'),
            storageId: 'group_123',
            folderId: 0
        });
    });
});
```

---

### 🔹 **5. Ошибки и решения**
#### Проблема: **"BX24 is not defined"**
- **Причина**: Скрипт API не загрузился.  
- **Решение**:  
  - Проверьте, что страница открыта **в Битрикс24** (не во внешнем сайте).  
  - Убедитесь, что нет блокировщиков скриптов (AdBlock, HTTPS/HTTP-конфликты).

#### Проблема: **"Disk module not available"**
- **Причина**: Модуль Диска не подключен.  
- **Решение**: Добавьте явное подключение:  
  ```html
  <script src="//api.bitrix24.com/api/v1/disk.js"></script>
  ```

#### Проблема: **Нет прав на доступ к файлам**
- **Решение**:  
  - Проверьте права пользователя в настройках группы/диска.  
  - Запросите дополнительные права через OAuth:  
    ```javascript
    BX24.callMethod("disk.folder.getchildren", { id: 0 }, function(res) {
        console.log(res);
    });
    ```

---

### 🔹 **6. Где найти ID хранилища (`storageId`)?**
- **Для группы**:  
  ```javascript
  // Через JS API
  BX24.callMethod("sonet_group.get", { ID: 123 }, function(res) {
      console.log("ID хранилища: group_" + res.data().ID);
  });
  ```
- **Для пользователя**:  
  ```javascript
  // ID текущего пользователя
  console.log("user_" + BX24.getUserId());
  ```

---

### 🔹 **Итог**
Для работы JS API Битрикс24 с Диском нужно:
1. Подключить `api/bitrix24.js` + `disk.js` (если требуется).  
2. Убедиться, что код выполняется **внутри Битрикс24** (не на внешнем сайте).  
3. Проверить права пользователя.  
4. Использовать корректные `storageId` и `folderId`.

Если нужно встроить Диск **на внешний сайт**, используйте [REST API](https://training.bitrix24.com/rest_help/disk/) с OAuth 2.0.
