**Глава 19. HTML Templates и механизм переиспользования**

### Главная идея главы

Элементы **`<template>`** и **`<slot>`** — это фундамент переиспользования в современном HTML.  
`<template>` хранит инертную разметку, которая не отображается и не выполняется, пока её явно не используют.  
`<slot>` задаёт публичные точки расширения компонента.

Вместе они позволяют:

- описывать структуру один раз и переиспользовать её многократно;
- разделять «скелет» компонента и внешний контент;
- собирать UI как систему контрактов, а не как набор копипасты;
- готовить основу для дизайн-системы и SSR-friendly компонентов.

### Что важно понимать

1. Содержимое `<template>` не является активным DOM, пока не клонировано.
2. `template.content` — это `DocumentFragment`.
3. `<slot>` — это API расширения компонента.
4. Именованные слоты задают контракт:
   - `slot="title"`
   - `slot="description"`
   - `slot="actions"`
5. Хороший компонент делает переиспользование простым: снаружи передают данные/контент, внутри сохраняется стабильная структура.

### Практическая задача главы

1. Вынести шаблон `task-card` в явный переиспользуемый вид.
2. Добавить шаблон для быстрого создания новой карточки задачи.
3. Научить приложение добавлять задачу в список на основе `<template>`.
4. Зафиксировать slots как публичный контракт компонента.
5. Убрать дублирование разметки при создании новых задач.

### Структура проекта после Главы 19

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── components/
│   └── task-card.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

Структура файлов не изменилась, но появились шаблоны переиспользования в `index.html` и логика клонирования в `main.js`.

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

  <main id="app-main">
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
        <div id="active-tasks-list">
          <task-card data-task-id="1" data-priority="high" data-status="active">
            <h4 slot="title">Изучить современный HTML</h4>
            <p class="task-media" slot="media">
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
            <p slot="description">Разобрать семантику, dialog, popover и Web Components.</p>
            <span slot="priority-label">Приоритет: высокий</span>
            <div slot="actions">
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
            </div>
          </task-card>

          <task-card data-task-id="2" data-priority="medium" data-status="active">
            <h4 slot="title">Собрать семантический каркас FocusBoard</h4>
            <p slot="description">Создать правильную структуру документа с landmarks.</p>
            <span slot="priority-label">Приоритет: средний</span>
            <div slot="actions">
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
            </div>
          </task-card>
        </div>
      </section>

      <section aria-labelledby="completed-tasks-heading">
        <h3 id="completed-tasks-heading">Выполненные</h3>
        <div id="completed-tasks-list">
          <task-card data-task-id="3" data-priority="low" data-status="completed">
            <h4 slot="title">Прочитать введение в Modern HTML 2026</h4>
            <p slot="description">Ознакомиться с основными идеями книги.</p>
            <span slot="priority-label">Выполнено</span>
          </task-card>
        </div>
      </section>
    </section>

    <section id="notes" class="app-section" data-section-panel="notes" hidden>
      <header>
        <h2>Заметки</h2>
        <p>Быстрые заметки и база знаний</p>
      </header>
      <p>Раздел заметок появится в следующих главах.</p>
    </section>

    <section id="settings" class="app-section" data-section-panel="settings" hidden>
      <header>
        <h2>Настройки</h2>
        <p>Параметры приложения и профиля</p>
      </header>
      <p>Раздел настроек появится позже.</p>
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

  <!-- ===== Переиспользуемый шаблон новой задачи ===== -->
  <template id="task-card-template">
    <task-card data-status="active">
      <h4 slot="title"></h4>
      <p slot="description"></p>
      <span slot="priority-label"></span>
      <div slot="actions">
        <button type="button" data-task-actions-btn aria-haspopup="menu">Действия</button>
        <div data-task-menu popover="auto" class="task-menu" role="menu" aria-label="Действия с задачей">
          <menu>
            <li role="none"><button type="button" role="menuitem" data-action="edit">Редактировать</button></li>
            <li role="none"><button type="button" role="menuitem" data-action="complete">Отметить выполненной</button></li>
            <li role="none"><button type="button" role="menuitem" data-action="delete">Удалить</button></li>
          </menu>
        </div>
      </div>
    </task-card>
  </template>

  <script src="components/task-card.js" type="module"></script>
  <script src="scripts/main.js" defer fetchpriority="low"></script>
</body>
</html>
```

#### `components/task-card.js`

```js
const taskCardTemplate = document.createElement('template');
taskCardTemplate.innerHTML = `
  <style>
    :host {
      display: block;
      border: 1px solid var(--task-border, #e2e8f0);
      border-radius: var(--task-radius, 12px);
      background: var(--task-bg, #fff);
      padding: var(--task-padding, 1rem);
      margin-bottom: 1rem;
      color: var(--task-text, inherit);
      box-shadow: var(--task-shadow, none);
    }

    :host([data-status="completed"]) {
      opacity: 0.75;
    }

    :host([data-priority="high"]) {
      border-color: color-mix(in srgb, #ef4444 35%, #e2e8f0);
    }

    .task {
      display: grid;
      gap: 0.75rem;
    }

    .meta {
      display: flex;
      justify-content: space-between;
      gap: 1rem;
      align-items: center;
    }

    .priority {
      font-size: 0.875rem;
      color: #475569;
    }

    .priority[data-priority="high"] {
      color: #b91c1c;
      font-weight: 600;
    }

    .priority[data-priority="medium"] {
      color: #b45309;
    }

    .priority[data-priority="low"] {
      color: #64748b;
    }

    button {
      font: inherit;
      cursor: pointer;
    }

    ::slotted([slot="title"]) {
      margin: 0;
      font-size: 1.05rem;
      line-height: 1.3;
    }

    ::slotted([slot="description"]) {
      margin: 0;
      color: #334155;
    }

    ::slotted([slot="media"]) {
      margin: 0;
    }
  </style>

  <article class="task" part="task">
    <header part="header">
      <slot name="title"></slot>
    </header>

    <div part="media">
      <slot name="media"></slot>
    </div>

    <div part="description">
      <slot name="description"></slot>
    </div>

    <footer class="meta" part="footer">
      <div class="priority" data-priority-label part="priority">
        <slot name="priority-label">Приоритет не указан</slot>
      </div>
      <div part="actions">
        <slot name="actions"></slot>
      </div>
    </footer>
  </article>
`;

class TaskCard extends HTMLElement {
  static get observedAttributes() {
    return ['data-priority', 'data-status', 'data-task-id'];
  }

  constructor() {
    super();

    if (!this.shadowRoot) {
      this.attachShadow({ mode: 'open' });
      this.shadowRoot.appendChild(taskCardTemplate.content.cloneNode(true));
    }
  }

  connectedCallback() {
    this.#syncPriorityState();
    this.#syncA11y();
  }

  attributeChangedCallback(name) {
    if (name === 'data-priority') {
      this.#syncPriorityState();
    }

    if (name === 'data-status' || name === 'data-task-id') {
      this.#syncA11y();
    }
  }

  #syncPriorityState() {
    const priority = this.getAttribute('data-priority') || 'medium';
    const priorityEl = this.shadowRoot?.querySelector('[data-priority-label]');
    if (priorityEl) {
      priorityEl.setAttribute('data-priority', priority);
    }
  }

  #syncA11y() {
    if (!this.hasAttribute('role')) {
      this.setAttribute('role', 'article');
    }
  }
}

if (!customElements.get('task-card')) {
  customElements.define('task-card', TaskCard);
}

export { TaskCard };
```

#### `scripts/main.js`

```js
// ===== Утилиты навигации =====
const sectionLinks = document.querySelectorAll('[data-section]');
const sectionPanels = document.querySelectorAll('[data-section-panel]');
const supportedSections = new Set(['tasks', 'notes', 'settings']);

function getSectionFromUrl(url = location.href) {
  const hash = new URL(url, location.origin).hash.replace('#', '');
  return supportedSections.has(hash) ? hash : 'tasks';
}

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

  const titles = {
    tasks: 'Задачи',
    notes: 'Заметки',
    settings: 'Настройки'
  };
  document.title = `FocusBoard 2026 — ${titles[sectionId] || 'Приложение'}`;
}

function renderSection(sectionId) {
  if (document.startViewTransition) {
    document.startViewTransition(() => setActiveSection(sectionId));
  } else {
    setActiveSection(sectionId);
  }
}

function navigateToSection(sectionId) {
  const nextHash = `#${sectionId}`;

  if (window.navigation) {
    const currentSection = getSectionFromUrl(navigation.currentEntry?.url || location.href);
    if (currentSection === sectionId) {
      renderSection(sectionId);
      return;
    }

    navigation.navigate(nextHash, {
      history: 'push',
      info: { sectionId }
    });
    return;
  }

  history.pushState({ sectionId }, '', nextHash);
  renderSection(sectionId);
}

if (window.navigation) {
  navigation.addEventListener('navigate', (event) => {
    if (!event.canIntercept) return;
    const sectionId = getSectionFromUrl(event.destination.url);
    if (!supportedSections.has(sectionId)) return;

    event.intercept({
      handler() {
        renderSection(sectionId);
      }
    });
  });
} else {
  window.addEventListener('popstate', (event) => {
    const sectionId = event.state?.sectionId || getSectionFromUrl();
    renderSection(sectionId);
  });
}

sectionLinks.forEach((link) => {
  link.addEventListener('click', (event) => {
    event.preventDefault();
    navigateToSection(link.dataset.section);
  });
});

renderSection(getSectionFromUrl());

// ===== Шаблоны и создание задач =====
const taskCardTemplate = document.getElementById('task-card-template');
const activeTasksList = document.getElementById('active-tasks-list');
let taskIdSequence = 100;

const priorityLabels = {
  high: 'Приоритет: высокий',
  medium: 'Приоритет: средний',
  low: 'Приоритет: низкий'
};

function createTaskCard({ title, description, priority = 'medium' }) {
  const fragment = taskCardTemplate.content.cloneNode(true);
  const card = fragment.querySelector('task-card');
  const titleEl = fragment.querySelector('[slot="title"]');
  const descriptionEl = fragment.querySelector('[slot="description"]');
  const priorityEl = fragment.querySelector('[slot="priority-label"]');
  const actionsBtn = fragment.querySelector('[data-task-actions-btn]');
  const menu = fragment.querySelector('[data-task-menu]');
  const actionButtons = fragment.querySelectorAll('[data-action]');

  const taskId = String(++taskIdSequence);
  const menuId = `task-${taskId}-menu`;

  card.setAttribute('data-task-id', taskId);
  card.setAttribute('data-priority', priority);
  card.setAttribute('data-status', 'active');

  titleEl.textContent = title;
  descriptionEl.textContent = description || 'Без описания';
  priorityEl.textContent = priorityLabels[priority] || priorityLabels.medium;

  menu.id = menuId;
  actionsBtn.setAttribute('popovertarget', menuId);
  actionsBtn.setAttribute('popovertargetaction', 'toggle');

  actionButtons.forEach((btn) => {
    btn.dataset.taskId = taskId;
  });

  return fragment;
}

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
      priority: formData.get('priority') || 'medium'
    };

    const cardFragment = createTaskCard(task);
    activeTasksList?.prepend(cardFragment);
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

#### `styles/main.css`

Основные стили сохраняются. Важное дополнение для темизации карточек:

```css
task-card {
  --task-border: #e2e8f0;
  --task-radius: 12px;
  --task-bg: #ffffff;
  --task-padding: 1rem;
  --task-shadow: 0 1px 2px rgb(15 23 42 / 0.04);
}
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 19. HTML Templates и механизм переиспользования**

### Что реализовано

- Добавлен переиспользуемый `<template id="task-card-template">`
- Создание новой задачи больше не требует копипасты разметки
- Карточка собирается через `template.content.cloneNode(true)`
- Слоты используются как стабильный контракт компонента
- После создания задачи карточка сразу появляется в списке активных

### Структура проекта

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── components/
│   └── task-card.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

### Как проверить

1. Откройте диалог «Создать задачу».
2. Заполните форму и нажмите «Создать».
3. Новая карточка должна появиться в списке активных задач.
4. У новой карточки должны работать меню действий и приоритет.

## Следующие шаги

- Глава 20. Web Components в реальном проекте
- Композиция компонентов, границы ответственности и путь к мини-дизайн-системе FocusBoard
```

---

### Важные выводы главы

`<template>` делает переиспользование предсказуемым:  
мы описываем структуру один раз и клонируем её когда нужно.

`<slot>` делает компонент расширяемым:  
снаружи передаётся смысл, внутри сохраняется архитектура.

Это ключевой шаг от «просто custom element» к настоящей компонентной системе.

В следующей главе соберём всё вместе и посмотрим, как Web Components работают в реальном проекте и мини-дизайн-системе FocusBoard.