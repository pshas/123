Вы правы — для работы с **Диском Битрикс24** через JavaScript лучше использовать **`BX24.callMethod()`**, а не `BX24.DiskFolderViewer` (который может быть недоступен в некоторых версиях).  

### 🔥 **Как правильно работать с Диском через `BX24.callMethod`?**  

#### **1. Основные методы API Диска**  
Для работы с файлами и папками используйте эти REST-методы:  
- **`disk.folder.getchildren`** — получить содержимое папки.  
- **`disk.folder.get`** — информация о папке.  
- **`disk.file.get`** — данные о файле.  
- **`disk.file.addversion`** — загрузить новую версию файла.  

---

### **2. Пример: Получить список файлов из папки**  
```javascript
BX24.init(function() {
    // ID хранилища (group_123, user_1, common)
    const storageId = 'group_123'; 
    // ID папки (0 — корневая папка)
    const folderId = 0; 

    BX24.callMethod(
        'disk.folder.getchildren',
        { 
            id: folderId,
            storageId: storageId,
        },
        function(result) {
            if (result.error()) {
                console.error('Ошибка:', result.error());
            } else {
                const files = result.data();
                console.log('Файлы:', files);
            }
        }
    );
});
```
**Как получить `storageId`?**  
- Для группы: `group_123` (где `123` — ID группы).  
- Для пользователя: `user_1` (где `1` — ID пользователя).  
- Общий диск: `common`.  

---

### **3. Пример: Загрузить файл на Диск**  
```javascript
const fileInput = document.getElementById('file-input');

fileInput.addEventListener('change', function(e) {
    const file = e.target.files[0];
    const reader = new FileReader();

    reader.onload = function() {
        const fileData = reader.result.split(',')[1]; // Base64 без префикса

        BX24.callMethod(
            'disk.folder.uploadfile',
            {
                id: 0, // ID папки (0 — корень)
                storageId: 'group_123',
                fileContent: fileData,
                fileName: file.name,
            },
            function(result) {
                if (result.error()) {
                    console.error('Ошибка загрузки:', result.error());
                } else {
                    console.log('Файл загружен:', result.data());
                }
            }
        );
    };

    reader.readAsDataURL(file);
});
```

---

### **4. Как получить ID группы или пользователя?**  
#### **Текущий пользователь**  
```javascript
BX24.init(function() {
    const userId = BX24.getUserId(); // ID текущего пользователя
    console.log('user_' + userId);
});
```

#### **ID группы**  
Через метод `sonet_group.get`:  
```javascript
BX24.callMethod(
    'sonet_group.get',
    { FILTER: { NAME: 'Название группы' } },
    function(result) {
        const groupId = result.data()[0].ID;
        console.log('group_' + groupId);
    }
);
```

---

### **5. Обработка ошибок**  
- **`result.error()`** — возвращает текст ошибки.  
- **Проверка кода ответа**:  
  ```javascript
  if (result.status >= 400) {
      console.error('HTTP Error:', result.status);
  }
  ```

---

### **6. Полный пример: Вывод списка файлов в HTML**  
```html
<div id="file-list"></div>

<script src="//api.bitrix24.com/api/v1/"></script>
<script>
    BX24.init(function() {
        BX24.callMethod(
            'disk.folder.getchildren',
            { id: 0, storageId: 'group_123' },
            function(result) {
                const files = result.data();
                const list = document.getElementById('file-list');

                files.forEach(file => {
                    const div = document.createElement('div');
                    div.textContent = file.NAME;
                    list.appendChild(div);
                });
            }
        );
    });
</script>
```

---

### **7. Важные нюансы**  
1. **Работает только внутри Битрикс24**  
   - `BX24.callMethod` не сработает на внешнем сайте. Для этого используйте **REST API с OAuth 2.0**.  

2. **Требуются права**  
   - У пользователя должны быть права на чтение/запись в Диске.  

3. **Лимиты запросов**  
   - Битрикс24 ограничивает частоту вызовов API (≈ 2 запроса/сек).  

---

### **Итог**  
Для работы с **Диском Битрикс24 через JS**:  
1. Используйте **`BX24.callMethod('disk.*')`**.  
2. Передавайте корректные **`storageId`** и **`folderId`**.  
3. Обрабатывайте ошибки через **`result.error()`**.  

Если нужно решение для **внешнего сайта**, примените [REST API](https://training.bitrix24.com/rest_help/).
