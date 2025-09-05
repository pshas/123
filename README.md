<script>
function updateSliderColor(slider) {
    let value = parseInt(slider.value);
    let color = "#0d6efd"; // по умолчанию синий

    if (value <= 2) color = "#dc3545"; // красный
    else if (value === 3) color = "#ffc107"; // жёлтый
    else if (value >= 4) color = "#28a745"; // зелёный

    slider.style.background = `linear-gradient(to right, ${color} ${(value-1)*25}%, #dee2e6 ${(value-1)*25}%)`;
}

// Инициализация всех слайдеров
document.querySelectorAll(".slider-colored").forEach(slider => {
    const span = document.getElementById("val_" + slider.name);

    function update() {
        span.innerText = slider.value;
        updateSliderColor(slider);
    }

    slider.addEventListener("input", update);
    update(); // первый запуск
});
</script>
