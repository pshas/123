function fetchSuggestions(query, fieldId) {
            if (query.length === 0) {
                document.getElementById(`suggestions${fieldId}`).innerHTML = '';
                return;
            }

            fetch('get-info.php', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
                body: `query=${encodeURIComponent(query)}&fieldId=${fieldId}` // Передача данных
            })
            .then(response => response.json())
            .then(data => {
                const suggestionsBox = document.getElementById(`suggestions${fieldId}`);
                suggestionsBox.innerHTML = '';

                data.forEach(item => {
                    const div = document.createElement('div');
					div.textContent = fieldId === 1 ? item['MODEL'] : item['NAME_IZD'];
                    div.onclick = () => {
						document.getElementById(`field${fieldId}`).value = fieldId === 1 ? item['MODEL'] : item['NAME_IZD'];
						suggestionsBox.innerHTML = '';
                    };
                    suggestionsBox.appendChild(div);
                });
            })
            .catch(error => console.error('Ошибка:', error));
        }

        // Обработчики ввода для двух полей
        document.getElementById('field1').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, 1);
        });

        document.getElementById('field2').addEventListener('input', (e) => {
            fetchSuggestions(e.target.value, 2);
        });
