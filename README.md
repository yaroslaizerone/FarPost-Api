![example event parameter](https://github.com/avanslov/farpost-django-api/actions/workflows/main.yml/badge.svg?event=push)

# Описание

**API-сервис для получения данных о первых 10 объявлениях по [ссылке](https://www.farpost.ru/vladivostok/service/construction/guard/+/%D0%A1%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B+%D0%B2%D0%B8%D0%B4%D0%B5%D0%BE%D0%BD%D0%B0%D0%B1%D0%BB%D1%8E%D0%B4%D0%B5%D0%BD%D0%B8%D1%8F/) с сервиса farpost.ru**

Проект разработан таким образом, чтобы его было легко расширять, тестровать и запускать в работу.

Выполнены все работы по предварительной настройке CI/CD.

Проект легко запустить локально с помощью docker-compose.

# Cтек
![Python](https://img.shields.io/badge/-Python-black?style=for-the-badge&logo=python)
![Django](https://img.shields.io/badge/-Django_REST_FRAMEWORK-black?style=for-the-badge&logo=Django)
![Scrapy](https://img.shields.io/badge/-Scrapy-black?style=for-the-badge&logo=Scrapy)
![Nginx](https://img.shields.io/badge/-Nginx-black?style=for-the-badge&logo=Nginx)
![Docker](https://img.shields.io/badge/-Docker-black?style=for-the-badge&logo=Docker)
![Swager](https://img.shields.io/badge/-Swager-black?style=for-the-badge&logo=Swager)
![GitHub](https://img.shields.io/badge/-GitHub_Actions-black?style=for-the-badge&logo=GitHub)


# Установка и запуск

***Клонировать репозиторий и перейти в него в командной строке:***

```
git clone git@github.com:your_username_in_github/farpost-django-api.git
```

***Cоздать и активировать виртуальное окружение:***
```

Для Windows:
python -m venv env
source venv/Script/activate

Для Linux/MacOS:
python3 -m venv env
source venv/bin/activate
```
***Установить зависимости из файла requirements.txt:***

```
python -m pip install --upgrade pip
pip install -r requirements.txt
```

***Как заполнить .env:***

В случае запуска в режиме DEBUG=False корневой папке проекта создайте файл .env и скопируйте в него код с поля ниже.

```
POSTGRES_USER=django_user
POSTGRES_PASSWORD=mysecretpassword
POSTGRES_DB=django
DB_HOST=db
DB_PORT=5432
```

**Запуск проекта**

Для запуска проекта поочередно выполните команды из листинга

```
sudo docker compose stop && sudo docker compose up --build

# миграции и запуск парсера выполняется автоматически в Dockerfile

sudo docker compose exec backend python manage.py collectstatic

sudo docker compose exec backend cp -r /app/farpost/collected_static/. /backend_static/static/

sudo docker compose exec backend python manage.py loaddata ../data/final_data_farpost_authors.json

sudo docker compose exec backend python manage.py loaddata ../data/final_data_farpost_adds.json
```

## Как реализован сбор данных

С помощью фреймворка для парсинга данных Scrapy собираются данные. Для этого последовательно выполняются две операции.

**Сбор данных непосредственно со страницы поиска:**
- количество просмотров;
- позиции в списке выдачи;
- названия позиций.

**С помощью CrawlScrapy во втором скрипте паук заходит на страницу каждого объявления:**
- для получения информации об авторе

Далее данные конвертируются и склеиваются в единый JSON для последующего импорта в БД.

## API

### Документация к API

Оформленная документация к API доступна после запуска приложения по адресу:
http://localhost:8000/swagger/

#### Авторизация

Авторизация реализована с помощью Djoser.
В настройках проекта задан срок жизни токена 1 день, установлен лимит запросов в сутки.

Регистрация и авторизация состоит из двух шагов.

POST запрос с логином и паролем

```
POST http://localhost:8000/auth/users/
Content-Type: application/json

{
    "username": "test",
    "password":  "lsjafw39hfd"
}
```
POST запрос для получения JWT токена для последющей передачи в каждом запросе

```
POST http://localhost:8000/auth/jwt/create/
Content-Type: application/json

{
    "username": "test",
    "password":  "lsjafw39hfd"
}
```
Токен следует передавать следующим образом:

```
GET http://localhost:8000/api/adds/
Content-Type: application/json
Authorization: Bearer <токен>
```

### Как протестировать API

Запросы для тестирования доступны в корневой папке проекта в файле requests.http

**В данном API есть возможность выполнить следующие запросы:**

- Получение всех объявлений, при этом на уровне проекта настроена пагинация по 5 объявлений на страницу.

- Получение конкретного объявления по его ID номеру, который указан на [сайте источнике](https://www.farpost.ru/vladivostok/service/construction/guard/+/%D0%A1%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D1%8B+%D0%B2%D0%B8%D0%B4%D0%B5%D0%BE%D0%BD%D0%B0%D0%B1%D0%BB%D1%8E%D0%B4%D0%B5%D0%BD%D0%B8%D1%8F/).

*ID объявления храниться в БД в поле add_id модели Add.*

**Примеры запросов**
```
###
GET http://localhost:8000/api/adds/
Content-Type: application/json
Authorization: Bearer <токен>
```
```
###
GET http://localhost:8000/api/adds/<add_id>/
Content-Type: application/json
Authorization: Bearer <токен>
```

## Идеи по развитию

1. **Оптимизировать логику парсера**:
    - Объединить два скрипта в один для генерации JSON.
    - Улучшить обработку ошибок и добавление логирования для отслеживания процесса парсинга.
    - Добавить возможность настройки параметров парсинга (например, задержка между запросами, количество одновременных запросов).

2. **Добавить логику для исключения дубликатов объектов автора в базе данных при импорте фикстур**:
    - Проверять наличие автора в базе данных перед добавлением нового объекта.
    - Реализовать механизм обновления информации об авторе, если он уже существует в базе данных.

3. **Написать тесты, используя Pytest**:
    - Покрыть тестами все основные функциональности парсера.
    - Написать тесты для API, включая тесты на авторизацию, получение списка объявлений и получение конкретного объявления по ID.
    - Настроить автоматический запуск тестов при каждом коммите (например, используя GitHub Actions).

4. **Улучшить внешний вид админ панели и уточнить локализацию**:
    - Настроить тему оформления админ панели, чтобы сделать её более удобной и привлекательной для пользователей.
    - Добавить переводы для всех используемых текстов, чтобы поддерживать многоязычность.

5. **Разработать систему уведомлений**:
    - Реализовать отправку уведомлений администратору при возникновении ошибок в процессе парсинга.
    - Добавить уведомления пользователям при успешной регистрации или изменении информации о профиле.

6. **Интеграция с другими сервисами**:
    - Рассмотреть возможность интеграции с другими платформами для автоматического импорта данных.
    - Реализовать экспорт данных в другие форматы или системы.

7. **Оптимизация производительности**:
    - Настроить кэширование часто запрашиваемых данных для уменьшения нагрузки на сервер.
    - Оптимизировать запросы к базе данных, чтобы минимизировать время выполнения.

8. **Дополнительные возможности для пользователей**:
    - Разработать личный кабинет пользователя с возможностью управления своими объявлениями.
    - Добавить возможность оставлять комментарии или отзывы под объявлениями.

9. **Мониторинг и аналитика**:
    - Внедрить систему мониторинга для отслеживания состояния сервера и приложения.
    - Разработать отчеты и аналитические панели для анализа собранных данных.

10. **Документация и поддержка**:
    - Подробно описать процесс установки и настройки приложения.
    - Создать раздел с часто задаваемыми вопросами.


## Contact me

[![Telegram](https://img.shields.io/badge/Telegram-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/yaroslaizerone)
[![E-mail](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:kolpackov.yarosl@yandex.ru)