**Глава 24. Progressive Enhancement на практике**

### Главная идея главы

**Progressive Enhancement** — это стратегия, при которой интерфейс строится слоями:

1. **Базовый слой** — HTML-смысл и базовая функциональность для всех  
2. **Слой представления** — CSS для удобства и ясности  
3. **Слой поведения** — JavaScript как улучшение, а не как единственный способ существования UI

В 2026 году это снова практический стандарт: так живут SSR, гибридные приложения, устойчивые интерфейсы и доступные продукты.

### Три слоя FocusBoard

| Слой | Технологии | Что должно работать |
|------|------------|---------------------|
| Base | HTML | Структура, задачи, навигация по якорям, формы как документ |
| Presentation | CSS / tokens | Читаемый layout, визуальная иерархия, design system |
| Enhancement | JS / Web Components behavior | Dialog, popover logic, View Transitions, Navigation API, динамическое создание задач |

### Правило главы

Если функцию можно сделать полезной уже на HTML-уровне — сначала делаем её там.  
JavaScript подключаем только чтобы усилить UX, а не чтобы «включить» сам смысл интерфейса.

### Практическая задача главы

1. Явно разделить base / enhancement в документации и коде.  
2. Проверить сценарии FocusBoard с отключённым JavaScript.  
3. Улучшить базовую навигацию и понятность без JS.  
4. Сделать enhancement-слой безопасным через проверки поддержки API.  
5. Зафиксировать чек-лист Progressive Enhancement для дальнейших фич.

### Структура проекта после Главы 24

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

#### `docs/progressive-enhancement.md`

```markdown
# Progressive Enhancement в FocusBoard

## Базовый слой (без JavaScript)

Должен работать:
- просмотр списка задач
- понимание приоритета и статуса
- переход по ссылкам навигации (`#tasks`, `#notes`, `#settings`)
- просмотр структуры страницы скринридером
- видимость семантики документа
- открытие/закрытие `<details>` там, где они есть

## Слой представления

Должен работать даже при медленном JS:
- читаемая типографика
- визуальная иерархия
- токены дизайн-системы
- базовая адаптивность

## Слой улучшений (JavaScript)

Подключается только поверх base-слоя:
- `dialog` для создания задачи
- Popover-меню действий
- View Transitions
- Navigation API / history enhancement
- динамическое добавление task-card через `<template>`
- клиентская фильтрация

## Правила добавления новых фич

1. Можно ли выразить смысл через HTML?
2. Можно ли дать базовый UX без JS?
3. Что именно улучшает JavaScript?
4. Есть ли fallback, если API недоступен?
5. Не ломает ли enhancement базовый сценарий?
```

#### `index.html` (ключевые изменения акцента)

Добавляем более явные base-friendly якоря и noscript-подсказку.

```html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="FocusBoard 2026 — современная персональная панель продуктивности, построенная на возможностях современного HTML">
  <meta name="theme-color" content="#0f172a">
  <title>FocusBoard 2026 — Задачи</title>

  <!--
    Progressive Enhancement:
    1. HTML несёт смысл и базовые сценарии
    2. CSS даёт представление
    3. JS усиливает взаимодействие
  -->
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
      JavaScript отключён. Базовые возможности FocusBoard доступны:
      просмотр задач и переход между разделами через ссылки навигации.
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
    <!-- tasks / notes / settings sections остаются как server HTML -->
    <!-- ... текущие секции проекта ... -->
  </main>

  <!-- dialog, templates и scripts считаются enhancement-слоем -->
  <script src="components/task-card.js" type="module"></script>
  <script src="components/filter-panel.js" type="module"></script>
  <script src="scripts/main.js" defer fetchpriority="low"></script>
</body>
</html>
```

> Полные секции tasks/notes/settings, карточки, sidebar и dialog сохраняются из предыдущих глав. В этой главе мы не переписываем их смысл, а фиксируем их роль как base HTML.

#### `scripts/main.js` (enhancement-слой с явными проверками)

```js
// ===== Enhancement utilities =====
const hasViewTransitions = typeof document.startViewTransition === 'function';
const hasNavigationAPI = typeof window.navigation !== 'undefined';

// ===== Навигация =====
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
  // Enhancement: плавный переход, если доступен
  if (hasViewTransitions) {
    document.startViewTransition(() => setActiveSection(sectionId));
  } else {
    // Base-compatible fallback
    setActiveSection(sectionId);
  }
}

function navigateToSection(sectionId) {
  const nextHash = `#${sectionId}`;

  // Enhancement: Navigation API
  if (hasNavigationAPI) {
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

  // Fallback: History API
  history.pushState({ sectionId }, '', nextHash);
  renderSection(sectionId);
}

if (hasNavigationAPI) {
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

// Enhancement поверх base-якорной навигации
sectionLinks.forEach((link) => {
  link.addEventListener('click', (event) => {
    // Если JS есть — улучшаем UX
    event.preventDefault();
    navigateToSection(link.dataset.section);
  });
});

// Инициализация enhancement-слоя
renderSection(getSectionFromUrl());

// ===== Dialog / templates / popover / filters =====
// (логика создания задач и меню сохраняется из предыдущих глав)
// Важно: без JS пользователь всё равно видит список задач и базовую структуру.
```

#### `docs/ssr-architecture.md`

Остаётся актуальным и дополняет эту главу: SSR и Progressive Enhancement работают как одна стратегия.

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 24. Progressive Enhancement на практике**

### Что реализовано

- Явно разделены base / presentation / enhancement слои
- Добавлен документ `docs/progressive-enhancement.md`
- Навигация сохраняет базовый HTML-сценарий через якорные ссылки
- JS-улучшения подключаются только при поддержке соответствующих API
- Добавлен `noscript`-fallback с понятным сообщением
- Зафиксирован чек-лист для новых возможностей

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

### Как проверить

1. Откройте страницу с включённым JS — доступны dialog, transitions, dynamic tasks.
2. Отключите JavaScript.
3. Убедитесь, что:
   - задачи читаются;
   - навигационные ссылки ведут к разделам;
   - структура страницы остаётся понятной;
   - появляется `noscript`-сообщение.

## Следующие шаги

**Часть 8. Итоговый проект и отточка мастерства**

- Глава 25. Сборка полного приложения FocusBoard 2026
- Финальная целостная сборка всех практик
```

#### `styles/*`, `components/*`

Без ломки API.  
Их роль в Progressive Enhancement уже правильная:

- tokens/styles дают presentation-слой;
- components усиливают переиспользование, но не должны быть единственным носителем смысла.

---

### Практический чек-лист Progressive Enhancement

Перед добавлением любой новой возможности спрашиваем:

1. **Работает ли базовый сценарий без JS?**  
2. **Понятен ли интерфейс скринридеру уже на HTML?**  
3. **Улучшает ли JS UX или только компенсирует плохую разметку?**  
4. **Есть ли fallback для старых/ограниченных сред?**  
5. **Не станет ли enhancement обязательной зависимостью?**

### Примеры из FocusBoard

- Список задач — **base**  
- Карточка как Web Component — **enhancement + reuse**  
- Переход по `#notes` — **base**  
- View Transition при переходе — **enhancement**  
- Форма как HTML — **base**  
- `dialog.showModal()` — **enhancement**  
- Popover-меню — **enhancement**  
- `<details>` фильтров — **base behavior**

---

### Краткий итог главы

Progressive Enhancement — это не про «сайт без JS».  
Это про правильный порядок ответственности:

- HTML даёт смысл и базовый доступ;
- CSS даёт ясность;
- JavaScript даёт удобство и силу.

FocusBoard теперь официально развивается по этой модели.

В следующей главе соберём всё в финальную целостную версию приложения и подведём практический итог курса.