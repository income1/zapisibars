<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BARS - Записи</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f4f8;
            color: #333;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #007BFF;
            color: white;
            padding: 15px;
            text-align: center;
        }
        .add-icon {
            display: inline-block;
            font-size: 24px;
            color: #007BFF;
            cursor: pointer;
            margin: 10px 0;
        }
        .container {
            padding: 20px;
        }
        .appointment-list {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        .appointment-list th, .appointment-list td {
            padding: 10px;
            border-bottom: 1px solid #ddd;
        }
        .appointment-list th {
            background-color: #007BFF;
            color: white;
        }
        .soon-appointment {
            background-color: #ffcccc;
        }
        .add-appointment {
            background-color: #007BFF;
            color: white;
            padding: 10px;
            border: none;
            cursor: pointer;
        }
        .form-input {
            margin-bottom: 10px;
            padding: 10px;
            width: 100%;
            max-width: 300px;
        }
        .status-checkbox {
            cursor: pointer;
        }
    </style>
</head>
<body>

<header>
    <h1>BARS - Записи</h1>
</header>

<div class="container">
    <h2>Записи на <span id="selectedDateDisplay">сегодня</span></h2>
    <input type="date" id="selectedDate" class="form-input" value="" onchange="filterAppointmentsByDate()">
    <span class="add-icon" onclick="toggleForm()">&#43;</span> <!-- Кнопка добавления записи под заголовком -->
    <table class="appointment-list">
        <thead>
            <tr>
                <th>Клиент</th>
                <th>Дата</th>
                <th>Время</th>
                <th>Барбер</th>
                <th>Статус</th>
                <th>Действие</th>
            </tr>
        </thead>
        <tbody id="appointmentData">
            <tr>
                <td>Иван Иванов</td>
                <td>2024-10-22</td>
                <td>10:00</td>
                <td>Лаура</td>
                <td><input type="checkbox" class="status-checkbox"></td>
                <td><button onclick="removeAppointment(this)">Удалить</button></td>
            </tr>
        </tbody>
    </table>

    <div id="appointmentFormContainer" style="display: none;">
        <h2>Добавить новую запись</h2>
        <form id="appointmentForm">
            <input type="text" id="clientName" class="form-input" placeholder="Имя клиента" required>
            <input type="date" id="appointmentDate" class="form-input" required>
            <input type="time" id="appointmentTime" class="form-input" required>
            <select id="barberName" class="form-input" required>
                <option value="">Выберите барбера</option>
                <option value="Лаура">Лаура</option>
                <option value="Алмас">Алмас</option>
            </select>
            <button type="submit" class="add-appointment">Добавить запись</button>
        </form>
    </div>
</div>

<script>
    document.getElementById('appointmentForm').addEventListener('submit', function(e) {
        e.preventDefault(); // Останавливаем перезагрузку страницы

        // Получаем данные из формы
        const clientName = document.getElementById('clientName').value;
        const appointmentDate = document.getElementById('appointmentDate').value;
        const appointmentTime = document.getElementById('appointmentTime').value;
        const barberName = document.getElementById('barberName').value;

        // Создаем новую строку для таблицы
        const newRow = document.createElement('tr');
        newRow.innerHTML = `
            <td>${clientName}</td>
            <td>${appointmentDate}</td>
            <td>${appointmentTime}</td>
            <td>${barberName}</td>
            <td><input type="checkbox" class="status-checkbox"></td>
            <td><button onclick="removeAppointment(this)">Удалить</button></td>
        `;

        // Добавляем строку в таблицу
        document.getElementById('appointmentData').appendChild(newRow);

        // Очищаем форму
        document.getElementById('clientName').value = '';
        document.getElementById('appointmentDate').value = '';
        document.getElementById('appointmentTime').value = '';
        document.getElementById('barberName').value = '';

        // Сортируем записи по дате и времени
        sortAppointments();

        // Проверка времени записи для выделения
        checkSoonAppointments();

        // Скрываем форму после добавления
        toggleForm();
    });

    // Функция для удаления записи
    function removeAppointment(button) {
        button.parentElement.parentElement.remove();
    }

    // Функция для переключения видимости формы
    function toggleForm() {
        const formContainer = document.getElementById('appointmentFormContainer');
        formContainer.style.display = formContainer.style.display === 'none' ? 'block' : 'none';
    }

    // Функция для отметки красным тех, у кого запись через 30 минут
    function checkSoonAppointments() {
        const rows = document.querySelectorAll('#appointmentData tr');
        const now = new Date();

        rows.forEach(row => {
            const dateCell = row.querySelector('td:nth-child(2)').innerText;
            const timeCell = row.querySelector('td:nth-child(3)').innerText;

            const [hours, minutes] = timeCell.split(':');
            const appointmentDate = new Date(dateCell + 'T' + timeCell);

            // Если запись через 30 минут или меньше
            if ((appointmentDate - now) <= 30 * 60 * 1000 && (appointmentDate - now) > 0) {
                row.classList.add('soon-appointment'); // Добавляем класс для подсветки
            } else {
                row.classList.remove('soon-appointment'); // Убираем класс, если время не соответствует
            }
        });
    }

    // Функция для сортировки записей по дате и времени
    function sortAppointments() {
        const table = document.getElementById('appointmentData');
        const rows = Array.from(table.querySelectorAll('tr'));

        rows.sort((a, b) => {
            const dateA = new Date(a.querySelector('td:nth-child(2)').innerText + 'T' + a.querySelector('td:nth-child(3)').innerText);
            const dateB = new Date(b.querySelector('td:nth-child(2)').innerText + 'T' + b.querySelector('td:nth-child(3)').innerText);

            return dateA - dateB;
        });

        // Очищаем таблицу и добавляем отсортированные строки
        table.innerHTML = '';
        rows.forEach(row => table.appendChild(row));
    }

    // Фильтрация записей по выбранной дате
    function filterAppointmentsByDate() {
        const selectedDate = document.getElementById('selectedDate').value;
        const rows = document.querySelectorAll('#appointmentData tr');
        document.getElementById('selectedDateDisplay').innerText = selectedDate ? selectedDate : 'сегодня';

        rows.forEach(row => {
            const rowDate = row.querySelector('td:nth-child(2)').innerText;
            if (rowDate === selectedDate || selectedDate === '') {
                row.style.display = '';
            } else {
                row.style.display = 'none';
            }
        });
    }

    // Таймер для проверки записей каждые 30 секунд
    setInterval(function() {
        checkSoonAppointments();
        sortAppointments();
    }, 30000); // Каждые 30 секунд

    // Проверка при загрузке страницы
    window.onload = function() {
        document.getElementById('selectedDate').valueAsDate = new Date();
        checkSoonAppointments();
        sortAppointments(); // Сортируем записи при загрузке страницы
        filterAppointmentsByDate();
    };
</script>

</body>
</html>
