**Глава 5. Popover API — всплывающие интерфейсы**

### Главная идея главы

Popover API (Baseline *Newly available* с января 2025 года) — это нативный механизм создания всплывающих элементов: контекстных меню, тултипов, выпадающих панелей и лёгких оверлеев. В отличие от `<dialog>`, popover не блокирует всю страницу и поддерживает «лёгкое закрытие» (Light Dismiss) — закрытие по клику вне элемента или по клавише Escape.

Основные преимущества:
- Автоматическое помещение в Top Layer (больше никаких проблем с `z-index`)
- Light Dismiss из коробки
- Простая связь кнопки и popover через атрибуты
- Хорошая доступность

### Основные возможности

- Атрибут `popover` (значения `auto` | `manual`)
- Атрибуты `popovertarget` и `popovertargetaction` на кнопке
- Методы JavaScript: `showPopover()`, `hidePopover()`, `togglePopover()`
- События `beforetoggle` и `toggle`
- Псевдокласс `:popover-open`

Режим `auto` (по умолчанию) — только один такой popover может быть открыт, работает Light Dismiss.  
Режим `manual` — управление полностью вручную.

### Практическая задача главы

Добавить к каждой задаче кнопку «Действия», которая открывает контекстное меню на основе Popover API с пунктами:
- Редактировать
- Отметить как выполненную
- Удалить

Пока пункты меню будут только выводить действие в консоль. Полноценную логику добавим позже.

### Структура проекта после Главы 5

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css
├── scripts/
│   └── main.js
└── README.md
```

Структура файлов не изменилась, но содержимое существенно обновилось.

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
      <input type="search" id="site-search" name="q" placeholder="Поиск задач и заметок…">
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

      <!-- Задача 1 -->
      <article class="task" aria-labelledby="task-1-title" data-task-id="1">
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

      <!-- Задача 2 -->
      <article class="task" aria-labelledby="task-2-title" data-task-id="2">
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

      <article class="task task--completed" aria-labelledby="task-3-title" data-task-id="3">
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

    <section aria-labelledby="filters-heading">
      <h3 id="filters-heading">Фильтры</h3>
      <p>Фильтры будут реализованы через &lt;details&gt; + &lt;summary&gt;</p>
    </section>

    <section aria-labelledby="quick-actions-heading">
      <h3 id="quick-actions-heading">Быстрые действия</h3>
      <p>Создание задачи доступно через кнопку выше.</p>
    </section>
  </aside>

  <!-- ===== Подвал ===== -->
  <footer>
    <p>FocusBoard 2026 · Построен на современном HTML</p>
  </footer>

  <!-- ===== Диалог создания задачи ===== -->
  <dialog id="create-task-dialog" aria-labelledby="create-task-title">
    <form method="dialog" id="create-task-form">
      <header>
        <h2 id="create-task-title">Новая задача</h2>
      </header>

      <p>
        <label for="task-title">Название</label>
        <input type="text" id="task-title" name="title" required placeholder="Что нужно сделать?">
      </p>

      <p>
        <label for="task-description">Описание</label>
        <textarea id="task-description" name="description" rows="3" placeholder="Дополнительные детали (необязательно)"></textarea>
      </p>

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
  max-width: 420px;
  width: 90%;
  box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1);
}

dialog::backdrop {
  background-color: rgb(15 23 42 / 0.5);
}

dialog header {
  margin-bottom: 1rem;
}

dialog menu {
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
  padding: 0;
  margin-top: 1.5rem;
}

dialog label {
  display: block;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

dialog input,
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

/* ===== Popover (контекстное меню задачи) ===== */

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

/* Можно дополнительно стилизовать открытое состояние */
.task-menu:popover-open {
  /* стили при открытии, если нужны */
}
```

#### `scripts/main.js`

```js
// ===== Диалог создания задачи =====
const createTaskBtn = document.getElementById('create-task-btn');
const createTaskDialog = document.getElementById('create-task-dialog');
const createTaskForm = document.getElementById('create-task-form');

createTaskBtn.addEventListener('click', () => {
  createTaskDialog.showModal();
});

createTaskForm.addEventListener('close', () => {
  const returnValue = createTaskDialog.returnValue;

  if (returnValue === 'confirm') {
    const formData = new FormData(createTaskForm);
    const title = formData.get('title');
    const description = formData.get('description');

    console.log('Новая задача:', { title, description });
    createTaskForm.reset();
  }

  if (returnValue === 'cancel') {
    createTaskForm.reset();
  }
});

// ===== Popover: действия с задачами =====
document.addEventListener('click', (event) => {
  const actionBtn = event.target.closest('[data-action]');
  
  if (!actionBtn) return;

  const action = actionBtn.dataset.action;
  const taskId = actionBtn.dataset.taskId;

  console.log(`Действие "${action}" для задачи #${taskId}`);

  // Закрываем popover после выбора действия
  const popover = actionBtn.closest('[popover]');
  if (popover) {
    popover.hidePopover();
  }
});
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 5. Popover API — всплывающие интерфейсы**

### Что реализовано

- Контекстные меню задач на основе Popover API
- Связь кнопки и меню через `popovertarget` / `popovertargetaction`
- Режим `popover="auto"` с Light Dismiss
- Обработка выбора пункта меню
- Стилизация popover

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

1. Откройте страницу.
2. Нажмите «Действия» у любой активной задачи.
3. Убедитесь, что меню появляется поверх контента.
4. Кликните вне меню — оно должно закрыться (Light Dismiss).
5. Выберите любой пункт — действие появится в консоли, меню закроется.

## Следующие шаги

- Глава 6. `<details>` и `<summary>`
- Реализация фильтров в боковой панели
```

---

### Важные замечания по реализации

- Мы использовали декларативный способ открытия (`popovertarget`) — это предпочтительный подход.
- Режим `auto` даёт удобное поведение «один открытый popover + закрытие по клику снаружи».
- После выбора пункта меню мы явно вызываем `hidePopover()` для мгновенного закрытия.
- Пока логика действий только логируется. В следующих главах свяжем её с реальным изменением состояния задач.

В следующей главе мы добавим декларативные раскрывающиеся блоки с помощью `<details>` и `<summary>` для фильтров.