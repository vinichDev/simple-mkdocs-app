# Отчет: Создание и развертывание статического сайта {#report}

Репозиторий: `https://github.com/vinichDev/simple-mkdocs-app`
---

## 1) О сайте {#about-site}

- Сайт сделан на **MkDocs (Python)**.
- Весь контент (включая этот отчет) хранится в `docs/`, поэтому **публикуется как страницы сайта**.
- Настроен CI/CD деплой на **GitHub Pages** через **GitHub Actions**

---


## 2) Локальная настройка и запуск (Ubuntu) {#local-setup}

### 3.1 Проверка Python {#python-check}

```bash
python3.13 --version
```

### 3.2 Создание виртуального окружения {#venv-setup}

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python --version
python -m pip --version
```

### 3.3 Установка зависимостей {#deps-install}

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

`requirements.txt`:

```txt
mkdocs==1.6.1
```

### 3.4 Локальный запуск сайта {#local-run}

Запуск dev-сервера:

```bash
python -m mkdocs serve
```

### 3.5 Сборка статических файлов {#static-build}

Сборка “как в продакшене”:

```bash
python -m mkdocs build --strict
```

Результат появляется в директории `site/` (готовая статика HTML/CSS/JS).

---

## 4) Автоматизация деплоя (CI/CD) через GitHub Actions {#cicd-actions}

Пайплайн состоит из двух стадий:

- **build**: собрать сайт и загрузить как артефакт Pages<br />
- **deploy**: развернуть артефакт на GitHub Pages
Каждый `push` в `main` автоматически запускает деплой

## 5) Исследование: отечественные CDN для ускорения доставки контента {#cdn-study}

### 5.1 Зачем CDN {#cdn-why}

CDN кэширует статический контент (HTML/CSS/JS/картинки) на узлах ближе к пользователям:
- снижает задержки (latency),
- ускоряет загрузку,
- уменьшает нагрузку на origin.

### 5.2 Примеры отечественных CDN {#cdn-examples}

- **Yandex Cloud CDN**
- **Selectel CDN**
- CDN-решения крупных операторов/платформ (как пример инфраструктуры)

### 5.3 Как подключить CDN к статическому сайту (типовой вариант) {#cdn-howto}

1. Разместить статику в **Object Storage** (S3-совместимом) или держать на origin-сервере.
2. Подключить **CDN** с origin на хранилище/сервер.
3. Настроить домен (**CNAME**) и **HTTPS** (сертификат).
4. Настроить правила кэша (TTL, `Cache-Control`, сжатие).
5. На релизе выполнять **purge/инвалидацию** кэша (часто автоматизируется в CI/CD через API).

---

## 6) Исследование: GitVerse для CI/CD {#gitverse-cicd}

### 6.1 Возможности GitVerse (обзор) {#gitverse-overview}

- Настройка CI/CD пайплайнов (подход близок к GitHub Actions).
- Запуск сборок/проверок, публикация артефактов, деплой.
- Runner’ы (self-hosted/контейнер/облако — зависит от конфигурации).

### 6.2 Как использовать GitVerse для деплоя статического сайта {#gitverse-deploy}

Варианты деплоя из GitVerse CI:
- собрать MkDocs и загрузить папку `site/`:
  - в Object Storage (S3) + подключенный CDN,
  - на VPS по SSH (rsync/scp),
  - в другой Pages-хостинг (если поддерживается).

---

## 7) Варианты деплоя статического сайта в production {#deploy-options}

### 7.1 GitHub Pages / GitLab Pages {#pages-option}

**Инструменты:** CI/CD, Pages-артефакты, домен, TLS.<br />
**Плюсы:** просто и быстро (удобно для учебных проектов).<br />
**Минусы:** только статика, ограничения платформы.

### 7.2 Object Storage + CDN (serverless hosting) {#object-storage}

**Инструменты:** S3/Object Storage, CDN, DNS, TLS, утилиты загрузки (`awscli`, `s3cmd`, `rclone`), CI/CD.<br />
**Плюсы:** высокая скорость, масштабирование, часто низкая цена.<br />
**Минусы:** нужно продумать кэш и инвалидацию.

### 7.3 VPS + Nginx/Apache {#vps}

**Инструменты:** сервер, Nginx/Apache, SSH, `rsync/scp`, `certbot`, CI/CD деплой по SSH.<br />
**Плюсы:** полный контроль, можно добавить backend.<br />
**Минусы:** администрирование, безопасность.

### 7.4 PaaS для статики (Netlify/Vercel/Cloudflare Pages и аналоги) {#paas}

**Инструменты:** встроенный build, preview deployments, CDN.<br />
**Плюсы:** удобно, превью на PR.<br />
**Минусы:** ограничения тарифов/географии, vendor lock-in.

### 7.5 Docker + Kubernetes {#docker-k8s}

**Инструменты:** Docker (nginx + статика), Kubernetes/Ingress, Helm, registry, CI/CD, мониторинг.<br />
**Плюсы:** корпоративный стандарт для больших систем.<br />
**Минусы:** избыточно для простого сайта.

---
