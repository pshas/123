/* Шапка: логотип и меню в один ряд, по центру по высоте */
.site-header .wrapper {
  display: flex;
  align-items: center;        /* вертикальное выравнивание по центру */
  justify-content: space-between; /* логотип слева, меню справа */
  height: 70px;               /* чтобы совпало с высотой меню */
}

/* Навигация: убираем лишние отступы, подгоняем под шапку */
#navigation {
  height: 100%;               /* занимает всю высоту .wrapper */
  display: flex;
  align-items: center;        /* вертикально по центру */
  margin: 0;                  /* убираем внешние отступы */
  padding: 0;                 /* убираем внутренние отступы */
}

/* Пунты меню */
#menu-container {
  display: flex;
  height: 100%;
}

/* Если нужно подсунуть меню ближе к логотипу */
#navigation {
  margin-left: 20px; /* подберите нужное значение */
}
