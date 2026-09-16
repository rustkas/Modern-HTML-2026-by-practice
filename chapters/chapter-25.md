**Глава 25. Сборка полного приложения FocusBoard 2026**

### Главная идея главы

До этой главы мы наращивали FocusBoard итеративно: семантика, нативные интерактивные элементы, формы, производительность, браузерные API, Web Components, дизайн-система, SSR-мышление и Progressive Enhancement.

Теперь задача другая — **собрать всё в целостное приложение** и зафиксировать финальную архитектуру.

Это не глава про новую фичу. Это глава про синтез.

### Что должно быть в финальной сборке

1. Семантический HTML-каркас и landmarks  
2. Нативные dialog / popover / details  
3. Современные доступные формы  
4. Performance-атрибуты и критический CSS  
5. View Transitions + Navigation API (с fallback)  
6. Web Components (`task-card`, `filter-panel`)  
7. Templates и динамическое создание задач  
8. Design tokens  
9. SSR-friendly base layer  
10. Progressive Enhancement

### Практическая задача главы

1. Зафиксировать финальную структуру проекта.  
2. Свести все практики в один согласованный код.  
3. Проверить, что приложение работает как цельный продукт.  
4. Описать финальные сценарии пользователя.  
5. Подготовить основу для аудита в следующей главе.

### Структура проекта после Главы 25

```text
focusboard-2026/
├── index.html
├── styles/
│   ├── main.css
│   └── tokens.css
├── scripts/
│   └── main.js
├── components/
│   ├── task-card.js
│   ├── filter-panel.js
│   └── README.md
├── docs/
│   ├── ssr-architecture.md
│   └── progressive-enhancement.md
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

---

### Содержание файлов на данном этапе

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности, собранная как практический итог руководства  
**«Современный HTML 2026 на практике»**.

## Финальный этап

**Глава 25. Сборка полного приложения FocusBoard 2026**

### Что умеет приложение

- Просмотр задач по разделам
- Создание задачи через нативный `<dialog>`
- Контекстные действия через Popover API
- Компонентные карточки задач (`task-card`)
- Панель фильтров (`filter-panel`)
- Плавные переходы между разделами
- Современная маршрутизация (Navigation API + fallback)
- Адаптивные изображения
- Design tokens и мини-дизайн-система
- SSR-friendly HTML и Progressive Enhancement

### Структура

```text
focusboard-2026/
├── index.html
├── styles/
│   ├── main.css
│   └── tokens.css
├── scripts/
│   └── main.js
├── components/
│   ├── task-card.js
│   ├── filter-panel.js
│   └── README.md
├── docs/
│   ├── ssr-architecture.md
│   └── progressive-enhancement.md
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

### Как открыть

Откройте `index.html` в современном браузере.

### Основной пользовательский сценарий

1. Пользователь видит список задач сразу (base HTML).
2. Может перейти в «Заметки» или «Настройки».
3. Может создать задачу через диалог.
4. Может открыть действия задачи через popover.
5. При наличии поддержки браузера получает плавные переходы и улучшенную навигацию.

## Следующие шаги

- Глава 26. Аудит и оптимизация
- Проверка a11y, performance, семантики и архитектурной целостности
```

#### `styles/tokens.css`

```css
:root {
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-text: #0f172a;
  --color-text-muted: #475569;
  --color-border: #e2e8f0;
  --color-primary: #2563eb;
  --color-danger: #dc2626;
  --color-warning: #d97706;
  --color-success: #16a34a;

  --font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-size-sm: 0.875rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.125rem;
  --line-height: 1.5;

  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;

  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.5rem;

  --shadow-sm: 0 1px 2px rgb(15 23 42 / 0.04);
  --shadow-md: 0 8px 20px rgb(15 23 42 / 0.08);

  --task-border: var(--color-border);
  --task-radius: var(--radius-md);
  --task-bg: var(--color-surface);
  --task-padding: var(--space-4);
  --task-shadow: var(--shadow-sm);
  --task-text: var(--color-text);

  --panel-border: var(--color-border);
  --panel-radius: var(--radius-md);
  --panel-bg: var(--color-surface);
}
```

#### `styles/main.css`

Финальные стили приложения на токенах: layout, dialog, popover, quick actions, view transitions.  
Используется как presentation-слой поверх семантического HTML.

#### `components/task-card.js`

Финальный UI-примитив карточки задачи:

- Shadow DOM
- slots: `title`, `media`, `description`, `priority-label`, `actions`
- attributes: `data-task-id`, `data-priority`, `data-status`
- темизация через CSS-переменные

#### `components/filter-panel.js`

Финальный UI-примитив панели фильтров:

- Shadow DOM
- slot `title` + default slot
- role/region для доступности

#### `components/README.md`

Документация мини-дизайн-системы и контрактов компонентов.

#### `docs/ssr-architecture.md`

Описание HTML-first / SSR-friendly модели.

#### `docs/progressive-enhancement.md`

Описание base / presentation / enhancement слоёв.

#### `scripts/main.js`

Финальная прикладная логика:

- навигация по разделам
- Navigation API + History fallback
- View Transitions (если поддерживаются)
- создание задачи через `<dialog>` + `<template>`
- обработка popover-действий
- логирование фильтров

#### `index.html` (финальная сборка)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="FocusBoard 2026 — современная персональная панель продуктивности на современном HTML">
  <meta name="theme-color" content="#0f172a">
  <title>FocusBoard 2026 — Задачи</title>

  <link rel="preload" href="styles/main.css" as="style" fetchpriority="high">
  <link rel="stylesheet" href="styles/main.css">
</head>
<body>
  <header>
    <h1>FocusBoard</h1>
    <p>Персональная панель продуктивности</p>
  </header>

  <noscript>
    <p>
      JavaScript отключён. Доступны базовые возможности:
      просмотр задач и переход между разделами через навигацию.
    </p>
  </noscript>

  <nav aria-label="Основная навигация">
    <ul>
      <li><a href="#tasks" data-section="tasks" aria-current="page">Задачи</a></li>
      <li><a href="#notes" data-section="notes">Заметки</a></li>
      <li><a href="#settings" data-section="settings">Настройки</a></li>
    </ul>
  </nav>

  <search>
    <form role="search" aria-label="Поиск по задачам и заметкам" action="#tasks" method="get">
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
        <p><button type="button" id="create-task-btn">Создать задачу</button></p>
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
      <p>Раздел готов к дальнейшему развитию.</p>
    </section>

    <section id="settings" class="app-section" data-section-panel="settings" hidden>
      <header>
        <h2>Настройки</h2>
        <p>Параметры приложения и профиля</p>
      </header>
      <p>Раздел готов к дальнейшему развитию.</p>
    </section>
  </main>

  <aside aria-labelledby="sidebar-heading">
    <h2 id="sidebar-heading">Фильтры и быстрые действия</h2>

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
    <p>FocusBoard 2026 · Современный HTML как основа приложения</p>
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

---

### Финальная архитектурная схема

```text
FocusBoard 2026
├── Base HTML
│   ├── semantics / landmarks
│   ├── tasks content
│   └── anchor navigation
├── Presentation
│   ├── tokens.css
│   └── main.css
├── Components
│   ├── task-card
│   └── filter-panel
└── Enhancement
    ├── dialog / popover
    ├── view transitions
    ├── navigation API
    └── dynamic templates
```

### Что считается результатом главы

После Главы 25 у читателя есть не набор упражнений, а:

- законченное учебное приложение;
- согласованная структура проекта;
- переиспользуемые компоненты;
- документированные архитектурные решения;
- база для аудита качества.

---

### Краткий итог главы

Мы собрали FocusBoard 2026 как целостный продукт на современном HTML.  
Все ключевые темы книги теперь существуют не как отдельные демо, а как части одной системы.

В следующей главе проведём **аудит и оптимизацию**: доступность, производительность, семантика и архитектурная чистота.