**Глава 6. `<details>` и `<summary>` — декларативные раскрывающиеся блоки**

### Главная идея главы

Элементы `<details>` и `<summary>` позволяют создавать раскрывающиеся блоки (аккордеоны, спойлеры, секции фильтров) полностью на HTML, без JavaScript. Это один из лучших примеров Progressive Enhancement: базовая функциональность работает даже при отключённом JavaScript.

Основные возможности:
- Нативное раскрытие/сворачивание по клику
- Атрибут `open` для управления состоянием
- Доступность из коробки
- Возможность стилизации
- Можно использовать как одиночный блок или собирать аккордеон

### Базовый синтаксис

```html
<details>
  <summary>Заголовок секции</summary>
  <p>Скрытое содержимое</p>
</details>
```

С открытым состоянием по умолчанию:

```html
<details open>
  <summary>Фильтры</summary>
  ...
</details>
```

### Практическая задача главы

Реализовать в боковой панели FocusBoard блок фильтров с помощью `<details>`:

- Фильтр по статусу (Активные / Выполненные / Все)
- Фильтр по приоритету (Высокий / Средний / Низкий)
- Секция «Быстрые действия»

Пока фильтры будут только визуальными (без реальной фильтрации списка задач). Логику фильтрации добавим позже.

### Структура проекта после Главы 6

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
└── README.md
```

Структура файлов остаётся прежней.

---

### Содержание файлов на данном этапе

#### `index.html`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="FocusBoard 2026 — современная персональная панель продуктивности, построенная на возможностях современного HTML">
  <meta name="theme-color" content="#0f172a">
  <title>FocusBoard 2026 — Задачи</title>
  <link rel="stylesheet" href="styles/main.css">
</head>
<body>
  <!-- ===== Шапка приложения ===== -->
  <header>
    <h1>FocusBoard</h1>
    <p>Персональная панель продуктивности</p>
  </header>

  <!-- ===== Основная навигация ===== -->
  <nav aria-label="Основная навигация">
    <ul>
      <li><a href="#tasks" aria-current="page">Задачи</a></li>
      <li><a href="#notes">Заметки</a></li>
      <li><a href="#settings">Настройки</a></li>
    </ul>
  </nav>

  <!-- ===== Поиск ===== -->
  <search>
    <form role="search" aria-label="Поиск по задачам и заметкам">
      <label for="site-search">Поиск</label>
      <input type="search" id="site-search" name="q" placeholder="Поиск задач и заметок…">
      <button type="submit">Найти</button>
    </form>
  </search>

  <!-- ===== Основное содержимое ===== -->
  <main id="tasks">
    <header>
      <h2>Задачи</h2>
      <p>Активные задачи на сегодня</p>
      
      <p>
        <button type="button" id="create-task-btn">
          Создать задачу
        </button>
      </p>
    </header>

    <section aria-labelledby="active-tasks-heading">
      <h3 id="active-tasks-heading">Активные</h3>

      <article class="task" aria-labelledby="task-1-title" data-task-id="1" data-priority="high" data-status="active">
        <header>
          <h4 id="task-1-title">Изучить современный HTML</h4>
        </header>
        <p>Разобрать семантику, dialog, popover и Web Components.</p>
        <footer>
          <p>Приоритет: высокий</p>
          
          <button type="button" popovertarget="task-1-menu" popovertargetaction="toggle">
            Действия
          </button>

          <div id="task-1-menu" popover="auto" class="task-menu">
            <menu>
              <li><button type="button" data-action="edit" data-task-id="1">Редактировать</button></li>
              <li><button type="button" data-action="complete" data-task-id="1">Отметить выполненной</button></li>
              <li><button type="button" data-action="delete" data-task-id="1">Удалить</button></li>
            </menu>
          </div>
        </footer>
      </article>

      <article class="task" aria-labelledby="task-2-title" data-task-id="2" data-priority="medium" data-status="active">
        <header>
          <h4 id="task-2-title">Собрать семантический каркас FocusBoard</h4>
        </header>
        <p>Создать правильную структуру документа с landmarks.</p>
        <footer>
          <p>Приоритет: средний</p>
          
          <button type="button" popovertarget="task-2-menu" popovertargetaction="toggle">
            Действия
          </button>

          <div id="task-2-menu" popover="auto" class="task-menu">
            <menu>
              <li><button type="button" data-action="edit" data-task-id="2">Редактировать</button></li>
              <li><button type="button" data-action="complete" data-task-id="2">Отметить выполненной</button></li>
              <li><button type="button" data-action="delete" data-task-id="2">Удалить</button></li>
            </menu>
          </div>
        </footer>
      </article>
    </section>

    <section aria-labelledby="completed-tasks-heading">
      <h3 id="completed-tasks-heading">Выполненные</h3>

      <article class="task task--completed" aria-labelledby="task-3-title" data-task-id="3" data-priority="low" data-status="completed">
        <header>
          <h4 id="task-3-title">Прочитать введение в Modern HTML 2026</h4>
        </header>
        <p>Ознакомиться с основными идеями книги.</p>
        <footer>
          <p>Выполнено</p>
        </footer>
      </article>
    </section>
  </main>

  <!-- ===== Боковая панель ===== -->
  <aside aria-labelledby="sidebar-heading">
    <h2 id="sidebar-heading">Фильтры и быстрые действия</h2>

    <!-- Фильтры через <details> -->
    <details class="filters" open>
      <summary>Фильтры</summary>

      <form id="filters-form">
        <fieldset>
          <legend>Статус</legend>
          <label>
            <input type="radio" name="status" value="all" checked>
            Все
          </label>
          <label>
            <input type="radio" name="status" value="active">
            Активные
          </label>
          <label>
            <input type="radio" name="status" value="completed">
            Выполненные
          </label>
        </fieldset>

        <fieldset>
          <legend>Приоритет</legend>
          <label>
            <input type="checkbox" name="priority" value="high">
            Высокий
          </label>
          <label>
            <input type="checkbox" name="priority" value="medium">
            Средний
          </label>
          <label>
            <input type="checkbox" name="priority" value="low">
            Низкий
          </label>
        </fieldset>
      </form>
    </details>

    <!-- Быстрые действия -->
    <details class="quick-actions">
      <summary>Быстрые действия</summary>
      <ul>
        <li><button type="button" id="create-task-btn-sidebar">Создать задачу</button></li>
        <li><button type="button" disabled>Создать заметку</button></li>
      </ul>
    </details>
  </aside>

  <!-- ===== Подвал ===== -->
  <footer>
    <p>FocusBoard 2026 · Построен на современном HTML</p>
  </footer>

  <!-- ===== Диалог создания задачи ===== -->
  <dialog id="create-task-dialog" aria-labelledby="create-task-title">
    <form method="dialog" id="create-task-form">
      <header>
        <h2 id="create-task-title">Новая задача</h2>
      </header>

      <p>
        <label for="task-title">Название</label>
        <input type="text" id="task-title" name="title" required placeholder="Что нужно сделать?">
      </p>

      <p>
        <label for="task-description">Описание</label>
        <textarea id="task-description" name="description" rows="3" placeholder="Дополнительные детали (необязательно)"></textarea>
      </p>

      <menu>
        <button type="submit" value="cancel">Отмена</button>
        <button type="submit" value="confirm">Создать</button>
      </menu>
    </form>
  </dialog>

  <script src="scripts/main.js"></script>
</body>
</html>
```

#### `styles/main.css`

```css
/* Базовые стили */

body {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  line-height: 1.5;
  margin: 0;
  padding: 1rem;
  color: #0f172a;
  background-color: #f8fafc;
}

header, nav, main, aside, footer, search {
  margin-bottom: 1.5rem;
}

nav ul {
  display: flex;
  gap: 1rem;
  list-style: none;
  padding: 0;
}

button {
  cursor: pointer;
  font: inherit;
}

/* ===== Диалог ===== */

dialog {
  border: 1px solid #cbd5e1;
  border-radius: 12px;
  padding: 1.5rem;
  max-width: 420px;
  width: 90%;
  box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1);
}

dialog::backdrop {
  background-color: rgb(15 23 42 / 0.5);
}

dialog header {
  margin-bottom: 1rem;
}

dialog menu {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  padding: 0;
  margin-top: 1.5rem;
}

dialog label {
  display: block;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

dialog input,
dialog textarea {
  width: 100%;
  font: inherit;
  padding: 0.5rem 0.75rem;
  border: 1px solid #cbd5e1;
  border-radius: 8px;
  box-sizing: border-box;
}

dialog textarea {
  resize: vertical;
}

/* ===== Popover (контекстное меню задачи) ===== */

.task-menu {
  border: 1px solid #cbd5e1;
  border-radius: 10px;
  padding: 0.35rem;
  background: white;
  box-shadow: 0 10px 15px -3px rgb(0 0 0 / 0.1);
  min-width: 180px;
}

.task-menu menu {
  list-style: none;
  margin: 0;
  padding: 0;
}

.task-menu button {
  display: block;
  width: 100%;
  text-align: left;
  padding: 0.5rem 0.75rem;
  border: none;
  background: transparent;
  border-radius: 6px;
  font: inherit;
}

.task-menu button:hover,
.task-menu button:focus-visible {
  background-color: #f1f5f9;
  outline: none;
}

/* ===== Details / Summary (фильтры) ===== */

details {
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  background: white;
  margin-bottom: 1rem;
  overflow: hidden;
}

details summary {
  padding: 0.75rem 1rem;
  font-weight: 600;
  cursor: pointer;
  list-style: none; /* убираем стандартный маркер в некоторых браузерах */
}

details summary::-webkit-details-marker {
  display: none;
}

details summary::before {
  content: "▸ ";
  display: inline-block;
  transition: transform 0.15s ease;
}

details[open] summary::before {
  transform: rotate(90deg);
}

details form,
details ul {
  padding: 0 1rem 1rem 1rem;
  margin: 0;
}

details fieldset {
  border: none;
  margin: 0 0 1rem 0;
  padding: 0;
}

details legend {
  font-weight: 600;
  margin-bottom: 0.5rem;
}

details label {
  display: block;
  margin-bottom: 0.35rem;
}

details ul {
  list-style: none;
}

details li {
  margin-bottom: 0.35rem;
}
```

#### `scripts/main.js`

```js
// ===== Диалог создания задачи =====
const createTaskBtn = document.getElementById('create-task-btn');
const createTaskBtnSidebar = document.getElementById('create-task-btn-sidebar');
const createTaskDialog = document.getElementById('create-task-dialog');
const createTaskForm = document.getElementById('create-task-form');

function openCreateTaskDialog() {
  createTaskDialog.showModal();
}

createTaskBtn.addEventListener('click', openCreateTaskDialog);
createTaskBtnSidebar.addEventListener('click', openCreateTaskDialog);

createTaskForm.addEventListener('close', () => {
  const returnValue = createTaskDialog.returnValue;

  if (returnValue === 'confirm') {
    const formData = new FormData(createTaskForm);
    const title = formData.get('title');
    const description = formData.get('description');

    console.log('Новая задача:', { title, description });
    createTaskForm.reset();
  }

  if (returnValue === 'cancel') {
    createTaskForm.reset();
  }
});

// ===== Popover: действия с задачами =====
document.addEventListener('click', (event) => {
  const actionBtn = event.target.closest('[data-action]');
  
  if (!actionBtn) return;

  const action = actionBtn.dataset.action;
  const taskId = actionBtn.dataset.taskId;

  console.log(`Действие "${action}" для задачи #${taskId}`);

  const popover = actionBtn.closest('[popover]');
  if (popover) {
    popover.hidePopover();
  }
});

// ===== Фильтры (пока только логирование) =====
const filtersForm = document.getElementById('filters-form');

filtersForm.addEventListener('change', () => {
  const formData = new FormData(filtersForm);
  const status = formData.get('status');
  const priorities = formData.getAll('priority');

  console.log('Фильтры изменены:', { status, priorities });
  // Реальную фильтрацию списка задач добавим в следующих главах
});
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 6. `<details>` и `<summary>` — декларативные раскрывающиеся блоки**

### Что реализовано

- Блок фильтров на основе `<details>` и `<summary>`
- Фильтры по статусу и приоритету
- Секция «Быстрые действия» также через `<details>`
- Стилизация маркера раскрытия
- Кнопка создания задачи продублирована в боковой панели
- Атрибуты `data-priority` и `data-status` у задач (заготовка под фильтрацию)

### Структура проекта

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
└── README.md
```

### Как проверить

1. Откройте страницу.
2. В боковой панели разверните и сверните секции «Фильтры» и «Быстрые действия».
3. Измените фильтры — в консоли появятся выбранные значения.
4. Убедитесь, что всё работает даже при отключённом JavaScript (раскрытие/сворачивание).

## Следующие шаги

**Часть 3. Формы нового поколения**

- Глава 7. Современная архитектура веб-форм
- Улучшение формы создания задачи и валидации
```

---

### Важные замечания

- `<details>` работает полностью без JavaScript — это идеальный пример Progressive Enhancement.
- Мы добавили атрибуты `data-status` и `data-priority` задачам заранее, чтобы позже было легко реализовать фильтрацию.
- Стили маркера сделаны через `::before`, чтобы получить более современный вид.
- Кнопка «Создать задачу» теперь доступна и в основном контенте, и в боковой панели.

На этом завершается **Часть 2. Нативные интерактивные элементы**.  
В следующей части мы углубимся в современные формы.