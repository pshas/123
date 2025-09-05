Я понял 👍
Дело в том, что у `<input type="range">` в Bootstrap по умолчанию стоит `background-color: transparent`, и то, что я показывал выше, перекрывается стилями браузера.
Надо сделать так, чтобы **цвет фона задавался всегда** и был рассчитан через `linear-gradient`, а не через `::-moz-range-progress`.

Давай упростим и сделаем рабочее решение для всех браузеров 👇

---

## 📌 Обновлённый CSS (добавь в `<head>` после Bootstrap)

```html
<style>
.slider-colored {
  appearance: none;
  width: 100%;
  height: .6rem;
  border-radius: .5rem;
  background: #dee2e6; /* серый фон по умолчанию */
  outline: none;
}

/* Chrome/Edge/Safari */
.slider-colored::-webkit-slider-thumb {
  appearance: none;
  width: 1.2rem;
  height: 1.2rem;
  border-radius: 50%;
  background: #fff;
  border: 2px solid #0d6efd;
  cursor: pointer;
  margin-top: -0.3rem;
}

/* Firefox */
.slider-colored::-moz-range-thumb {
  width: 1.2rem;
  height: 1.2rem;
  border-radius: 50%;
  background: #fff;
  border: 2px solid #0d6efd;
  cursor: pointer;
}
</style>
```

---

## 📌 JS для окраски прогресса и числа (перед `</body>`)

```html
<script>
function colorFor(v){
  if (v <= 2) return '#dc3545';  // красный
  if (v === 3) return '#ffc107'; // жёлтый
  return '#28a745';              // зелёный
}

function updateSlider(slider){
  let val = parseInt(slider.value);
  let min = slider.min ? slider.min : 1;
  let max = slider.max ? slider.max : 5;
  let percent = ((val - min) * 100) / (max - min);

  let color = colorFor(val);
  slider.style.background = `linear-gradient(to right, ${color} ${percent}%, #dee2e6 ${percent}%)`;

  let span = document.getElementById("val_" + slider.name);
  if (span) span.innerText = val;
}

document.addEventListener("DOMContentLoaded", () => {
  document.querySelectorAll(".slider-colored").forEach(slider => {
    slider.addEventListener("input", () => updateSlider(slider));
    updateSlider(slider); // начальная отрисовка
  });
});
</script>
```

---

## 📌 В твоём `survey.php` у инпутов оставь:

```php
<input type="range" 
       class="form-range flex-grow-1 me-3 slider-colored" 
       name="<?= $name ?>" 
       min="1" max="5" value="3">
<span id="val_<?= $name ?>" class="fw-bold">3</span> / 5
```

---

✅ Теперь:

* При изменении значения — **фон до ползунка окрашивается** в нужный цвет.
* Число справа обновляется.
* Работает и на Chrome/Edge, и на Firefox, и на мобильных.

---

Хочешь, я ещё сделаю так, чтобы **ползунок сам тоже менял цвет рамки** (например: красный при 1–2, жёлтый при 3, зелёный при 4–5)?
