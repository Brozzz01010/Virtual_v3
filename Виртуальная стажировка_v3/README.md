## Задача виртуальной стажировки:
Проект Федерации спортивного туризма России (ФСТР) "Перевал Online" представляет
самую большую базу по горным объектам России и СНГ. Проект создан
специально для горных путешественников.
На сайте https://pereval.online/ ФСТР ведёт базу горных перевалов,
которая пополняется туристами.

Задачей стажировки являлось разработать REST API для мобильного приложения ФСТР и релизовать методы:

1. POST submitData - отправка данных о перевале,
2. GET /<id> - получение данных о перевале по id,
3. GET /submitData/?email_user=<email> - получить данные об отправленных перевалах
по email пользователя,
4. PATCH /submitData/<id> - отредактировать данные о перевале, принимает JSON
в теле запроса (тот же что и при отправке нового перевала). Можно редактировать
все поля кроме данных пользователя (ФИО, email и пр.)

## Requirements
Для реализации задачи был выбран FastAPI, PostgreSQL, Docker Compose.

asyncpg~=0.22   
fastapi~=0.79   
sqlalchemy~=1.4     
psycopg2-binary~=2.9    
uvicorn~=0.18   
python-dotenv~=0.20     
pydantic[email] 
requests~=2.28
httpx
pytest~=7.2

## Выполнено:
В рамках основной задачи:
1. Написан скрипт для создания БД;
2. Изначальная БД, предоставленная ФСТР, была улучшена
в соответствии с рекомендациями (pereval.sql), были добавлены дополнительные столбцы, таблицы.


## Запуск проекта в Docker Compose
    $ docker-compose up -d --build
При первом запуске будет создана БД (pereval.sql) с одной записью
о перевале (id=1).

API будет доступно по ссылке: http://0.0.0.0:8008/  
Документация API (Swagger UI): http://0.0.0.0:8008/docs

Методы API:

1. Метод GET/pereval/{id} - получение информации о первевале по ID.
Если перевал есть в БД, будет выведена вся информация о перевале,
включая данные о пользователе, координатах, уровне.
Пример запроса: http://0.0.0.0:8008/pereval/1/

Пример вывода:

    {
        "id": 1,
        "add_time": "2021-09-22T13:18:13",
        "beauty_title": "пер.",
        "title": "Пхия",
        "other_titles": "Триев",
        "connect": null,
        "user": {
            "email": "user@email.ltd",
            "phone": 79031234567,
            "fam": "Пупкин",
            "name": "Василий",
            "otc": "Иванович"
        },
        "coords": {
            "latitude": "45.3842",
            "longitude": "7.1525",
            "height": "1200"
        },
        "level": {
            "winter": "",
            "summer": "1A",
            "autumn": "1A",
            "spring": ""
        },
        "status": "pending"
    }

Если перевала нет в БД: http://0.0.0.0:8008/pereval/2/

        {
          "status": 400,
          "message": "Перевал не найден",
          "id": "2"
        }

2. Метод POST/submitData/(принимает JSON в теле запроса) - отправка
данных по перевалу.

Пример JSON с информацией о перевале:

    {
      "beauty_title": "пер.",
      "title": "Пхия",
      "other_titles": "Триев",
      "connect": "",
      "add_time": "2021-09-22 13:18:13",
      "user": {
        "email": "user@email.ltd",
        "phone": "75555555",
        "fam": "Пупкин",
        "name": "Василий",
        "otc": "Иванович"
      },
      "coords": {
        "latitude": "45.3842",
        "longitude": "7.1525",
        "height": "1200"
      },
      "level": {
        "winter": "",
        "summer": "1A",
        "autumn": "1A",
        "spring": ""
      },
      "images": [{"data": "image1", "title": "Сeдловина"},
       {"data": "image2", "title": "Подъем"}]
    }

Пример вывода в случае успешной отправки:

    {
      "status": 200,
      "message": "Отправлено успешно",
      "id": 2
    }



3. Метод GET/submitData/email/{email} - по почте пользователя выводит краткую
информацию о всех добавленых перевалах, включая статус.
Пример запроса: http://0.0.0.0:8008/submitData/email/user@email.ltd 

Пример вывода:

    {
      "id": 1,
      "title": "Пхия",
      "other_titles": "Триев",
      "add_time": "2022-02-21T14:14:00",
      "status": "new"
    },
    {
      "id": 2,
      "title": "новый",
      "other_titles": "перевал",
      "add_time": "2021-09-21T12:12:12",
      "status": "new"
    }

Если пользователь не найден:

    {
    "status": 400,
    "message": "Пользователь с данным email не зарегистрирован"
    }

4. Метод PATCH/submitData/{id} - отредактировать существующую запись
(замена), если она в статусе new.
Редактировать можно все поля, кроме тех, что содержат в себе ФИО,
адрес почты и номер телефона. Метод принимает тот же JSON,
что и при отправке данных нового перевала.

Пример JSON:

    {
      "beauty_title": "пер.",
      "title": "Пхия",
      "other_titles": "Триев",
      "connect": "",
      "add_time": "2021-09-22 13:18:13",
      "user": {
        "email": "qwerty@mail.ru",
        "phone": "75555555",
        "fam": "Пупкин",
        "name": "Василий",
        "otc": "Иванович"
      },
      "coords": {
        "latitude": "45.3842",
        "longitude": "7.1525",
        "height": "1200"
      },
      "level": {
        "winter": "",
        "summer": "1A",
        "autumn": "1A",
        "spring": ""
      },
      "images": [{"data": "image1", "title": "Сeдловина"},
       {"data": "image2", "title": "Подъем"}]
    }

После внесения изменений в запись, будет сообщение об успешной отправке

    {
      "state": 1,
      "message": "Отправлено успешно"
    }

Если статус перевала не "new", будет выведено сообщение с актуальным статусом:

    {
      "state": 0,
      "message": "Невозможно внести изменения. Статус перевала: в работе"
    }

В случае, если в БД нет перевала с таким id:

    {
      "status": 400,
      "message": "Перевал не найден",
      "id": "123"
    }

## Тесты

    $ docker-compose exec web pytest