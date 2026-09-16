**Глава 8. Новые атрибуты форм: тонкая настройка UX и поведения**

### Главная идея главы

Современный HTML даёт разработчику множество атрибутов, которые позволяют тонко настраивать поведение полей ввода — особенно на мобильных устройствах. С помощью `inputmode`, `enterkeyhint`, `autocomplete`, `autocapitalize` и других атрибутов можно значительно улучшить пользовательский опыт без написания JavaScript.

Эти атрибуты помогают браузеру:
- показать правильную виртуальную клавиатуру;
- подсказать, какое действие выполняет клавиша Enter;
- корректно предлагать автозаполнение;
- управлять капитализацией и другими аспектами ввода.

### Ключевые атрибуты

| Атрибут              | Назначение                                      | Пример значений |
|----------------------|--------------------------------------------------|-----------------|
| `inputmode`          | Тип виртуальной клавиатуры                       | `text`, `search`, `email`, `numeric`, `tel`, `url` |
| `enterkeyhint`       | Подпись/действие клавиши Enter                   | `enter`, `done`, `go`, `next`, `previous`, `search`, `send` |
| `autocomplete`       | Подсказки автозаполнения                         | `off`, `name`, `email`, `organization` и др. |
| `autocapitalize`     | Автоматическая капитализация                     | `off`, `sentences`, `words`, `characters` |
| `spellcheck`         | Проверка орфографии                              | `true` / `false` |
| `autocorrect`        | Автокоррекция (поддержка ограничена)             | `on` / `off` |

### Практическая задача главы

Улучшить поля форм FocusBoard с помощью современных атрибутов:

1. Поле поиска — правильный `inputmode` и `enterkeyhint`.
2. Поле названия задачи — улучшить ввод и автозаполнение.
3. Поле описания — настроить поведение клавиатуры.
4. Добавить атрибуты, полезные для будущих форм (настройки профиля и т.д.).

### Структура проекта после Главы 8

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
└── README.md
```

Структура файлов не изменилась.

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
      <input 
        type="search" 
        id="site-search" 
        name="q" 
        placeholder="Поиск задач и заметок…"
        inputmode="search"
        enterkeyhint="search"
        autocomplete="off"
        spellcheck="false"
      >
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

      <fieldset>
        <legend>Основная информация</legend>

        <p>
          <label for="task-title">Название <span aria-hidden="true">*</span></label>
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
            aria-describedby="task-title-error"
          >
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
          ></textarea>
        </p>
      </fieldset>

      <fieldset>
        <legend>Приоритет</legend>

        <p>
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
        </p>
      </fieldset>

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

/* Состояния валидации */
dialog input:user-invalid,
dialog textarea:user-invalid {
  border-color: #dc2626;
}

dialog .error-message {
  display: block;
  color: #dc2626;
  font-size: 0.875rem;
  margin-top: 0.25rem;
  min-height: 1.25rem;
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

function openCreateTaskDialog() {
  createTaskForm.reset();
  taskTitleError.textContent = '';
  createTaskDialog.showModal();
  taskTitleInput.focus();
}

createTaskBtn.addEventListener('click', openCreateTaskDialog);
createTaskBtnSidebar.addEventListener('click', openCreateTaskDialog);

// Кастомная проверка перед закрытием диалога
createTaskForm.addEventListener('submit', (event) => {
  if (event.submitter?.value === 'cancel') {
    taskTitleError.textContent = '';
    return;
  }

  if (!createTaskForm.checkValidity()) {
    event.preventDefault();

    if (!taskTitleInput.validity.valid) {
      if (taskTitleInput.validity.valueMissing) {
        taskTitleError.textContent = 'Введите название задачи';
      } else if (taskTitleInput.validity.tooShort) {
        taskTitleError.textContent = 'Название должно содержать не менее 3 символов';
      } else {
        taskTitleError.textContent = 'Проверьте название задачи';
      }
      taskTitleInput.focus();
    }
  } else {
    taskTitleError.textContent = '';
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
  taskTitleError.textContent = '';
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

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 8. Новые атрибуты форм: тонкая настройка UX и поведения**

### Что реализовано

- Улучшено поле поиска: `inputmode="search"`, `enterkeyhint="search"`
- Поле названия задачи: `inputmode`, `enterkeyhint="next"`, `autocapitalize`, `spellcheck`
- Поле описания: настроены `enterkeyhint` и `autocapitalize`
- Отключено ненужное автозаполнение там, где оно мешает
- Подготовка к удобному вводу на мобильных устройствах

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

1. Откройте страницу на мобильном устройстве или в режиме эмуляции.
2. Тапните по полю поиска — должна появиться поисковая клавиатура.
3. Откройте диалог создания задачи и проверьте поведение клавиши Enter на разных полях.
4. Обратите внимание на автоматическую капитализацию предложений.

## Следующие шаги

- Глава 9. Доступность веб-форм (A11y)
- Углублённая работа с доступностью форм и сообщениями об ошибках
```

---

### Важные замечания

- `inputmode` не заменяет `type`, а дополняет его. Он влияет именно на виртуальную клавиатуру.
- `enterkeyhint` особенно полезен в многошаговых формах и на мобильных устройствах.
- `autocomplete="off"` стоит использовать осознанно — только там, где автозаполнение действительно мешает.
- Эти атрибуты почти не требуют JavaScript и отлично работают в рамках Progressive Enhancement.

В следующей главе мы сосредоточимся на глубокой доступности форм.