/* высота всего меню */
#navigation {
  height: 70px;
}

/* контейнер пунктов — делаем флекс, чтобы пункты шли в ряд */
#navigation #menu-container {
  display: flex;
}

/* стили для каждого пункта (аналог .item) */
#navigation .ga-nav {
  width: 192px;
  color: #fff;
  background-color: #78ccfd;
  /* скорректируйте путь к изображению-разделителю */
  background-image: url('/bitrix/templates/lab/img/menu-sepa.jpg');
  background-repeat: no-repeat;
  background-position: left center;
  text-transform: uppercase;
  text-align: center;
  line-height: 70px;
  font-size: 14px;
  font-weight: 400;
  cursor: pointer;
  display: block;
  text-decoration: none;
}

/* убираем первый разделитель */
#navigation .ga-nav:first-child {
  background-image: none;
}

/* активный и hovered состояние */
#navigation .ga-nav.active,
#navigation .ga-nav:hover {
  background-color: #248fca;
}
