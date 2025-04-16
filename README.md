<script src="//api.bitrix24.com/api/v1/"></script>
<script>
    BX24.init(function() {
        // Просмотр файлов
        BX24.DiskFolderViewer.start({
            targetNode: document.getElementById('disk-viewer'),
            storageId: 'group_123',
            folderId: 0,
            mode: 'grid',
        });

        // Загрузка файлов
        BX24.DiskFileUpload.start({
            targetNode: document.getElementById('disk-upload'),
            storageId: 'group_123',
            folderId: 0,
        });
    });
</script>

<!-- Контейнер для просмотра файлов -->
<div id="disk-viewer"></div>

<!-- Контейнер для загрузки -->
<div id="disk-upload"></div>
