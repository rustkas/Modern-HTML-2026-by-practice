**Глава 3. HTML как API браузера**

### Главная идея главы

Самое важное изменение последних лет заключается не просто в появлении новых тегов, а в изменении роли HTML. Если раньше HTML описывал структуру, а поведение реализовывалось исключительно на JavaScript, то современная Web Platform всё чаще переносит поведение на декларативный уровень. HTML становится высокоуровневым API браузера, а сам браузер — интеллектуальной средой выполнения.

В 2026 году хороший фронтенд-разработчик думает не «как написать это на JavaScript», а «есть ли уже нативное решение в HTML/CSS».

### Эволюция подхода

Кратко история изменения роли HTML:

- **1995–2005** — HTML как язык документов  
- **2005–2015** — HTML + JavaScript (Web 2.0, AJAX)  
- **2015–2022** — HTML как основа для JavaScript-фреймворков (SPA)  
- **2022–2026** — HTML как декларативный API браузера (Native Components + Baseline)

Современный подход: браузер берёт на себя всё больше ответственности (модальные окна, всплывающие панели, валидация, приоритеты загрузки, переходы, инкапсуляция и т.д.).

### Концепция Baseline

Чтобы понимать, насколько безопасно использовать ту или иную возможность, важно ориентироваться на статус **Baseline**:

- **Baseline Newly available** — поддерживается во всех основных браузерах в актуальных версиях.
- **Baseline Widely available** — стабильно поддерживается уже минимум 30 месяцев.

Примеры (актуально на 2026 год):

| Возможность                      | Статус Baseline      | Примечание |
|----------------------------------|----------------------|----------|
| `<dialog>` + `showModal()`       | Widely available     | Можно использовать смело |
| Popover API                      | Newly available      | Уже можно, но учитывать поддержку |
| Declarative Shadow DOM           | Newly available      | Отлично для SSR |
| View Transition API              | Newly available      | Зависит от целевой аудитории |
| Navigation API                   | В процессе           | Следить за поддержкой |

### HTML как декларативный API — примеры мышления

Вместо того чтобы сразу писать JavaScript, задаём вопрос:

> «Можно ли решить эту задачу средствами HTML?»

Примеры:

- Модальное окно → `<dialog>`
- Всплывающее меню / тултип → Popover API
- Аккордеон → `<details>` + `<summary>`
- Валидация формы → встроенная Constraint Validation
- Ленивая загрузка изображений → `loading="lazy"`
- Приоритет загрузки → `fetchpriority`
- Инкапсуляция компонента при SSR → Declarative Shadow DOM

Именно такое мышление мы будем практиковать на протяжении всего руководства.

### Практическая задача главы

На этом этапе мы не добавляем сложную интерактивность (это будет в следующих главах). Задача Главы 3 — зафиксировать правильный подход и подготовить проект к использованию нативных API.

Что делаем:

1. Обновляем документацию проекта с учётом философии «HTML как API».
2. Добавляем в разметку семантические заготовки под будущие нативные компоненты.
3. Явно обозначаем места, где позже появятся `<dialog>`, popover и `<details>`.
4. Фиксируем принцип Progressive Enhancement.

### Структура проекта после Главы 3

```text
focusboard-2026/
├── index.html
└── README.md
```

Структура файлов пока не усложняется — мы по-прежнему работаем на уровне одного документа и правильной семантики.

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
</head>
<body>
  <!-- ===== Шапка приложения (landmark: banner) ===== -->
  <header>
    <h1>FocusBoard</h1>
    <p>Персональная панель продуктивности</p>
  </header>

  <!-- ===== Основная навигация (landmark: navigation) ===== -->
  <nav aria-label="Основная навигация">
    <ul>
      <li><a href="#tasks" aria-current="page">Задачи</a></li>
      <li><a href="#notes">Заметки</a></li>
      <li><a href="#settings">Настройки</a></li>
    </ul>
  </nav>

  <!-- ===== Поиск (landmark: search) ===== -->
  <search>
    <form role="search" aria-label="Поиск по задачам и заметкам">
      <label for="site-search">Поиск</label>
      <input type="search" id="site-search" name="q" placeholder="Поиск задач и заметок…">
      <button type="submit">Найти</button>
    </form>
  </search>

  <!-- ===== Основное содержимое (landmark: main) ===== -->
  <main id="tasks">
    <header>
      <h2>Задачи</h2>
      <p>Активные задачи на сегодня</p>
      
      <!-- Заготовка под будущую кнопку открытия <dialog> -->
      <p>
        <button type="button" id="create-task-btn" disabled>
          Создать задачу
        </button>
        <small>(появится в Главе 4 с использованием &lt;dialog&gt;)</small>
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
          <!-- Заготовка под popover-меню действий -->
          <p>
            <button type="button" disabled>Действия</button>
            <small>(Popover API — Глава 5)</small>
          </p>
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

  <!-- ===== Боковая панель (landmark: complementary) ===== -->
  <aside aria-labelledby="sidebar-heading">
    <h2 id="sidebar-heading">Фильтры и быстрые действия</h2>

    <section aria-labelledby="filters-heading">
      <h3 id="filters-heading">Фильтры</h3>
      <!-- Заготовка под <details> (Глава 6) -->
      <p>Фильтры будут реализованы через &lt;details&gt; + &lt;summary&gt;</p>
    </section>

    <section aria-labelledby="quick-actions-heading">
      <h3 id="quick-actions-heading">Быстрые действия</h3>
      <p>Создание задачи, заметки и другие действия появятся в следующих главах с использованием нативных API.</p>
    </section>
  </aside>

  <!-- ===== Подвал (landmark: contentinfo) ===== -->
  <footer>
    <p>FocusBoard 2026 · Построен на современном HTML как API браузера</p>
  </footer>

  <!-- 
    Здесь позже появятся:
    - <dialog> для создания и редактирования задач
    - элементы с popover
    - <details> для фильтров
  -->
</body>
</html>
```

#### `README.md`

```markdown
# FocusBoard 2026

Современная персональная панель продуктивности, создаваемая в рамках практического руководства  
**«Современный HTML 2026 на практике»**.

## Философия проекта

Мы строим FocusBoard по принципу:

> **Сначала нативные возможности HTML, потом JavaScript.**

HTML рассматривается не просто как язык разметки, а как декларативный API браузера.

## Текущий этап

**Глава 3. HTML как API браузера**

### Что сделано

- Зафиксирован подход «HTML как API»
- Добавлены семантические заготовки под будущие нативные компоненты
- Обозначены места для `<dialog>`, Popover API и `<details>`
- Сохранена полная поддержка Progressive Enhancement (страница полностью работоспособна без JavaScript)

### Структура проекта

```text
focusboard-2026/
├── index.html
└── README.md
```

### Принципы, которым мы следуем

1. Семантика прежде всего
2. Progressive Enhancement
3. Максимум нативных возможностей браузера
4. Доступность с первого дня
5. Производительность как часть архитектуры

## Следующие шаги

**Часть 2. Нативные интерактивные элементы**

- Глава 4. Элемент `<dialog>`
- Глава 5. Popover API
- Глава 6. `<details>` и `<summary>`
```

---

### Краткий итог главы

В этой главе мы не добавляли сложную функциональность, а зафиксировали правильный способ мышления. С этого момента при решении любой интерфейсной задачи мы сначала спрашиваем:

> «Есть ли уже нативное решение в HTML?»

И только потом пишем JavaScript.

В следующей главе мы начнём применять этот подход на практике и реализуем создание задач с помощью элемента `<dialog>`.