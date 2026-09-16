**Глава 18. Shadow DOM и инкапсуляция**

### Главная идея главы

**Shadow DOM** создаёт границу инкапсуляции между внутренним устройством компонента и остальной страницей. Благодаря этому:

- стили компонента не «протекают» наружу;
- внешние стили не ломают внутреннюю структуру без явного разрешения;
- разметка компонента становится предсказуемой;
- API компонента можно проектировать через attributes, properties, events и slots.

В этой главе мы углубим работу с Shadow DOM в `task-card`:

- явно разделим внутренние и внешние стили;
- усилим роль `:host` и `::slotted()`;
- покажем, какие части должны оставаться снаружи через slots;
- подготовим компонент к более строгой инкапсуляции.

### Что важно понимать про Shadow DOM

1. **`:host`** — стиль самого компонента.
2. **`::slotted()`** — стиль проецируемого внешнего контента (ограниченно).
3. **Slots** — публичные «точки расширения» компонента.
4. **Open vs Closed**
   - `open` — `element.shadowRoot` доступен
   - `closed` — снаружи shadow root напрямую не получить
5. События из shadow-дерева при всплытии **retarget** на host-элемент.

### Практическая задача главы

1. Улучшить инкапсуляцию стилей `task-card`.
2. Сделать внешний API компонента понятнее через slots.
3. Добавить внутренние CSS-переменные для темизации.
4. Показать безопасную работу с shadow root из Custom Element.
5. Подготовить компонент к переиспользованию в дизайн-системе.

### Структура проекта после Главы 18

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

Структура файлов не изменилась, но компонент стал заметно зрелее.

---

### Содержание файлов на данном этапе

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

    /* Стили для проецируемого контента */
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

    // Если Declarative Shadow DOM уже есть — используем его.
    // Если нет — создаём shadow root императивно (fallback).
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
    const status = this.getAttribute('data-status') || 'active';
    const taskId = this.getAttribute('data-task-id');

    // Повышаем доступность host-элемента как карточки
    if (!this.hasAttribute('role')) {
      this.setAttribute('role', 'article');
    }

    this.setAttribute('data-state', status);

    if (taskId) {
      this.setAttribute('data-task-id', taskId);
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

        <task-card data-task-id="1" data-priority="high" data-status="active">
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

  <script src="components/task-card.js" type="module"></script>
  <script src="scripts/main.js" defer fetchpriority="low"></script>
</body>
</html>
```

#### `styles/main.css` (ключевые дополнения)

```css
/* Темизация task-card через CSS-переменные */
task-card {
  --task-border: #e2e8f0;
  --task-radius: 12px;
  --task-bg: #ffffff;
  --task-padding: 1rem;
  --task-shadow: 0 1px 2px rgb(15 23 42 / 0.04);
}

task-card[data-priority="high"] {
  --task-shadow: 0 0 0 1px rgb(239 68 68 / 0.08);
}
```

Остальные стили страницы сохраняются из предыдущей главы.

#### `scripts/main.js`

Без изменений по сравнению с Главой 17 (навигация, dialog, popover, фильтры).

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 18. Shadow DOM и инкапсуляция**

### Что реализовано

- Усилена инкапсуляция `task-card` через Shadow DOM
- Добавлен шаблон компонента с `:host`, `::slotted()`, `part`
- Поддержан fallback: если Declarative Shadow DOM отсутствует, shadow root создаётся императивно
- Введены CSS-переменные для внешней темизации компонента
- Улучшен доступный host API карточки
- Разметка снаружи стала чище: в `index.html` остались только slots

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

1. Откройте DevTools и исследуйте shadow root у `task-card`.
2. Измените CSS-переменные у `task-card` в `main.css`.
3. Убедитесь, что внутренние стили карточки не ломаются внешними правилами.
4. Проверьте, что слоты `title`, `description`, `actions` корректно проецируются.

## Следующие шаги

- Глава 19. HTML Templates и механизм переиспользования
- Вынос шаблонов, слоты как контракт компонента, подготовка к библиотеке UI
```

---

### Важные выводы главы

Shadow DOM — это не «просто спрятать HTML». Это инструмент проектирования границ компонента:

- что является внутренним устройством;
- что является публичным API;
- как компонент темизируется снаружи;
- как он остаётся устойчивым к изменениям страницы.

В следующей главе сосредоточимся на **`<template>` и слотах** как на системе переиспользования и контрактов между компонентом и внешним кодом.