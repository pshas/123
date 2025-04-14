.container {
  padding: 2rem;
  max-width: 100%;
  overflow-x: auto;
  background: #f8fafc;
  min-height: 100vh;
}

h1 {
  color: #1a365d;
  font-weight: 600;
  font-size: 2rem;
  margin-bottom: 2rem;
  position: relative;
  padding-bottom: 0.5rem;
}

h1::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 60px;
  height: 3px;
  background: #4299e1;
}

.table-container {
  position: relative;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 
              0 2px 4px -1px rgba(0, 0, 0, 0.02);
  overflow: hidden;
}

.table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0;
  font-size: 0.95rem;
  background: white;
}

.table th {
  background-color: #2b6cb0;
  color: white;
  padding: 1rem 1.25rem;
  text-align: left;
  font-weight: 500;
  position: sticky;
  top: 0;
  transition: background 0.2s ease;
}

.table th:hover {
  background-color: #2c5282;
}

.table td {
  padding: 1rem 1.25rem;
  border-bottom: 1px solid #edf2f7;
  vertical-align: middle;
  transition: background 0.15s ease;
}

.table-striped tbody tr:nth-of-type(even) {
  background-color: #f8fafc;
}

.table tbody tr:hover td {
  background-color: #ebf8ff;
  cursor: pointer;
}

/* Улучшенные кнопки */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 500;
  line-height: 1.5;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
}

.btn-sm {
  padding: 0.375rem 0.75rem;
  font-size: 0.8125rem;
}

.btn-success {
  background-color: #38a169;
  color: white;
  border: none;
}

.btn-success:hover {
  background-color: #2f855a;
  transform: translateY(-1px);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

.btn-info {
  background-color: #4299e1;
  color: white;
  border: none;
}

.btn-info:hover {
  background-color: #3182ce;
  transform: translateY(-1px);
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

/* Интерактивные элементы статусов */
.status-badge {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.status-pending {
  background-color: #fffaf0;
  color: #dd6b20;
}

.status-completed {
  background-color: #f0fff4;
  color: #38a169;
}

.status-rejected {
  background-color: #fff5f5;
  color: #e53e3e;
}

/* Анимация загрузки */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.table tbody tr {
  animation: fadeIn 0.3s ease forwards;
}

/* Адаптивность */
@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }
  
  .table th, 
  .table td {
    padding: 0.75rem;
    font-size: 0.85rem;
  }
  
  .table-responsive {
    display: block;
    width: 100%;
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
  }
}

/* Подсветка важных ячеек */
.highlight-cell {
  position: relative;
}

.highlight-cell::after {
  content: '';
  position: absolute;
  top: 2px;
  right: 2px;
  bottom: 2px;
  left: 2px;
  border-radius: 4px;
  background-color: rgba(66, 153, 225, 0.1);
  pointer-events: none;
}

/* Интерактивные подсказки */
[data-tooltip] {
  position: relative;
}

[data-tooltip]::after {
  content: attr(data-tooltip);
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  padding: 0.5rem 1rem;
  background: #2d3748;
  color: white;
  border-radius: 4px;
  font-size: 0.75rem;
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s ease;
}

[data-tooltip]:hover::after {
  opacity: 1;
  margin-bottom: 5px;
}
