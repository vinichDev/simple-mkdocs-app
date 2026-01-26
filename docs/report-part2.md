# Отчет (часть 2): кастомная тема MkDocs и CI/CD сборка {#report-part2}

> Репозиторий: `https://github.com/vinichDev/simple-mkdocs-app`
>
> GitHub Pages: `https://vinichDev.github.io/simple-mkdocs-app/`

---

## 1) Результат {#result}

- Создана **собственная тема** на HTML/CSS/JS с кастомными `header` и `footer`.
- Домашняя страница получила отдельную стилизацию (карточка с тенью и крупным заголовком).
- В шапке и подвале прописаны метаданные сайта.
- Настроена сборка через **PostCSS**.
- Добавлены этапы **валидации HTML** и **минификации HTML** в CI.
- Сайт собирается и деплоится на **GitHub Pages** через GitHub Actions.

---

## 2) Структура кастомной темы {#theme-structure}

```
/theme
  base.html
  main.html
  /css
    style.css
  /js
    main.js
/theme-src
  style.css
```

- `theme-src/style.css` — исходники CSS.
- `theme/css/style.css` — итоговый файл после PostCSS.
- `theme/base.html` — основной HTML-шаблон с `header`, `footer` и meta.
- `theme/main.html` — вставка Markdown-контента в шаблон.

---

## 3) PostCSS, валидация и минификация {#postcss-html}

Используется `postcss` + `autoprefixer` + `cssnano`.

```bash
npm run build:css
```

HTML проверяется и минифицируется после сборки MkDocs:

```bash
npm run validate:html
npm run minify:html
```

---

## 4) Сборка и деплой в GitHub Actions {#gha-deploy}

Файл: `.github/workflows/deploy-pages.yml`

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: ["master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install Node dependencies
        run: npm install

      - name: Build CSS with PostCSS
        run: npm run build:css

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install Python dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Build MkDocs site
        run: mkdocs build --strict

      - name: Validate HTML
        run: npm run validate:html

      - name: Minify HTML
        run: npm run minify:html

      - name: Configure Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 5) Локальный запуск {#local-run}

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

npm install
npm run build:css

mkdocs serve
```

---

## 6) Итоги {#summary}

- Сайт доступен на GitHub Pages.
- В CI добавлены PostCSS, HTML-валидация и минификация.
