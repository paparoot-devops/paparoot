# DevOps Cases Blog

Блог на GitHub Pages с практическими кейсами из DevOps.

## Локальный запуск

Если нужен локальный просмотр:

```bash
bundle install
bundle exec jekyll serve
```

## Публикация

1. Push в `main`
2. В GitHub: `Settings -> Pages`
3. Source: `Deploy from a branch`
4. Branch: `main`, folder: `/ (root)`

После деплоя блог будет доступен на `https://<username>.github.io/`.
