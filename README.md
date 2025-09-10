<form method="POST">
    <?= bitrix_sessid_post() ?>
    <table class="form">
      <tbody>
        <tr>
          <td rowspan="2" colspan="2">Заявка на измерение</td>
          <td colspan="2">Дата</td>
          <td colspan="2"><? echo date("d.m.y"); ?></td>
        </tr>
        <tr>
          <td colspan="2">Номер протокола</td>
          <td colspan="2"><? echo $title['title']; ?>-<? echo date("Y"); ?>-id</td>
        </tr>
        <tr>
          <td colspan="2">Наименование</td>
          <td colspan="2">Номер детали</td>
          <td colspan="2">Дата предоставления на замер</td>
        </tr>
        <tr>
          <td colspan="2">
            <input type="text" id="field2" name="name_izd" style="width: 100%">
            <div id="suggestions2" class="suggestions"></div>
          </td>
          <td colspan="2">
            <input type="text" id="field1" name="id_i" style="width: 100%">
            <div id="suggestions1" class="suggestions"></div>
          </td>
          <td colspan="2"></td>
        </tr>
        <tr>
          <td>Дата изготовления</td>
          <td><input type="date" id="date_man" name="date_man"></td>
          <td rowspan="3">Поставщик</td>
          <td rowspan="3"><input type="text" id="COUNTRY" name="country"></td>
          <td rowspan="3">Количество</td>
          <td rowspan="3"><input type="text" id="count_detail" name="count_detail"></td>
        </tr>
        <tr>
          <td>Дата поставки</td>
          <td><input type="date" id="date_del" name="date_del"></td>
        </tr>
        <tr>
          <td>Номер партии</td><td>&nbsp;</td>
        </tr>
        <tr>
          <td colspan="2">Примечание</td>
          <td colspan="4">Заполняется сотрудником лаборатории</td>
        </tr>
        <tr>
          <td colspan="2">&nbsp;</td>
          <td colspan="4" style="padding:0">
            <div style="display:flex;margin:0">
              <div style="width:50%;display:flex;flex-direction:column">
                <label><input type="checkbox" name="material1" value="Plastic"> Plastic</label>
                <label><input type="checkbox" name="material2" value="Metal"> Metal</label>
                <label><input type="checkbox" name="material3" value="Painted"> Painted</label>
              </div>
              <div style="width:50%;display:flex;flex-direction:column">
                <label><input type="checkbox" name="processing1" value="Zeiss"> Zeiss</label>
                <label><input type="checkbox" name="processing2" value="LaserTracer"> LaserTracer</label>
              </div>
            </div>
          </td>
        </tr>
        <tr><td colspan="6">Протокол поставки</td></tr>
        <tr>
          <td colspan="2">Дата предоставление детали</td>
          <td colspan="2"><input type="date" id="date_detail" name="date_detail"></td>
          <td>Цех/отдел</td>
          <td><input type="text" id="WORK_DEPARTMENT" name="WORK_DEPARTMENT" readonly="" value="<? $arUser = CUser::GetByID($USER->GetId())->Fetch(); echo htmlspecialcharsbx($arUser["WORK_DEPARTMENT"]); ?>"></td>
        </tr>
        <tr>
          <td colspan="2">Лицо, предоставившее деталь на измерение</td>
          <td colspan="2"><input type="text" name="full_name" readonly
             value="<?= htmlspecialcharsbx($USER->GetFullName()) ?>" style="width:100%"></td>
          <td>Тел.</td>
          <td><input type="text" name="personal_phone"
             value="<?= htmlspecialcharsbx($arUser["PERSONAL_PHONE"]) ?>"></td>
        </tr>
        <tr>
          <td colspan="2">Должность</td>
          <td colspan="2"><input type="text" name="work_position" readonly = ""value="<?= htmlspecialcharsbx($arUser["WORK_POSITION"]) ?>" style="width:100%"></td>
          <td>Тел. рабочий</td>
          <td><input type="text" name="work_phone" readonly = "" value="<?= htmlspecialcharsbx($arUser["WORK_PHONE"]) ?>"></td>
        </tr>
        <tr><td colspan="6">Предоставленные детали</td></tr>
        <tr>
          <td colspan="4"><input type="text" readonly= "" style="width:100%"></td>
          <td><input type="text" readonly= "" style="width:100%"></td>
          <td><input type="text" id="VIN" name="VIN"></td>
        </tr>
        <tr><td colspan="4"></td><td></td><td></td></tr>
        <tr><td colspan="4"></td><td></td><td></td></tr>
        <tr><td colspan="6">Задание для измерения</td></tr>
        <tr><td colspan="6"><input type="text" name="task" style="width:100%"></td></tr>
        <tr><td colspan="6">Обоснование/примечание</td></tr>
        <tr><td colspan="6"><input type="text" name="note" style="width:100%"></td></tr>
        <tr><td colspan="6">Прием/сдача</td></tr>
        <tr>
          <td colspan="6" class="td-col">
            <div style="display:flex">
              <input type="text" name="accept" style="width:50%">
              <input type="text" name="handover" style="width:50%">
            </div>
          </td>
        </tr>
        <tr>
          <td>Время измерения</td><td colspan="2"></td>
          <td colspan="2">Сотрудник ОИиУК</td><td>Подпись</td>
        </tr>
      </tbody>
    </table>
