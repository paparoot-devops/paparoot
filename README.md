# Ops Notes

Блог на GitHub Pages про Linux, DevOps та системну інженерію.

## Що зараз реалізовано

- Кастомна головна сторінка у стилі стрічки: `index.html`
- Єдиний стиль для всіх сторінок:
  - `_layouts/default.html`
  - `_layouts/post.html`
- Окрема сторінка "Про блог": `about.md`
- Вбудований швидкий пошук на головній сторінці (за заголовком, прев'ю та тегами)
- Пости у форматі Jekyll в каталозі: `_posts/`
- Шаблони статей у каталозі: `templates/`

## Формати статей

### 1) Existing System Change (`change-case`)

Для кейсів, де система вже існує, а ти вносиш точкові зміни.

Шаблон:
- `templates/post-existing-system.md`

### 2) Migration / Refactor (`migration`)

Для міграцій, переробок або поетапного переносу.

Шаблон:
- `templates/post-migration.md`

## Структура репозиторію

```text
.
├── _config.yml
├── _layouts/
│   ├── default.html
│   └── post.html
├── _posts/
├── templates/
│   ├── post-existing-system.md
│   └── post-migration.md
├── about.md
├── index.html
└── README.md
```

## Як додати нову статтю

1. Обери шаблон у `templates/`.
2. Скопіюй його у `_posts/` з назвою:
   - `YYYY-MM-DD-slug.md`
3. Заповни front matter (`title`, `date`, `tags`, `case_type`, `featured`).
4. Заповни розділи статті за шаблоном.

## Публікація на GitHub Pages

1. Commit і push у `main`
2. У GitHub (`Settings -> Pages`) має бути:
   - `Deploy from a branch`
   - branch: `main`
   - folder: `/ (root)`

Сайт буде доступний за адресою:
- `https://paparoot-devops.github.io/paparoot/`

## Нотатка щодо даних

Для публікацій використовуй знеособлений формат:
- без секретів
- без реальних доменів
- без внутрішніх ідентифікаторів
