**Глава 14. View Transition API — плавные переходы между состояниями**

### Главная идея главы

**View Transition API** позволяет создавать плавные визуальные переходы между состояниями интерфейса или даже между страницами, делегируя сложную работу по анимации браузеру. Вместо ручного управления opacity, transform и «миганием» контента мы описываем переход декларативно и запускаем его через небольшой JavaScript-API.

Основные сценарии использования:
- переключение разделов интерфейса;
- обновление списка задач;
- переход между состояниями «список → детали»;
- в перспективе — MPA-переходы между страницами.

### Базовые возможности

- `document.startViewTransition(callback)`
- псевдоэлементы `::view-transition-old()` и `::view-transition-new()`
- анимация через CSS
- поддержка progressive enhancement (если API нет — интерфейс просто обновляется без анимации)

### Практическая задача главы

Добавить в FocusBoard плавный переход при переключении между разделами:

- Задачи
- Заметки
- Настройки

Пока разделы «Заметки» и «Настройки» будут простыми заглушками, но сам переход между состояниями главного контента реализуем через View Transition API.

### Структура проекта после Главы 14

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
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

  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="dns-prefetch" href="https://fonts.googleapis.com">

  <link rel="preload" href="styles/main.css" as="style" fetchpriority="high">
  <link rel="stylesheet" href="styles/main.css">
  <link rel="prefetch" href="scripts/main.js" as="script" fetchpriority="low">
</head>
<body>
  <header>
    <h1>FocusBoard</h1>
    <p>Персональная панель продуктивности</p>
  </header>

  <nav aria-label="Основная навигация">
    <ul>
      <li><a href="#tasks" data-section="tasks" aria-current="page">Задачи</a></li>
      <li><a href="#notes" data-section="notes">Заметки</a></li>
      <li><a href="#settings" data-section="settings">Настройки</a></li>
    </ul>
  </nav>

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

  <!-- Основная область, которую будем анимировать -->
  <main id="app-main">
    <!-- Секция: Задачи -->
    <section id="tasks" class="app-section" data-section-panel="tasks">
      <header>
        <h2>Задачи</h2>
        <p>Активные задачи на сегодня</p>
        <p>
          <button type="button" id="create-task-btn">Создать задачу</button>
        </p>
      </header>

      <section aria-labelledby="active-tasks-heading">
        <h3 id="active-tasks-heading">Активные</h3>

        <article class="task" aria-labelledby="task-1-title" data-task-id="1" data-priority="high" data-status="active">
          <header>
            <h4 id="task-1-title">Изучить современный HTML</h4>
          </header>

          <p class="task-media">
            <picture>
              <source 
                srcset="
                  images/task-320.svg 320w,
                  images/task-640.svg 640w,
                  images/task-960.svg 960w
                "
                sizes="(max-width: 600px) 100vw, 320px"
                type="image/svg+xml"
              >
              <img 
                src="images/task-640.svg"
                alt="Иллюстрация к задаче по изучению HTML"
                width="320"
                height="180"
                loading="lazy"
                decoding="async"
                fetchpriority="low"
              >
            </picture>
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
    </section>

    <!-- Секция: Заметки -->
    <section id="notes" class="app-section" data-section-panel="notes" hidden>
      <header>
        <h2>Заметки</h2>
        <p>Быстрые заметки и база знаний</p>
      </header>
      <p>Раздел заметок появится в следующих главах. Пока это заглушка для демонстрации View Transition API.</p>
    </section>

    <!-- Секция: Настройки -->
    <section id="settings" class="app-section" data-section-panel="settings" hidden>
      <header>
        <h2>Настройки</h2>
        <p>Параметры приложения и профиля</p>
      </header>
      <p>Раздел настроек появится позже. Сейчас он нужен для демонстрации плавных переходов.</p>
    </section>
  </main>

  <aside aria-labelledby="sidebar-heading">
    <h2 id="sidebar-heading">Фильтры и быстрые действия</h2>

    <details class="filters" open>
      <summary>Фильтры</summary>
      <form id="filters-form" aria-label="Фильтры задач">
        <fieldset>
          <legend>Статус</legend>
          <label><input type="radio" name="status" value="all" checked> Все</label>
          <label><input type="radio" name="status" value="active"> Активные</label>
          <label><input type="radio" name="status" value="completed"> Выполненные</label>
        </fieldset>
        <fieldset>
          <legend>Приоритет</legend>
          <label><input type="checkbox" name="priority" value="high"> Высокий</label>
          <label><input type="checkbox" name="priority" value="medium"> Средний</label>
          <label><input type="checkbox" name="priority" value="low"> Низкий</label>
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

  <footer>
    <p>FocusBoard 2026 · Построен на современном HTML</p>
  </footer>

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
          <label><input type="radio" name="priority" value="high"> Высокий</label>
          <label><input type="radio" name="priority" value="medium" checked> Средний</label>
          <label><input type="radio" name="priority" value="low"> Низкий</label>
        </div>
      </fieldset>

      <menu>
        <button type="submit" value="cancel">Отмена</button>
        <button type="submit" value="confirm">Создать</button>
      </menu>
    </form>
  </dialog>

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

.task-media {
  margin: 0.75rem 0;
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

/* ===== View Transitions ===== */

@supports (view-transition-name: none) {
  /* Имя корневого перехода */
  :root {
    view-transition-name: root;
  }

  /* Основная область контента */
  #app-main {
    view-transition-name: app-main;
  }

  /* Плавное появление/исчезновение */
  ::view-transition-old(app-main),
  ::view-transition-new(app-main) {
    animation-duration: 220ms;
    animation-timing-function: ease;
  }

  ::view-transition-old(app-main) {
    animation-name: fade-out;
  }

  ::view-transition-new(app-main) {
    animation-name: fade-in;
  }

  @keyframes fade-out {
    from { opacity: 1; transform: translateY(0); }
    to   { opacity: 0; transform: translateY(-6px); }
  }

  @keyframes fade-in {
    from { opacity: 0; transform: translateY(6px); }
    to   { opacity: 1; transform: translateY(0); }
  }
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
// ===== Навигация по разделам + View Transition API =====
const sectionLinks = document.querySelectorAll('[data-section]');
const sectionPanels = document.querySelectorAll('[data-section-panel]');

function setActiveSection(sectionId) {
  sectionPanels.forEach((panel) => {
    panel.hidden = panel.dataset.sectionPanel !== sectionId;
  });

  sectionLinks.forEach((link) => {
    if (link.dataset.section === sectionId) {
      link.setAttribute('aria-current', 'page');
    } else {
      link.removeAttribute('aria-current');
    }
  });

  // Обновляем title для понятности
  const titles = {
    tasks: 'Задачи',
    notes: 'Заметки',
    settings: 'Настройки'
  };
  document.title = `FocusBoard 2026 — ${titles[sectionId] || 'Приложение'}`;
}

function navigateToSection(sectionId) {
  // Progressive Enhancement: если View Transition API недоступен
  if (!document.startViewTransition) {
    setActiveSection(sectionId);
    return;
  }

  document.startViewTransition(() => {
    setActiveSection(sectionId);
  });
}

sectionLinks.forEach((link) => {
  link.addEventListener('click', (event) => {
    event.preventDefault();
    const sectionId = link.dataset.section;
    navigateToSection(sectionId);

    // Обновляем hash без перезагрузки
    history.pushState({ sectionId }, '', `#${sectionId}`);
  });
});

// Поддержка кнопки «Назад/Вперёд»
window.addEventListener('popstate', (event) => {
  const sectionId = event.state?.sectionId || location.hash.replace('#', '') || 'tasks';
  navigateToSection(sectionId);
});

// Инициализация при загрузке
const initialSection = location.hash.replace('#', '') || 'tasks';
setActiveSection(initialSection);

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

createTaskBtn?.addEventListener('click', openCreateTaskDialog);
createTaskBtnSidebar?.addEventListener('click', openCreateTaskDialog);

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
  if (popover) popover.hidePopover();
});

// ===== Фильтры =====
const filtersForm = document.getElementById('filters-form');
filtersForm?.addEventListener('change', () => {
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

**Глава 14. View Transition API — плавные переходы между состояниями**

### Что реализовано

- Переключение разделов «Задачи / Заметки / Настройки»
- Плавные переходы через `document.startViewTransition()`
- CSS-анимация `::view-transition-old/new`
- Progressive Enhancement: без поддержки API интерфейс просто переключается
- Обновление `history.pushState` и поддержка кнопок Назад/Вперёд
- Заглушки разделов Notes и Settings

### Структура проекта

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

### Как проверить

1. Откройте страницу в браузере с поддержкой View Transitions.
2. Переключайтесь между «Задачи», «Заметки» и «Настройки».
3. Обратите внимание на плавное исчезновение/появление основного контента.
4. Проверьте кнопки браузера «Назад» и «Вперёд».
5. В браузере без поддержки API разделы должны переключаться без анимации, но корректно.

## Следующие шаги

- Глава 15. Navigation API
- Более современное управление историей и маршрутизацией
```

---

### Важные замечания

- View Transition API используем с проверкой поддержки — это правильный Progressive Enhancement.
- Анимируем осмысленную область (`#app-main`), а не всё подряд.
- `view-transition-name` позволяет браузеру сопоставлять старое и новое состояние элементов.
- Пока анимация простая (fade + небольшой сдвиг). Позже её можно сделать более «кинематографичной».

В следующей главе перейдём к **Navigation API** и сделаем управление историей ещё более современным и надёжным.