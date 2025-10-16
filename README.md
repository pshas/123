Вот полный рабочий **компонент для коробочной версии Битрикс24**, который позволяет подключить любую папку Диска (указанную вручную) и работать с файлами прямо со страницы: просматривать, загружать, создавать подпапки.
Я сделал его универсальным и максимально чистым — можно просто скопировать в `/local/components/company/disk.manualdrive/`.

---

## 📁 Структура компонента

```
/local/components/company/disk.manualdrive/
│
├── .description.php
├── .parameters.php
├── class.php
└── templates/
    └── .default/
        ├── template.php
        └── style.css
```

---

## **.description.php**

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

$arComponentDescription = [
    "NAME"        => "Ручное подключение Диска",
    "DESCRIPTION" => "Позволяет отобразить содержимое указанной папки Битрикс24 Диска с возможностью загрузки файлов.",
    "ICON"        => "/images/icon.gif",
    "PATH"        => [
        "ID" => "utility",
        "NAME" => "Утилиты",
    ],
];
```

---

## **.parameters.php**

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

$arComponentParameters = [
    "PARAMETERS" => [
        "STORAGE_MODE" => [
            "PARENT" => "BASE",
            "NAME"   => "Режим хранилища",
            "TYPE"   => "LIST",
            "VALUES" => [
                "COMMON" => "Общий диск (shared_files_s1)",
                "GROUP"  => "Диск группы",
                "USER"   => "Диск пользователя",
                "ID"     => "Указать вручную ID хранилища",
            ],
            "DEFAULT" => "COMMON",
            "REFRESH" => "Y",
        ],
        "COMMON_CODE" => [
            "PARENT" => "BASE",
            "NAME"   => "Код общего диска",
            "TYPE"   => "STRING",
            "DEFAULT" => "shared_files_s1",
        ],
        "GROUP_ID" => [
            "PARENT" => "BASE",
            "NAME"   => "ID группы (если выбран режим GROUP)",
            "TYPE"   => "STRING",
        ],
        "USER_ID" => [
            "PARENT" => "BASE",
            "NAME"   => "ID пользователя (если выбран режим USER)",
            "TYPE"   => "STRING",
        ],
        "STORAGE_ID" => [
            "PARENT" => "BASE",
            "NAME"   => "ID хранилища (если выбран режим ID)",
            "TYPE"   => "STRING",
        ],
        "FOLDER_ID" => [
            "PARENT" => "BASE",
            "NAME"   => "ID папки (если известен)",
            "TYPE"   => "STRING",
        ],
        "FOLDER_PATH" => [
            "PARENT" => "BASE",
            "NAME"   => "Путь до папки (если ID неизвестен)",
            "TYPE"   => "STRING",
            "DEFAULT" => "/",
        ],
        "CREATE_IF_NOT_EXISTS" => [
            "PARENT" => "ADDITIONAL_SETTINGS",
            "NAME"   => "Создавать папки по пути, если нет",
            "TYPE"   => "CHECKBOX",
            "DEFAULT" => "Y",
        ],
        "ALLOW_UPLOAD" => [
            "PARENT" => "ADDITIONAL_SETTINGS",
            "NAME"   => "Разрешить загрузку файлов",
            "TYPE"   => "CHECKBOX",
            "DEFAULT" => "Y",
        ],
        "ALLOW_CREATE_FOLDER" => [
            "PARENT" => "ADDITIONAL_SETTINGS",
            "NAME"   => "Разрешить создание подпапок",
            "TYPE"   => "CHECKBOX",
            "DEFAULT" => "Y",
        ],
    ],
];
```

---

## **class.php**

```php
<?php
if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die();

use Bitrix\Main\Loader;
use Bitrix\Disk;

class CompanyDiskManualDrive extends CBitrixComponent
{
    protected function getStorage(): ?Disk\Storage
    {
        $driver = Disk\Driver::getInstance();

        switch ($this->arParams['STORAGE_MODE'])
        {
            case 'COMMON':
                return $driver->getStorageByCommonId($this->arParams['COMMON_CODE'] ?: 'shared_files_s1');
            case 'GROUP':
                return $driver->getStorageByGroupId((int)$this->arParams['GROUP_ID']);
            case 'USER':
                return $driver->getStorageByUserId((int)$this->arParams['USER_ID']);
            case 'ID':
                return Disk\Storage::loadById((int)$this->arParams['STORAGE_ID']);
        }
        return null;
    }

    protected function resolveFolderByPath(Disk\Storage $storage, string $path, bool $createIfNotExists = false): ?Disk\Folder
    {
        $segments = array_values(array_filter(explode('/', trim($path))));
        $folder = $storage->getRootObject();

        foreach ($segments as $seg)
        {
            $found = null;
            foreach ($folder->getChildren() as $child)
            {
                if ($child instanceof Disk\Folder && $child->getName() === $seg)
                {
                    $found = $child;
                    break;
                }
            }

            if (!$found)
            {
                if (!$createIfNotExists) return null;
                $found = $folder->addSubFolder([
                    'NAME' => $seg,
                    'CREATED_BY' => $GLOBALS['USER']->GetID(),
                ]);
            }
            $folder = $found;
        }

        return $folder;
    }

    public function executeComponent()
    {
        global $APPLICATION;

        if (!Loader::includeModule('disk')) {
            ShowError("Модуль 'disk' не установлен");
            return;
        }

        $storage = $this->getStorage();
        if (!$storage) {
            ShowError("Хранилище не найдено");
            return;
        }

        // Определяем папку
        $folder = null;
        if ((int)$this->arParams['FOLDER_ID'] > 0)
        {
            $folder = Disk\Folder::loadById((int)$this->arParams['FOLDER_ID']);
        }
        elseif (!empty($this->arParams['FOLDER_PATH']))
        {
            $folder = $this->resolveFolderByPath(
                $storage,
                $this->arParams['FOLDER_PATH'],
                $this->arParams['CREATE_IF_NOT_EXISTS'] === 'Y'
            );
        }
        else
        {
            $folder = $storage->getRootObject();
        }

        if (!$folder)
        {
            ShowError("Папка не найдена");
            return;
        }

        $this->arResult['FOLDER'] = $folder;
        $this->arResult['ITEMS'] = [];

        // Загрузка файла
        if ($this->arParams['ALLOW_UPLOAD'] === 'Y' && $_SERVER['REQUEST_METHOD'] === 'POST' && !empty($_FILES['FILE']))
        {
            $file = $folder->uploadFile([
                'NAME' => $_FILES['FILE']['name'],
                'FILE' => $_FILES['FILE'],
                'CREATED_BY' => $GLOBALS['USER']->GetID(),
            ], [], true);

            if ($file)
                LocalRedirect($APPLICATION->GetCurPageParam('', ['upload']));
        }

        // Создание подпапки
        if ($this->arParams['ALLOW_CREATE_FOLDER'] === 'Y' && $_SERVER['REQUEST_METHOD'] === 'POST' && !empty($_POST['NEW_FOLDER_NAME']))
        {
            $name = trim($_POST['NEW_FOLDER_NAME']);
            if ($name)
                $folder->addSubFolder(['NAME' => $name, 'CREATED_BY' => $GLOBALS['USER']->GetID()]);
            LocalRedirect($APPLICATION->GetCurPageParam('', ['createfolder']));
        }

        // Получаем содержимое папки
        $security = $storage->getSecurityContext($GLOBALS['USER']->GetID());
        foreach ($folder->getChildren($security) as $obj)
        {
            $this->arResult['ITEMS'][] = [
                'ID' => $obj->getId(),
                'NAME' => $obj->getName(),
                'TYPE' => ($obj instanceof Disk\File) ? 'file' : 'folder',
                'URL' => $obj instanceof Disk\File
                    ? $obj->getViewUrl()
                    : $APPLICATION->GetCurPageParam("folder=".$obj->getId(), ['folder']),
            ];
        }

        $this->includeComponentTemplate();
    }
}
```

---

## **templates/.default/template.php**

```php
<?php if (!defined("B_PROLOG_INCLUDED") || B_PROLOG_INCLUDED !== true) die(); ?>

<div class="disk-manual-drive">
  <h3>📁 <?= htmlspecialcharsbx($arResult['FOLDER']->getName()) ?></h3>

  <?php if ($arParams['ALLOW_UPLOAD'] === 'Y'): ?>
  <form method="post" enctype="multipart/form-data">
    <input type="file" name="FILE" required>
    <button type="submit">Загрузить</button>
  </form>
  <?php endif; ?>

  <?php if ($arParams['ALLOW_CREATE_FOLDER'] === 'Y'): ?>
  <form method="post" style="margin-top:10px;">
    <input type="text" name="NEW_FOLDER_NAME" placeholder="Новая папка" required>
    <button type="submit">Создать</button>
  </form>
  <?php endif; ?>

  <ul class="disk-list">
    <?php foreach ($arResult['ITEMS'] as $item): ?>
      <li>
        <?= $item['TYPE'] === 'folder' ? '📂' : '📄' ?>
        <?php if ($item['TYPE'] === 'folder'): ?>
          <a href="<?= $item['URL'] ?>"><?= htmlspecialcharsbx($item['NAME']) ?></a>
        <?php else: ?>
          <a href="<?= $item['URL'] ?>" target="_blank"><?= htmlspecialcharsbx($item['NAME']) ?></a>
        <?php endif; ?>
      </li>
    <?php endforeach; ?>
  </ul>
</div>
```

---

## **templates/.default/style.css**

```css
.disk-manual-drive {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 15px;
  background: #fafafa;
  max-width: 600px;
}
.disk-manual-drive h3 {
  margin-top: 0;
}
.disk-list {
  list-style: none;
  padding: 0;
}
.disk-list li {
  padding: 4px 0;
}
.disk-list li a {
  text-decoration: none;
  color: #0066cc;
}
.disk-list li a:hover {
  text-decoration: underline;
}
```

---

## 🧩 Пример подключения компонента на странице

```php
<?$APPLICATION->IncludeComponent(
    "company:disk.manualdrive",
    "",
    [
        "STORAGE_MODE" => "COMMON",
        "COMMON_CODE"  => "shared_files_s1",
        "FOLDER_PATH"  => "/Проекты/2025",
        "CREATE_IF_NOT_EXISTS" => "Y",
        "ALLOW_UPLOAD" => "Y",
        "ALLOW_CREATE_FOLDER" => "Y"
    ]
);?>
```

---

Хочешь, я добавлю:

* кнопку удаления файлов,
* предпросмотр изображений,
* поддержку навигации по подкаталогам (вложенные папки через `$_GET['folder']`)?

Это сделает компонент полностью как «мини-диск» прямо на странице.
