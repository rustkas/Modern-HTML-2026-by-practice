**Глава 20. Web Components в реальном проекте**

### Главная идея главы

Отдельный Custom Element — это ещё не архитектура.  
В реальном проекте Web Components начинают работать тогда, когда появляются:

- понятные границы ответственности;
- композиция компонентов;
- стабильные публичные API;
- повторное использование без копипасты;
- согласованные правила темизации и доступности.

В этой главе мы превращаем FocusBoard из «страницы с одним компонентом» в небольшую компонентную систему.

### Что считаем «реальным» использованием Web Components

1. Компонент решает одну задачу.
2. У компонента есть явный контракт: attributes, slots, events.
3. Компоненты можно вкладывать друг в друга.
4. Внешний код не лезет во внутренности shadow-дерева без необходимости.
5. Новые экраны собираются из готовых блоков, а не пишутся с нуля.

### Практическая задача главы

1. Добавить второй компонент — `filter-panel`.
2. Показать композицию: страница собирается из `task-card` + `filter-panel`.
3. Унифицировать подход к атрибутам и событиям.
4. Вынести общие правила компонентов.
5. Зафиксировать мини-дизайн-систему FocusBoard на уровне HTML-компонентов.

### Структура проекта после Главы 20

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── components/
│   ├── task-card.js
│   └── filter-panel.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

---

### Содержание файлов на данном этапе

#### `components/filter-panel.js`

```js
const filterPanelTemplate = document.createElement('template');
filterPanelTemplate.innerHTML = `
  <style>
    :host {
      display: block;
      border: 1px solid var(--panel-border, #e2e8f0);
      border-radius: var(--panel-radius, 12px);
      background: var(--panel-bg, #fff);
      overflow: hidden;
    }

    details {
      border: none;
      margin: 0;
    }

    summary {
      padding: 0.75rem 1rem;
      font-weight: 600;
      cursor: pointer;
      list-style: none;
    }

    summary::-webkit-details-marker {
      display: none;
    }

    summary::before {
      content: "▸ ";
      display: inline-block;
      transition: transform 0.15s ease;
    }

    details[open] summary::before {
      transform: rotate(90deg);
    }

    .content {
      padding: 0 1rem 1rem 1rem;
    }

    fieldset {
      border: none;
      margin: 0 0 1rem 0;
      padding: 0;
    }

    legend {
      font-weight: 600;
      margin-bottom: 0.5rem;
    }

    label {
      display: block;
      margin-bottom: 0.35rem;
    }
  </style>

  <details open>
    <summary>
      <slot name="title">Фильтры</slot>
    </summary>
    <div class="content">
      <slot></slot>
    </div>
  </details>
`;

class FilterPanel extends HTMLElement {
  constructor() {
    super();
    if (!this.shadowRoot) {
      this.attachShadow({ mode: 'open' });
      this.shadowRoot.appendChild(filterPanelTemplate.content.cloneNode(true));
    }
  }

  connectedCallback() {
    if (!this.hasAttribute('role')) {
      this.setAttribute('role', 'region');
    }
    if (!this.hasAttribute('aria-label')) {
      this.setAttribute('aria-label', 'Панель фильтров');
    }
  }
}

if (!customElements.get('filter-panel')) {
  customElements.define('filter-panel', FilterPanel);
}

export { FilterPanel };
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
    if (!this.hasAttribute('role')) {
      this.setAttribute('role', 'article');
    }
  }

  attributeChangedCallback(name) {
    if (name === 'data-priority') {
      this.#syncPriorityState();
    }
  }

  #syncPriorityState() {
    const priority = this.getAttribute('data-priority') || 'medium';
    const priorityEl = this.shadowRoot?.querySelector('[data-priority-label]');
    if (priorityEl) {
      priorityEl.setAttribute('data-priority', priority);
    }
  }
}

if (!customElements.get('task-card')) {
  customElements.define('task-card', TaskCard);
}

export { TaskCard };
```

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

    <!-- Композиция: filter-panel как самостоятельный компонент -->
    <filter-panel>
      <span slot="title">Фильтры задач</span>
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
    </filter-panel>

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
  <script src="components/filter-panel.js" type="module"></script>
  <script src="scripts/main.js" defer fetchpriority="low"></script>
</body>
</html>
```

#### `scripts/main.js`

Логика навигации, создания задач через `<template>`, диалога и popover сохраняется из Главы 19.  
Существенное изменение архитектурное: страница теперь собирается из нескольких независимых компонентов.

#### `styles/main.css` (дополнения для мини-дизайн-системы)

```css
/* Общие токены компонентов */
:root {
  --panel-border: #e2e8f0;
  --panel-radius: 12px;
  --panel-bg: #ffffff;

  --task-border: #e2e8f0;
  --task-radius: 12px;
  --task-bg: #ffffff;
  --task-padding: 1rem;
  --task-shadow: 0 1px 2px rgb(15 23 42 / 0.04);
}

task-card,
filter-panel {
  font-family: inherit;
}
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 20. Web Components в реальном проекте**

### Что реализовано

- Добавлен второй компонент `filter-panel`
- Страница собирается через композицию компонентов
- Зафиксированы границы ответственности:
  - `task-card` — отображение задачи
  - `filter-panel` — оболочка фильтров
  - `main.js` — прикладная логика приложения
- Заложены токены мини-дизайн-системы через CSS-переменные
- Показан практический путь от одиночного компонента к системе компонентов

### Структура проекта

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
├── components/
│   ├── task-card.js
│   └── filter-panel.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

### Архитектурные правила FocusBoard

1. Компонент = одна зона ответственности
2. Публичный API = attributes + slots (+ events при необходимости)
3. Внутренности shadow DOM не используются снаружи напрямую
4. Темизация — через CSS-переменные
5. Прикладная логика живёт в `scripts/main.js`, а не в каждом компоненте без необходимости

## Итог Части 6

К этому моменту у FocusBoard есть:
- Custom Elements
- Shadow DOM
- Templates и slots
- Композиция компонентов
- Зачатки собственной UI-системы на нативном HTML

## Следующие шаги

**Часть 7. Архитектура и масштабирование**

- Глава 21. Компонентное мышление
- Правила проектирования интерфейсов на HTML-компонентах
```

---

### Важные выводы главы

Web Components становятся полезными не тогда, когда вы создали один красивый элемент, а когда:

- компоненты начинают собираться вместе;
- у каждого есть понятная роль;
- страница читается как композиция блоков, а не как монолитная разметка.

FocusBoard теперь развивается как набор нативных компонентов, а не как «один HTML-файл с скриптами».

В следующей главе перейдём к более общему **компонентному мышлению** и правилам масштабирования HTML-архитектуры.