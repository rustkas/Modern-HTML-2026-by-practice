**Глава 4. Элемент `<dialog>` — нативные модальные окна**

### Главная идея главы

Элемент `<dialog>` — одна из самых зрелых и полезных возможностей современного HTML (Baseline *Widely available* с марта 2022 года). Он позволяет создавать доступные модальные и немодальные окна без необходимости вручную управлять фокусом, `z-index`, блокировкой фона и обработкой клавиши Escape.

В этой главе мы заменим заготовку кнопки «Создать задачу» на полноценный нативный диалог.

### Что умеет `<dialog>`

- Модальный режим (`showModal()`) — блокирует взаимодействие с остальной страницей и добавляет `::backdrop`
- Немодальный режим (`show()`) — окно открывается, но страница остаётся активной
- Автоматическое управление фокусом
- Закрытие по клавише `Escape`
- Поддержка формы с `method="dialog"`
- Псевдоэлемент `::backdrop` для затемнения фона
- Методы `close()` и `requestClose()`

### Базовый синтаксис

```html
<dialog id="create-task-dialog">
  <form method="dialog">
    <h2>Новая задача</h2>
    <!-- поля формы -->
    <menu>
      <button value="cancel">Отмена</button>
      <button value="confirm">Создать</button>
    </menu>
  </form>
</dialog>
```

Открытие:

```js
const dialog = document.getElementById('create-task-dialog');
dialog.showModal();
```

### Практическая задача главы

Реализовать создание новой задачи через нативный `<dialog>`:

1. Кнопка «Создать задачу» открывает модальное окно.
2. Внутри диалога — форма с полями названия и описания.
3. Кнопка «Отмена» закрывает диалог.
4. Кнопка «Создать» закрывает диалог и пока просто выводит данные в консоль (логику сохранения добавим позже).
5. Полная доступность из коробки.

### Структура проекта после Главы 4

```text
focusboard-2026/
├── index.html
├── styles/
│   └── main.css          # Минимальные стили для диалога
├── scripts/
│   └── main.js           # Логика открытия/закрытия диалога
└── README.md
```

На этом этапе мы впервые добавляем CSS и JavaScript (минимально необходимое).

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

      <article class="task" aria-labelledby="task-1-title">
        <header>
          <h4 id="task-1-title">Изучить современный HTML</h4>
        </header>
        <p>Разобрать семантику, dialog, popover и Web Components.</p>
        <footer>
          <p>Приоритет: высокий</p>
        </footer>
      </article>

      <article class="task" aria-labelledby="task-2-title">
        <header>
          <h4 id="task-2-title">Собрать семантический каркас FocusBoard</h4>
        </header>
        <p>Создать правильную структуру документа с landmarks.</p>
        <footer>
          <p>Приоритет: средний</p>
        </footer>
      </article>
    </section>

    <section aria-labelledby="completed-tasks-heading">
      <h3 id="completed-tasks-heading">Выполненные</h3>

      <article class="task task--completed" aria-labelledby="task-3-title">
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
/* Минимальные стили для читаемости и диалога */

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
```

#### `scripts/main.js`

```js
const createTaskBtn = document.getElementById('create-task-btn');
const createTaskDialog = document.getElementById('create-task-dialog');
const createTaskForm = document.getElementById('create-task-form');

// Открытие диалога
createTaskBtn.addEventListener('click', () => {
  createTaskDialog.showModal();
});

// Обработка закрытия через форму (method="dialog")
createTaskForm.addEventListener('close', (event) => {
  // event.target.returnValue содержит value нажатой кнопки
  const returnValue = createTaskDialog.returnValue;

  if (returnValue === 'confirm') {
    const formData = new FormData(createTaskForm);
    const title = formData.get('title');
    const description = formData.get('description');

    console.log('Новая задача:', { title, description });

    // Пока просто очищаем форму.
    // В следующих главах будем добавлять задачу на страницу.
    createTaskForm.reset();
  }

  // При отмене тоже очищаем форму
  if (returnValue === 'cancel') {
    createTaskForm.reset();
  }
});
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности.

## Текущий этап

**Глава 4. Элемент `<dialog>` — нативные модальные окна**

### Что реализовано

- Кнопка «Создать задачу»
- Нативный модальный диалог на основе `<dialog>`
- Форма с `method="dialog"`
- Управление открытием через `showModal()`
- Затемнение фона через `::backdrop`
- Базовая обработка подтверждения и отмены
- Минимальные стили для диалога

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

1. Откройте `index.html` в браузере.
2. Нажмите «Создать задачу».
3. Проверьте, что диалог открывается модально, фокус попадает внутрь, фон затемняется.
4. Нажмите Escape или кнопку «Отмена» — диалог должен закрыться.
5. Заполните форму и нажмите «Создать» — данные появятся в консоли.

## Следующие шаги

- Глава 5. Popover API
- Добавление контекстных меню к задачам
```

---

### Важные замечания

- Мы использовали `method="dialog"` — это позволяет закрывать диалог нативно, без лишнего JavaScript.
- `returnValue` позволяет понять, какой кнопкой закрыли диалог.
- Стили минимальны — акцент на функциональности, а не на дизайне.
- JavaScript пока только открывает диалог и обрабатывает результат. Логику добавления задачи на страницу реализуем позже.

В следующей главе мы добавим Popover API для контекстных меню действий над задачами.