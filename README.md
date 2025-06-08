# E-commerce Platform: Telegram Bot & Django Backend

Это полнофункциональная e-commerce платформа, реализованная в виде Telegram-бота с мощной админ-панелью на Django. Проект полностью контейнеризирован с использованием Docker и Docker-Compose, что обеспечивает простое развертывание и масштабируемость.

![[media/scheme.png]]

## 🚀 Ключевые возможности

### Telegram-бот (клиентская часть):
- **Сложные сценарии (FSM):** Полноценная машина состояний (Aiogram 3 FSM) для навигации по каталогу, управления корзиной, оформления заказа и раздела FAQ.
- **Динамическая пагинация:** Собственный переиспользуемый компонент `Paginator` для асинхронной подгрузки и постраничного отображения данных в любом разделе бота.
- **Управление корзиной:** Добавление/удаление товаров, изменение количества, просмотр и оформление заказа.
- **Прием платежей:** Интеграция с Telegram Payments для обработки тестовых заказов.
- **Проверка подписки:** Middleware для проверки подписки на обязательные каналы.

### Django (бэк-офис):
- **Централизованное управление:** Админ-панель для управления пользователями, иерархией категорий, товарами, заказами и контентом (FAQ, каналы).
- **Массовые рассылки:** Система для создания и управления немедленными и отложенными рассылками через Celery.
- **Экспорт данных:** Выгрузка заказов в форматы CSV/Excel с помощью `django-import-export`.

### Архитектура и DevOps:
- **Мульти-контейнерная среда:** Проект разворачивается одной командой `docker-compose up` и включает сервисы: Nginx, Django, Aiogram Bot, PostgreSQL, Redis и **два отдельных Celery-воркера**.
- **Асинхронные задачи:** Использование Celery для обработки ресурсоемких задач (рассылки), что гарантирует отзывчивость бота.
- **Разделение очередей:** Для оптимизации производительности созданы две разные очереди Celery:
  - `default`: для общих задач Django.
  - `telegram_sending_queue`: для отправки сообщений в Telegram, чтобы избежать блокировок из-за API-лимитов.

## 🛠️ Технологический стек

| Категория       | Технологии                                                    |
| --------------- | ------------------------------------------------------------- |
| **Backend**     | Python 3.13, Django 5, Aiogram 3, Celery 5                      |
| **Базы данных** | PostgreSQL (основная), Redis (брокер сообщений Celery)      |
| **Инфраструктура** | Docker, Docker-Compose, Nginx, Gunicorn                       |
| **Инструменты**   | Poetry, Pydantic (для конфигурации)                          |

## ⚙️ Быстрый старт

**Предварительные требования:**
*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/)

---

### 1. Клонирование репозитория
```bash
git clone https://github.com/Chthonic-Galaxy/merchandise-store.git
cd merchandise-store
```

### 2. Настройка окружения
Создайте файл `.env` в корне проекта, скопировав `.env.example`, и заполните его своими данными:
```bash
cp .env.example .env
```
**Содержимое `.env`:**
```env
# PostgreSQL
POSTGRES_DB=store_db
POSTGRES_USER=store_user
POSTGRES_PASSWORD=your_strong_password

# Django
DJANGO_KEY='your_super_secret_django_key_here' # Можно сгенерировать через: python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
DJANGO_DEBUG=True

# Telegram Bot
BOT_TOKEN='your_telegram_bot_token_from_botfather'
```

### 3. Сборка и запуск
Запустите все сервисы с помощью Docker Compose. При первом запуске будут созданы образы, применены миграции и запущены все компоненты.
```bash
docker-compose up --build -d
```

### 4. Создание суперпользователя
Для доступа к админ-панели Django создайте суперпользователя:
```bash
docker-compose exec django python src/admin_panel/manage.py createsuperuser
```
Следуйте инструкциям в терминале.

## ✅ Доступ к сервисам

*   **Django Admin Panel:** [http://localhost:8000/admin/](http://localhost:8000/admin/)
*   **Telegram Bot:** Найдите вашего бота в Telegram по имени пользователя и отправьте команду `/start`.

## 🎛️ Основные команды

| Действие                                | Команда                                                                   |
| --------------------------------------- | ------------------------------------------------------------------------- |
| **Запустить все сервисы**              | `docker-compose up -d`                                                    |
| **Остановить все сервисы**             | `docker-compose down`                                                     |
| **Остановить и удалить данные** (DB, logs) | `docker-compose down -v`                                                  |
| **Посмотреть логи сервиса** (`django`)  | `docker-compose logs -f django`                                           |
| **Выполнить команду Django**            | `docker-compose exec django python src/admin_panel/manage.py <команда>`   |

## 📝 Управление проектом

### Django
- **Создать миграции** (после изменения моделей в `clients`):
  ```bash
  docker-compose exec django python src/admin_panel/manage.py makemigrations clients
  ```
- **Применить миграции:**
  ```bash
  docker-compose exec django python src/admin_panel/manage.py migrate
  ```
  *(Обычно выполняется автоматически при старте контейнера `django`)*

### Celery
- **Просмотр логов воркеров:**
  ```bash
  # Для Django-задач
  docker-compose logs -f celery_worker_django
  
  # Для задач отправки в Telegram
  docker-compose logs -f bot_sender_worker
  ```