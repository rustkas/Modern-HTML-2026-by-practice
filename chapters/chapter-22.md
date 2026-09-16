**Глава 22. HTML и Design Systems**

### Главная идея главы

Дизайн-система — это не только макеты в Figma и не только CSS-классы. В современной веб-разработке **HTML становится фундаментом дизайн-системы**:

- задаёт смысловые роли интерфейса;
- фиксирует публичные контракты компонентов;
- позволяет темизировать UI через токены;
- делает блоки переиспользуемыми между экранами и проектами.

В этой главе мы оформляем FocusBoard как мини-дизайн-систему на базе современного HTML и Web Components.

### Что входит в HTML-ориентированную дизайн-систему

1. **Foundations (токены)** — цвет, типографика, отступы, радиусы, тени  
2. **Components** — `task-card`, `filter-panel` и другие UI-примитивы  
3. **Contracts** — attributes, slots, parts, events  
4. **Composition rules** — как компоненты собираются в экраны  
5. **Accessibility defaults** — роли, подписи, клавиатурный доступ  
6. **Documentation** — правила использования системы

### Практическая задача главы

1. Вынести визуальные токены в отдельный файл.  
2. Согласовать темизацию компонентов через CSS-переменные.  
3. Описать foundations и правила композиции.  
4. Зафиксировать HTML-компоненты как source of truth для UI-примитивов FocusBoard.  
5. Обновить документацию дизайн-системы.

### Структура проекта после Главы 22

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
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

---

### Содержание файлов на данном этапе

#### `styles/tokens.css`

```css
:root {
  /* ===== Color ===== */
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-text: #0f172a;
  --color-text-muted: #475569;
  --color-border: #e2e8f0;
  --color-primary: #2563eb;
  --color-danger: #dc2626;
  --color-warning: #d97706;
  --color-success: #16a34a;

  /* ===== Typography ===== */
  --font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-size-sm: 0.875rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.125rem;
  --line-height: 1.5;

  /* ===== Radius ===== */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;

  /* ===== Space ===== */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.5rem;

  /* ===== Shadow ===== */
  --shadow-sm: 0 1px 2px rgb(15 23 42 / 0.04);
  --shadow-md: 0 8px 20px rgb(15 23 42 / 0.08);

  /* ===== Component tokens: task-card ===== */
  --task-border: var(--color-border);
  --task-radius: var(--radius-md);
  --task-bg: var(--color-surface);
  --task-padding: var(--space-4);
  --task-shadow: var(--shadow-sm);
  --task-text: var(--color-text);

  /* ===== Component tokens: filter-panel ===== */
  --panel-border: var(--color-border);
  --panel-radius: var(--radius-md);
  --panel-bg: var(--color-surface);
}
```

#### `styles/main.css`

```css
@import url("./tokens.css");

body {
  font-family: var(--font-sans);
  line-height: var(--line-height);
  margin: 0;
  padding: var(--space-4);
  color: var(--color-text);
  background-color: var(--color-bg);
}

header, nav, main, aside, footer, search {
  margin-bottom: var(--space-5);
}

nav ul {
  display: flex;
  gap: var(--space-4);
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
  border-radius: var(--radius-sm);
}

.task-media {
  margin: var(--space-3) 0;
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
  color: var(--color-danger);
}

/* ===== View Transitions ===== */
@supports (view-transition-name: none) {
  :root {
    view-transition-name: root;
  }

  #app-main {
    view-transition-name: app-main;
  }

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

/* ===== Dialog ===== */
dialog {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  padding: var(--space-5);
  max-width: 480px;
  width: 90%;
  box-shadow: var(--shadow-md);
}

dialog::backdrop {
  background-color: rgb(15 23 42 / 0.5);
}

dialog fieldset {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  margin: 0 0 var(--space-4) 0;
  padding: var(--space-4);
}

dialog menu {
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
  padding: 0;
  margin-top: var(--space-4);
}

dialog label {
  display: block;
  font-weight: 600;
  margin-bottom: var(--space-1);
}

dialog input[type="text"],
dialog textarea {
  width: 100%;
  font: inherit;
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-sm);
  box-sizing: border-box;
}

dialog input:user-invalid,
dialog textarea:user-invalid,
dialog input[aria-invalid="true"] {
  border-color: var(--color-danger);
}

dialog .error-message {
  display: block;
  color: var(--color-danger);
  font-size: var(--font-size-sm);
  margin-top: var(--space-1);
  min-height: 1.25rem;
}

dialog small {
  display: block;
  color: var(--color-text-muted);
  font-size: var(--font-size-sm);
  margin-top: var(--space-1);
}

.form-error-summary {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: var(--space-3) var(--space-4);
  border-radius: var(--radius-sm);
  margin-bottom: var(--space-4);
}

/* ===== Popover ===== */
.task-menu {
  border: 1px solid var(--color-border);
  border-radius: 10px;
  padding: 0.35rem;
  background: var(--color-surface);
  box-shadow: var(--shadow-md);
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
  padding: var(--space-2) var(--space-3);
  border: none;
  background: transparent;
  border-radius: 6px;
}

.task-menu button:hover,
.task-menu button:focus-visible {
  background-color: #f1f5f9;
  outline: none;
}

/* ===== Quick actions ===== */
details.quick-actions {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  background: var(--color-surface);
  margin-bottom: var(--space-4);
  overflow: hidden;
}

details.quick-actions summary {
  padding: var(--space-3) var(--space-4);
  font-weight: 600;
  cursor: pointer;
  list-style: none;
}

details.quick-actions summary::-webkit-details-marker {
  display: none;
}

details.quick-actions ul {
  list-style: none;
  margin: 0;
  padding: 0 var(--space-4) var(--space-4);
}
```

#### `components/README.md`

```markdown
# FocusBoard Design System (HTML Components)

Мини-дизайн-система FocusBoard построена на современном HTML и Web Components.

## Foundations

### Токены
Базовые значения вынесены в `styles/tokens.css`:
- цвет
- типографика
- отступы
- радиусы
- тени
- component-level tokens

### Принципы
1. HTML-компонент — основной UI-примитив
2. Визуальный язык задаётся токенами
3. Компоненты темизируются снаружи через CSS-переменные
4. Смысл и доступность важнее декоративных классов
5. Композиция важнее сложных монолитных блоков

## Components

### `task-card`
Карточка задачи.

**Когда использовать**
- список задач
- результаты поиска
- коллекции однотипных action items

**Variants**
- `data-priority="high|medium|low"`
- `data-status="active|completed"`

**Slots**
- `title`
- `media`
- `description`
- `priority-label`
- `actions`

### `filter-panel`
Панель-оболочка для фильтров и вторичных настроек.

**Когда использовать**
- сайдбар фильтров
- группы настроек
- сворачиваемые панели управления

**Slots**
- `title`
- default slot — содержимое панели

## Composition

### Список задач
```html
<section>
  <task-card data-priority="high" data-status="active">...</task-card>
  <task-card data-priority="medium" data-status="active">...</task-card>
</section>
```

### Сайдбар
```html
<aside>
  <filter-panel>
    <span slot="title">Фильтры</span>
    <form>...</form>
  </filter-panel>
</aside>
```
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 22. HTML и Design Systems**

### Что реализовано

- Foundations дизайн-системы вынесены в `styles/tokens.css`
- Компоненты используют общие токены
- Зафиксирован HTML-first подход к UI-системе
- Обновлена документация компонентов как части design system
- Согласованы правила темизации и композиции

### Структура проекта

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
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

### Слои дизайн-системы FocusBoard

1. **Tokens** — `styles/tokens.css`
2. **Components** — `components/*`
3. **Composition** — секции и страницы в `index.html`
4. **App logic** — `scripts/main.js`

## Следующие шаги

- Глава 23. HTML в эпоху SSR
- Как семантический HTML, declarative components и design tokens помогают серверному рендерингу и гибридным архитектурам
```

#### `index.html`, `components/*.js`, `scripts/main.js`

Функциональное поведение остаётся прежним.  
Главный результат главы — системность:

- визуальные решения централизованы в токенах;
- компоненты опираются на единый язык стилей;
- документация описывает не только код, но и правила использования.

---

### Важные выводы главы

Хорошая дизайн-система на современном HTML строится так:

- **токены** задают визуальный язык;
- **компоненты** задают UI-примитивы;
- **slots/attributes** задают контракт;
- **страницы** задают пользовательские сценарии;
- **документация** делает систему пригодной для команды.

FocusBoard больше не просто учебное приложение — это маленькая, но цельная HTML-дизайн-система.

В следующей главе разберём, почему такой подход особенно хорошо работает в эпоху **SSR и гибридных архитектур**.