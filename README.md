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

## Структура репозиторію

```text
.
├── _config.yml
├── _layouts/
│   ├── default.html
│   └── post.html
├── _posts/
├── about.md
├── index.html
└── README.md
```

## Як додати нову статтю

1. Створи файл у `_posts/` з назвою:
   - `YYYY-MM-DD-slug.md`
2. Додай front matter:

```yaml
---
layout: post
title: "Назва статті"
date: 2026-05-25 10:00:00 +0200
categories: [devops]
tags: [kubernetes, ci-cd]
featured: false
---
```

3. Заповни текст статті за структурою:
   - `Проблема`
   - `Діагностика`
   - `Рішення`
   - `Перевірка результату`
   - `Підсумки` / `Що далі`

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
