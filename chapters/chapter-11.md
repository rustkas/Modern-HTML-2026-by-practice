**Глава 11. Управление приоритетами загрузки ресурсов**

### Главная идея главы

Браузер самостоятельно расставляет приоритеты загрузки ресурсов, но не всегда угадывает намерения разработчика. Современный HTML позволяет явно подсказывать, что важно, а что можно загрузить позже, с помощью атрибутов:

- `fetchpriority`
- `loading`
- `decoding`

Это особенно важно для метрик Core Web Vitals (LCP, INP, CLS) и общего ощущения скорости интерфейса.

### Основные атрибуты

| Атрибут | Значения | Назначение |
|---------|----------|----------|
| `fetchpriority` | `high`, `low`, `auto` | Приоритет загрузки ресурса |
| `loading` | `lazy`, `eager` | Ленивая или немедленная загрузка |
| `decoding` | `async`, `sync`, `auto` | Как декодировать изображение |

### Когда что использовать

- `fetchpriority="high"` — для LCP-изображения, критического CSS/шрифта
- `fetchpriority="low"` — для второстепенных изображений, иконок, ниже-fold контента
- `loading="lazy"` — для изображений и iframe, которые не видны сразу
- `loading="eager"` — для того, что должно загрузиться сразу (по умолчанию для большинства ресурсов)

### Практическая задача главы

1. Явно задать приоритеты для CSS и скрипта.
2. Подготовить атрибуты для будущих изображений.
3. Добавить в интерфейс пример изображения задачи с правильными атрибутами загрузки.
4. Зафиксировать подход к приоритизации ресурсов в проекте.

### Структура проекта после Главы 11

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── images/
│   └── task-placeholder.svg
└── README.md
```

Добавлена папка `images/` с простым placeholder-изображением.

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

  <!-- ===== Приоритеты и спекулятивная загрузка ===== -->

  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="dns-prefetch" href="https://fonts.googleapis.com">

  <!-- Критический CSS — высокий приоритет -->
  <link rel="preload" href="styles/main.css" as="style" fetchpriority="high">
  <link rel="stylesheet" href="styles/main.css">

  <!-- Скрипт не критичен для LCP -->
  <link rel="prefetch" href="scripts/main.js" as="script" fetchpriority="low">
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
      <input 
        type="search" 
        id="site-search" 
        name="q" 
        placeholder="Поиск задач и заметок…"
        inputmode="search"
        enterkeyhint="search"
        autocomplete="off"
        spellcheck="false"
        aria-describedby="site-search-hint"
      >
      <small id="site-search-hint">Введите название задачи или заметки</small>
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

        <!-- Пример изображения с приоритетами загрузки -->
        <p>
          <img 
            src="images/task-placeholder.svg" 
            alt="Иллюстрация к задаче по изучению HTML"
            width="320"
            height="180"
            loading="lazy"
            decoding="async"
            fetchpriority="low"
          >
        </p>

        <p>Разобрать семантику, dialog, popover и Web Components.</p>
        <footer>
          <p>Приоритет: высокий</p>
          
          <button type="button" popovertarget="task-1-menu" popovertargetaction="toggle" aria-haspopup="menu">
            Действия
          </button>

          <div id="task-1-menu" popover="auto" class="task-menu" role="menu" aria-label="Действия с задачей">
            <menu>
              <li role="none"><button type="button" role="menuitem" data-action="edit" data-task-id="1">Редактировать</button></li>
              <li role="none"><button type="button" role="menuitem" data-action="complete" data-task-id="1">Отметить выполненной</button></li>
              <li role="none"><button type="button" role="menuitem" data-action="delete" data-task-id="1">Удалить</button></li>
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
          
          <button type="button" popovertarget="task-2-menu" popovertargetaction="toggle" aria-haspopup="menu">
            Действия
          </button>

          <div id="task-2-menu" popover="auto" class="task-menu" role="menu" aria-label="Действия с задачей">
            <menu>
              <li role="none"><button type="button" role="menuitem" data-action="edit" data-task-id="2">Редактировать</button></li>
              <li role="none"><button type="button" role="menuitem" data-action="complete" data-task-id="2">Отметить выполненной</button></li>
              <li role="none"><button type="button" role="menuitem" data-action="delete" data-task-id="2">Удалить</button></li>
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

    <details class="filters" open>
      <summary>Фильтры</summary>

      <form id="filters-form" aria-label="Фильтры задач">
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
    <form method="dialog" id="create-task-form" novalidate>
      <header>
        <h2 id="create-task-title">Новая задача</h2>
      </header>

      <div id="form-error-summary" class="form-error-summary" role="alert" hidden></div>

      <fieldset>
        <legend>Основная информация</legend>

        <p>
          <label for="task-title">
            Название
            <span class="required-marker" aria-hidden="true">*</span>
            <span class="visually-hidden"> (обязательное поле)</span>
          </label>
          <input 
            type="text" 
            id="task-title" 
            name="title" 
            required 
            minlength="3" 
            maxlength="120"
            placeholder="Что нужно сделать?"
            inputmode="text"
            enterkeyhint="next"
            autocomplete="off"
            autocapitalize="sentences"
            spellcheck="true"
            aria-required="true"
            aria-describedby="task-title-hint task-title-error"
          >
          <small id="task-title-hint">Не менее 3 символов</small>
          <span id="task-title-error" class="error-message" role="alert"></span>
        </p>

        <p>
          <label for="task-description">Описание</label>
          <textarea 
            id="task-description" 
            name="description" 
            rows="3" 
            maxlength="500"
            placeholder="Дополнительные детали (необязательно)"
            enterkeyhint="enter"
            autocapitalize="sentences"
            spellcheck="true"
            aria-describedby="task-description-hint"
          ></textarea>
          <small id="task-description-hint">Необязательное поле, до 500 символов</small>
        </p>
      </fieldset>

      <fieldset>
        <legend>Приоритет</legend>

        <div role="radiogroup" aria-label="Выберите приоритет задачи">
          <label>
            <input type="radio" name="priority" value="high">
            Высокий
          </label>
          <label>
            <input type="radio" name="priority" value="medium" checked>
            Средний
          </label>
          <label>
            <input type="radio" name="priority" value="low">
            Низкий
          </label>
        </div>
      </fieldset>

      <menu>
        <button type="submit" value="cancel">Отмена</button>
        <button type="submit" value="confirm">Создать</button>
      </menu>
    </form>
  </dialog>

  <!-- Скрипт с низким приоритетом для первой отрисовки -->
  <script src="scripts/main.js" defer fetchpriority="low"></script>
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

img {
  max-width: 100%;
  height: auto;
  display: block;
  border-radius: 8px;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

.required-marker {
  color: #dc2626;
}

/* ===== Диалог ===== */

dialog {
  border: 1px solid #cbd5e1;
  border-radius: 12px;
  padding: 1.5rem;
  max-width: 480px;
  width: 90%;
  box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1);
}

dialog::backdrop {
  background-color: rgb(15 23 42 / 0.5);
}

dialog header {
  margin-bottom: 1rem;
}

dialog fieldset {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  margin: 0 0 1rem 0;
  padding: 1rem;
}

dialog legend {
  font-weight: 600;
  padding: 0 0.35rem;
}

dialog menu {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  padding: 0;
  margin-top: 1.25rem;
}

dialog label {
  display: block;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

dialog input[type="text"],
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

dialog input[type="radio"] + label,
dialog label:has(input[type="radio"]) {
  font-weight: 400;
  display: inline-flex;
  align-items: center;
  gap: 0.35rem;
  margin-right: 1rem;
}

dialog input:user-invalid,
dialog textarea:user-invalid,
dialog input[aria-invalid="true"] {
  border-color: #dc2626;
}

dialog .error-message {
  display: block;
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  min-height: 1.25rem;
}

dialog small {
  display: block;
  color: #64748b;
  font-size: 0.875rem;
  margin-top: 0.25rem;
}

.form-error-summary {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 0.75rem 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
}

/* ===== Popover ===== */

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

/* ===== Details / Summary ===== */

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
  list-style: none;
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
const taskTitleInput = document.getElementById('task-title');
const taskTitleError = document.getElementById('task-title-error');
const formErrorSummary = document.getElementById('form-error-summary');

function openCreateTaskDialog() {
  createTaskForm.reset();
  clearFormErrors();
  createTaskDialog.showModal();
  taskTitleInput.focus();
}

function clearFormErrors() {
  taskTitleError.textContent = '';
  taskTitleInput.removeAttribute('aria-invalid');
  formErrorSummary.hidden = true;
  formErrorSummary.textContent = '';
}

createTaskBtn.addEventListener('click', openCreateTaskDialog);
createTaskBtnSidebar.addEventListener('click', openCreateTaskDialog);

createTaskForm.addEventListener('submit', (event) => {
  if (event.submitter?.value === 'cancel') {
    clearFormErrors();
    return;
  }

  if (!createTaskForm.checkValidity()) {
    event.preventDefault();

    if (!taskTitleInput.validity.valid) {
      let message = 'Проверьте название задачи';

      if (taskTitleInput.validity.valueMissing) {
        message = 'Введите название задачи';
      } else if (taskTitleInput.validity.tooShort) {
        message = 'Название должно содержать не менее 3 символов';
      }

      taskTitleError.textContent = message;
      taskTitleInput.setAttribute('aria-invalid', 'true');

      formErrorSummary.textContent = 'Исправьте ошибки в форме перед созданием задачи.';
      formErrorSummary.hidden = false;

      taskTitleInput.focus();
    }
  } else {
    clearFormErrors();
  }
});

createTaskForm.addEventListener('close', () => {
  const returnValue = createTaskDialog.returnValue;

  if (returnValue === 'confirm') {
    const formData = new FormData(createTaskForm);
    const task = {
      title: formData.get('title'),
      description: formData.get('description'),
      priority: formData.get('priority')
    };

    console.log('Новая задача:', task);
  }

  createTaskForm.reset();
  clearFormErrors();
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

// ===== Фильтры =====
const filtersForm = document.getElementById('filters-form');

filtersForm.addEventListener('change', () => {
  const formData = new FormData(filtersForm);
  const status = formData.get('status');
  const priorities = formData.getAll('priority');

  console.log('Фильтры изменены:', { status, priorities });
});
```

#### `images/task-placeholder.svg`

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="320" height="180" viewBox="0 0 320 180" role="img" aria-label="Заглушка изображения задачи">
  <rect width="320" height="180" fill="#e2e8f0"/>
  <rect x="20" y="20" width="280" height="140" rx="12" fill="#cbd5e1"/>
  <text x="160" y="100" text-anchor="middle" font-family="system-ui, sans-serif" font-size="18" fill="#475569">
    Task image
  </text>
</svg>
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 11. Управление приоритетами загрузки ресурсов**

### Что реализовано

- `fetchpriority="high"` для критического CSS
- `fetchpriority="low"` для скрипта и второстепенных изображений
- `loading="lazy"` и `decoding="async"` для изображений
- Пример изображения задачи с правильными атрибутами производительности
- Добавлена папка `images/`

### Структура проекта

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── images/
│   └── task-placeholder.svg
└── README.md
```

### Как проверить

1. Откройте DevTools → Network.
2. Обновите страницу с отключённым кэшем.
3. Проверьте приоритеты загрузки CSS, JS и изображения.
4. Убедитесь, что изображение имеет атрибуты `loading="lazy"` и `fetchpriority="low"`.

## Следующие шаги

- Глава 12. Современная работа с изображениями и медиаконтентом
- `srcset`, `sizes`, элемент `<picture>` и адаптивные изображения
```

---

### Важные замечания

- `fetchpriority` — это подсказка, а не жёсткая команда. Браузер учитывает её вместе с другими факторами.
- Для LCP-изображения обычно ставят `fetchpriority="high"` и `loading="eager"`.
- Для ниже-fold изображений лучше `loading="lazy"` + `fetchpriority="low"`.
- `decoding="async"` помогает не блокировать основной поток при декодировании картинок.

В следующей главе мы углубимся в адаптивные изображения и современную работу с медиаконтентом.