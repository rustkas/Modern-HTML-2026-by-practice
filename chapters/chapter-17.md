**Глава 17. Custom Elements — пользовательские элементы**

### Главная идея главы

**Custom Elements** позволяют создавать собственные HTML-теги с собственным поведением, жизненным циклом и API. Вместе с Shadow DOM и Templates они образуют основу Web Components.

В этой главе мы превратим `<task-card>` из «просто тега с Declarative Shadow DOM» в полноценный Custom Element:

- зарегистрируем элемент через `customElements.define`
- добавим жизненный цикл (`connectedCallback`)
- научим компонент читать attributes (`data-task-id`, `data-priority`, `data-status`)
- подготовим основу для переиспользования

### Базовый каркас Custom Element

```js
class TaskCard extends HTMLElement {
  connectedCallback() {
    // элемент добавлен в DOM
  }
}

customElements.define('task-card', TaskCard);
```

### Практическая задача главы

1. Вынести логику `task-card` в отдельный файл компонента.
2. Зарегистрировать Custom Element.
3. Сохранить Declarative Shadow DOM как основу разметки.
4. Добавить минимальное поведение компонента (например, отражение приоритета и статуса).
5. Подготовить структуру папки `components/`.

### Структура проекта после Главы 17

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

        <!-- Полноценный Custom Element + Declarative Shadow DOM -->
        <task-card data-task-id="1" data-priority="high" data-status="active">
          <template shadowrootmode="open">
            <style>
              :host {
                display: block;
                border: 1px solid #e2e8f0;
                border-radius: 12px;
                background: #fff;
                padding: 1rem;
                margin-bottom: 1rem;
              }

              :host([data-status="completed"]) {
                opacity: 0.75;
              }

              .meta {
                display: flex;
                justify-content: space-between;
                gap: 1rem;
                align-items: center;
                margin-top: 0.75rem;
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

              ::slotted(h4) {
                margin: 0 0 0.5rem 0;
              }

              ::slotted(p) {
                margin: 0.5rem 0;
              }
            </style>

            <article class="task">
              <header>
                <slot name="title"></slot>
              </header>

              <slot name="media"></slot>
              <slot name="description"></slot>

              <footer class="meta">
                <div class="priority" data-priority-label>
                  <slot name="priority-label">Приоритет не указан</slot>
                </div>
                <div>
                  <slot name="actions"></slot>
                </div>
              </footer>
            </article>
          </template>

          <h4 slot="title" id="task-1-title">Изучить современный HTML</h4>

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
          <template shadowrootmode="open">
            <style>
              :host {
                display: block;
                border: 1px solid #e2e8f0;
                border-radius: 12px;
                background: #fff;
                padding: 1rem;
                margin-bottom: 1rem;
              }

              :host([data-status="completed"]) {
                opacity: 0.75;
              }

              .meta {
                display: flex;
                justify-content: space-between;
                gap: 1rem;
                align-items: center;
                margin-top: 0.75rem;
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

              ::slotted(h4) {
                margin: 0 0 0.5rem 0;
              }

              ::slotted(p) {
                margin: 0.5rem 0;
              }
            </style>

            <article class="task">
              <header>
                <slot name="title"></slot>
              </header>
              <slot name="description"></slot>
              <footer class="meta">
                <div class="priority" data-priority-label>
                  <slot name="priority-label">Приоритет не указан</slot>
                </div>
                <div>
                  <slot name="actions"></slot>
                </div>
              </footer>
            </article>
          </template>

          <h4 slot="title" id="task-2-title">Собрать семантический каркас FocusBoard</h4>
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
      </section>

      <section aria-labelledby="completed-tasks-heading">
        <h3 id="completed-tasks-heading">Выполненные</h3>

        <task-card data-task-id="3" data-priority="low" data-status="completed">
          <template shadowrootmode="open">
            <style>
              :host {
                display: block;
                border: 1px solid #e2e8f0;
                border-radius: 12px;
                background: #fff;
                padding: 1rem;
                margin-bottom: 1rem;
              }

              :host([data-status="completed"]) {
                opacity: 0.75;
              }

              .meta {
                display: flex;
                justify-content: space-between;
                gap: 1rem;
                align-items: center;
                margin-top: 0.75rem;
              }

              .priority {
                font-size: 0.875rem;
                color: #64748b;
              }

              ::slotted(h4) {
                margin: 0 0 0.5rem 0;
              }

              ::slotted(p) {
                margin: 0.5rem 0;
              }
            </style>

            <article class="task">
              <header>
                <slot name="title"></slot>
              </header>
              <slot name="description"></slot>
              <footer class="meta">
                <div class="priority">
                  <slot name="priority-label">Выполнено</slot>
                </div>
              </footer>
            </article>
          </template>

          <h4 slot="title" id="task-3-title">Прочитать введение в Modern HTML 2026</h4>
          <p slot="description">Ознакомиться с основными идеями книги.</p>
          <span slot="priority-label">Выполнено</span>
        </task-card>
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

  <!-- Компоненты и основной скрипт -->
  <script src="components/task-card.js" type="module"></script>
  <script src="scripts/main.js" defer fetchpriority="low"></script>
</body>
</html>
```

#### `components/task-card.js`

```js
class TaskCard extends HTMLElement {
  static get observedAttributes() {
    return ['data-priority', 'data-status', 'data-task-id'];
  }

  connectedCallback() {
    this.#syncPriorityState();
    this.#syncStatusState();
  }

  attributeChangedCallback(name) {
    if (name === 'data-priority') {
      this.#syncPriorityState();
    }

    if (name === 'data-status') {
      this.#syncStatusState();
    }
  }

  #syncPriorityState() {
    const priority = this.getAttribute('data-priority') || 'medium';
    const priorityEl = this.shadowRoot?.querySelector('[data-priority-label]');

    if (priorityEl) {
      priorityEl.setAttribute('data-priority', priority);
    }
  }

  #syncStatusState() {
    // Статус уже отражается через :host([data-status="completed"])
    // Метод оставлен для будущего расширения поведения
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
    document.startViewTransition(() => {
      setActiveSection(sectionId);
    });
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

#### `styles/main.css`

(основной файл стилей остаётся прежним; стили карточки частично инкапсулированы в Shadow DOM)

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 17. Custom Elements — пользовательские элементы**

### Что реализовано

- Создан Custom Element `<task-card>`
- Компонент вынесен в `components/task-card.js`
- Используется `customElements.define`
- Добавлены `connectedCallback` и `attributeChangedCallback`
- Сохранён Declarative Shadow DOM
- Все задачи переведены на `<task-card>`

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

1. Откройте страницу.
2. В DevTools убедитесь, что `task-card` является custom element.
3. Измените `data-priority` или `data-status` у элемента и проверьте обновление состояния.
4. Проверьте, что shadow root сохраняется.

## Следующие шаги

- Глава 18. Shadow DOM и инкапсуляция
- Углубление в открытый/закрытый режим, слоты, стилизацию и границы компонента
```

---

### Важные замечания

- Custom Element не обязан сразу создавать Shadow DOM в JavaScript — Declarative Shadow DOM отлично дополняет его.
- `observedAttributes` + `attributeChangedCallback` дают реактивность к изменениям attributes.
- Вынос компонента в отдельный файл — первый шаг к настоящей библиотеке UI-элементов FocusBoard.

В следующей главе углубимся в **Shadow DOM и инкапсуляцию**: открытый/закрытый режим, стилизацию, слоты и границы ответственности компонента.