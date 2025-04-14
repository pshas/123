/* Базовые настройки */
.container {
  padding: 2rem;
  background-color: #f9fafb;
  min-height: 100vh;
}

/* Заголовок */
.page-title {
  color: #111827;
  font-size: 1.8rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 1px solid #e5e7eb;
}

/* Контейнер таблицы */
.table-wrapper {
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

/* Сама таблица */
.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
}

/* Шапка таблицы */
.data-table thead {
  background-color: #f3f4f6;
}

.data-table th {
  padding: 0.875rem 1rem;
  text-align: left;
  color: #374151;
  font-weight: 500;
  border-bottom: 2px solid #e5e7eb;
}

/* Ячейки таблицы */
.data-table td {
  padding: 0.875rem 1rem;
  border-bottom: 1px solid #e5e7eb;
  vertical-align: top;
}

/* Чередование строк */
.data-table tbody tr:nth-child(even) {
  background-color: #f9fafb;
}

/* Эффект при наведении */
.data-table tbody tr:hover {
  background-color: #f0f3f9;
}

/* Кнопки */
.action-btn {
  display: inline-block;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  text-align: center;
  transition: all 0.15s ease;
}

.btn-submit {
  background-color: #3b82f6;
  color: white;
  border: 1px solid #2563eb;
}

.btn-submit:hover {
  background-color: #2563eb;
}

.btn-report {
  background-color: #10b981;
  color: white;
  border: 1px solid #059669;
}

.btn-report:hover {
  background-color: #059669;
}

/* Статусы */
.status {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.8125rem;
  font-weight: 500;
}

.status-pending {
  background-color: #fef3c7;
  color: #d97706;
}

.status-completed {
  background-color: #d1fae5;
  color: #059669;
}

.status-rejected {
  background-color: #fee2e2;
  color: #dc2626;
}

/* Адаптивность */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }
  
  .data-table th,
  .data-table td {
    padding: 0.75rem;
    font-size: 0.875rem;
  }
  
  .action-btn {
    padding: 0.375rem 0.75rem;
    font-size: 0.8125rem;
  }
}
