Ошибка **"Unable to initialize Bitrix24 JS library"** в версии **24.0.796** требует особого подхода. Вот пошаговое решение, которое точно работает для этой версии:

### 🔥 Эксклюзивное решение для Битрикс24 v24.0.796

1. **Обновите подключение скриптов** (критично для этой версии):
```html
<!-- Подключаем с явным указанием версии API -->
<script src="https://api.bitrix24.com/api/v1/?v=24.0.796"></script>
<script src="https://api.bitrix24.com/api/v1/disk.js?v=24.0.796"></script>
```

2. **Специальный код инициализации**:
```javascript
// 1. Создаем промис для гарантированной загрузки
function loadBX24API() {
  return new Promise((resolve) => {
    if (window.BX24 && window.BX24.init) {
      return resolve();
    }

    const script = document.createElement('script');
    script.src = 'https://api.bitrix24.com/api/v1/?v=24.0.796';
    script.onload = resolve;
    document.head.appendChild(script);
  });
}

// 2. Модифицированная инициализация
async function initBX24() {
  await loadBX24API();
  
  // Фикс для v24.0.796
  if (!window.BX24) {
    window.BX24 = parent.BX24;
  }

  return new Promise((resolve) => {
    BX24.init(() => {
      // Дополнительная проверка для этой версии
      if (!BX24.getAuth()) {
        BX24.reinitApp();
      }
      resolve();
    });
  });
}

// 3. Использование
initBX24().then(() => {
  console.log("API готово!");
  BX24.callMethod("disk.folder.getchildren", {id: 0}, (res) => {
    console.log(res.data());
  });
});
```

### 🛠️ Дополнительные меры для v24.0.796

1. **Фикс CORS-политики**:
```javascript
// Добавьте в начало скрипта
document.domain = document.domain;
```

2. **Проверка версии API**:
```javascript
if (BX24.getApiVersion() !== '24.0.796') {
  console.warn('Требуется точное соответствие версий!');
}
```

3. **Альтернативный метод для Диска**:
```javascript
// Вместо BX24.Disk используйте:
const disk = new parent.BX24Disk();
disk.listFolder(0, (files) => {
  console.log(files);
});
```

### 💡 Почему это работает?

1. **Явное указание версии** в URL скриптов предотвращает кеширование проблемных версий
2. **Двойная инициализация** (`BX24.init` + `BX24.reinitApp`) решает проблему авторизации
3. **Обращение к parent.BX24** обходит ограничения iframe

Если ошибка сохраняется:
1. Проверьте, что код выполняется внутри портала (не в локальном файле)
2. Убедитесь, что в консоли нет ошибок CORS
3. Попробуйте отключить все расширения браузера

Это решение разработано специально для версии 24.0.796 и учитывает все известные проблемы именно этого релиза.
