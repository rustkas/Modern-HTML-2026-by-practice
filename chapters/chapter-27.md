**Глава 27. Библиотека шаблонов и паттернов**

### Главная идея главы

Финальная цель практического руководства — не только собрать FocusBoard, но и вынести из него **переносимые шаблоны**.  
После этой главы у читателя должна остаться личная библиотека паттернов современного HTML, которую можно применять в новых проектах без привязки к одному дашборду.

Мы фиксируем:

1. HTML-паттерны  
2. Component-паттерны  
3. Form / a11y-паттерны  
4. Performance-паттерны  
5. Архитектурные чек-листы  

### Что входит в итоговую библиотеку

| Категория | Примеры |
|----------|---------|
| Semantics | landmarks, article/section, heading hierarchy |
| Interaction | dialog, popover, details |
| Forms | fieldset/legend, validation, error messaging |
| Components | custom element + shadow DOM + slots |
| Performance | preload, fetchpriority, lazy images |
| Architecture | progressive enhancement, SSR-first, design tokens |

### Практическая задача главы

1. Создать папку `patterns/` с переносимыми шаблонами.  
2. Зафиксировать короткие production-ready примеры.  
3. Собрать финальный чек-лист «Современный HTML 2026».  
4. Связать паттерны с реальными местами использования в FocusBoard.  
5. Закрыть практическое руководство как систему, а не как набор разрозненных глав.

### Структура проекта после Главы 27

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
│   ├── progressive-enhancement.md
│   ├── audit-checklist.md
│   └── modern-html-checklist.md
├── patterns/
│   ├── README.md
│   ├── dialog-form.html
│   ├── popover-actions.html
│   ├── details-filters.html
│   ├── responsive-image.html
│   └── custom-element-card.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

---

### Содержание файлов на данном этапе

#### `patterns/README.md`

```markdown
# Библиотека паттернов Modern HTML 2026

Эти шаблоны извлечены из FocusBoard и предназначены для переиспользования в других проектах.

## Список паттернов

1. `dialog-form.html` — модальная форма на `<dialog>`
2. `popover-actions.html` — меню действий на Popover API
3. `details-filters.html` — фильтры на `<details>` + `<fieldset>`
4. `responsive-image.html` — адаптивное изображение
5. `custom-element-card.js` — минимальный Web Component с slots

## Как использовать

- копируйте паттерн в новый проект;
- сохраняйте семантику и a11y-атрибуты;
- JS подключайте как enhancement, а не как обязательную основу.
```

#### `patterns/dialog-form.html`

```html
<dialog id="example-dialog" aria-labelledby="example-dialog-title">
  <form method="dialog" id="example-form" novalidate>
    <header>
      <h2 id="example-dialog-title">Новый элемент</h2>
    </header>

    <div id="example-error-summary" class="form-error-summary" role="alert" hidden></div>

    <fieldset>
      <legend>Основная информация</legend>
      <p>
        <label for="example-title">
          Название
          <span aria-hidden="true">*</span>
        </label>
        <input
          id="example-title"
          name="title"
          type="text"
          required
          minlength="3"
          maxlength="120"
          aria-describedby="example-title-hint example-title-error"
        >
        <small id="example-title-hint">Не менее 3 символов</small>
        <span id="example-title-error" role="alert"></span>
      </p>
    </fieldset>

    <menu>
      <button type="submit" value="cancel">Отмена</button>
      <button type="submit" value="confirm">Сохранить</button>
    </menu>
  </form>
</dialog>
```

#### `patterns/popover-actions.html`

```html
<button type="button" popovertarget="item-menu" popovertargetaction="toggle" aria-haspopup="menu">
  Действия
</button>

<div id="item-menu" popover="auto" role="menu" aria-label="Действия">
  <menu>
    <li role="none"><button type="button" role="menuitem" data-action="edit">Редактировать</button></li>
    <li role="none"><button type="button" role="menuitem" data-action="archive">В архив</button></li>
    <li role="none"><button type="button" role="menuitem" data-action="delete">Удалить</button></li>
  </menu>
</div>
```

#### `patterns/details-filters.html`

```html
<details open>
  <summary>Фильтры</summary>
  <form>
    <fieldset>
      <legend>Статус</legend>
      <label><input type="radio" name="status" value="all" checked> Все</label>
      <label><input type="radio" name="status" value="active"> Активные</label>
      <label><input type="radio" name="status" value="done"> Завершённые</label>
    </fieldset>

    <fieldset>
      <legend>Метки</legend>
      <label><input type="checkbox" name="tag" value="work"> Работа</label>
      <label><input type="checkbox" name="tag" value="personal"> Личное</label>
    </fieldset>
  </form>
</details>
```

#### `patterns/responsive-image.html`

```html
<picture>
  <source
    srcset="
      images/item-320.webp 320w,
      images/item-640.webp 640w,
      images/item-960.webp 960w
    "
    sizes="(max-width: 600px) 100vw, 320px"
    type="image/webp"
  >
  <img
    src="images/item-640.jpg"
    alt="Описание изображения"
    width="320"
    height="180"
    loading="lazy"
    decoding="async"
    fetchpriority="low"
  >
</picture>
```

#### `patterns/custom-element-card.js`

```js
const cardTemplate = document.createElement('template');
cardTemplate.innerHTML = `
  <style>
    :host {
      display: block;
      border: 1px solid var(--card-border, #e2e8f0);
      border-radius: var(--card-radius, 12px);
      background: var(--card-bg, #fff);
      padding: var(--card-padding, 1rem);
    }
    ::slotted([slot="title"]) { margin: 0 0 .5rem; }
    ::slotted([slot="body"]) { margin: 0; }
  </style>
  <article>
    <header><slot name="title"></slot></header>
    <div><slot name="body"></slot></div>
    <footer><slot name="actions"></slot></footer>
  </article>
`;

class UiCard extends HTMLElement {
  constructor() {
    super();
    if (!this.shadowRoot) {
      this.attachShadow({ mode: 'open' });
      this.shadowRoot.appendChild(cardTemplate.content.cloneNode(true));
    }
  }
}

if (!customElements.get('ui-card')) {
  customElements.define('ui-card', UiCard);
}
```

#### `docs/modern-html-checklist.md`

```markdown
# Чек-лист «Современный HTML 2026»

## Семантика
- [ ] Есть корректные landmarks
- [ ] Заголовки выстроены иерархически
- [ ] Смысловые блоки не заменены бессмысленными обёртками

## Интерактивные примитивы
- [ ] Dialog используется для модальных сценариев
- [ ] Popover используется для лёгких контекстных панелей
- [ ] Details используется для раскрываемых секций

## Формы и a11y
- [ ] У полей есть label
- [ ] Ошибки связаны с полями
- [ ] Обязательность не передаётся только цветом
- [ ] Клавиатурный сценарий завершён

## Производительность
- [ ] Критический CSS приоритезирован
- [ ] JS не блокирует first paint
- [ ] Media имеет размеры и lazy-стратегию
- [ ] Есть preconnect/preload там, где это оправдано

## Компоненты
- [ ] У компонента одна ответственность
- [ ] Публичный API явен (attributes/slots/events)
- [ ] Shadow DOM не скрывает смысл документа целиком

## Архитектура
- [ ] Base HTML полезен без JS
- [ ] Enhancement отделён от base-слоя
- [ ] Design tokens согласованы
- [ ] Документация позволяет переиспользовать систему
```

#### `README.md`

```markdown
# FocusBoard 2026

Практический итоговый проект руководства «Современный HTML 2026».

## Текущий этап

**Глава 27. Библиотека шаблонов и паттернов**

### Что добавлено

- Папка `patterns/` с переносимыми HTML/JS-шаблонами
- Финальный чек-лист `docs/modern-html-checklist.md`
- Связка между учебным приложением и библиотекой паттернов
- Завершён практический контур книги: от теории к системе шаблонов

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
│   ├── progressive-enhancement.md
│   ├── audit-checklist.md
│   └── modern-html-checklist.md
├── patterns/
│   ├── README.md
│   ├── dialog-form.html
│   ├── popover-actions.html
│   ├── details-filters.html
│   ├── responsive-image.html
│   └── custom-element-card.js
├── images/
│   ├── task-placeholder.svg
│   ├── task-320.svg
│   ├── task-640.svg
│   └── task-960.svg
└── README.md
```

## Как использовать результаты книги

1. Изучите FocusBoard как целостный пример.
2. Берите готовые паттерны из `patterns/`.
3. Проверяйте новые экраны по `docs/modern-html-checklist.md`.
4. Сохраняйте принцип: HTML → CSS → JS enhancement.

## Итог практического руководства

После проработки всех глав читатель получает:
- законченный проект FocusBoard 2026;
- навыки работы с современным HTML;
- библиотеку шаблонов;
- архитектурные чек-листы для следующих продуктов.
```

#### Основные рабочие файлы приложения

`index.html`, `styles/*`, `scripts/main.js`, `components/*` сохраняют финальную сборку FocusBoard.  
Глава 27 не ломает приложение — она **упаковывает опыт** в переносимый вид.

---

### Карта «паттерн → место в FocusBoard»

| Паттерн | Где использован |
|--------|------------------|
| Dialog form | создание задачи |
| Popover actions | меню действий задачи |
| Details filters | боковые фильтры / раскрывающиеся панели |
| Responsive image | media в карточке задачи |
| Custom element card | `task-card` и компонентный подход |

### Финальный принцип книги

Современный HTML — это не «верстка до CSS».  
Это набор браузерных возможностей для построения интерфейсов:

- смысл через семантику;
- поведение через нативные API;
- переиспользование через components/templates;
- устойчивость через progressive enhancement;
- масштаб через токены и документацию.

---

### Краткий итог главы и всего практического курса

Вы прошли путь от семантического каркаса до законченной системы:

1. Собрали FocusBoard 2026  
2. Освоили современные HTML-возможности на практике  
3. Вынесли из проекта библиотеку шаблонов  
4. Зафиксировали чек-листы качества  

На этом практическое руководство завершается как цельный результат:  
**есть проект, есть навыки, есть переносимые паттерны.**