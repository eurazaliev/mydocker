1. git clone 
2. docker-compose build
3. mkdir src
4. cd src
5. git clone https://github.com/eurazaliev/bookJsonAPI.git .
6. cp .env .env.local
7. в .env.local добавить строку:
    DATABASE_URL="mysql://symfony:symfony@database:3306/books??serverVersion=8.0"
    (изменить логин пароль и имя БД, если поменяли в ../.env)
8. docker-compose up -d
9. docker-compose ps - убедиться, что все 3 контейнера в up
10. docker-compose exec php-fpm /bin/bash
11. внутри контейнера: composer install
12. убедиться, что БД доступна mysql -h database -u symfony -p после логина проверить, что БД books существует
13. bin/console doctrine:migrations:migrate - выполнить миграцию(одна из миграций содержит заливку текстур)

endpoints, вживую можно проверить на моем хостине:

1. http://aab.clouds.today:801/author/create/ - POST создание автора, нужно оптравить json следующего вида:
{"name": "Alexander Pushkin"}
response:
{
    "id": 10002,
    "name": "Alexander Pushkin"
}

2. http://aab.clouds.today/book/create/ - POST создание книги, формат json запроса:
{"nameRu": "Полтава", "nameEn": "Полтава", "authorId": 10002}
nameRu/En - названия книги на языке соответственно, authorId - ид автора в базе
если json отправили плохой или несуществущий автор, придет с кодом 400:
{
    "errors": "Incorrect author"
}

если все ок, то придет с кодом 200
response:
{
    "id": 1002,
    "author": {
        "name": "Alexander Pushkin"
    }
}

3. http://aab.clouds.today/book/search/ - GET поиск книги по названию.
я тут не делал в тестовом задании полноценный поиск по индексам, просто сделал через LIKE.
но чтобы сервер и клиент не поперхнулись от результатов, сделал, что выдача результатов с пейджинацией.
поэтому запрос, помимо запроса на поиск, может содержать еще и параметры для педжинга:
{"query": "Мир", "itemsPerPage": 5, "page": 4}
позиций на страницу 1-5(по-умолчанию 5). если запросить больше - сервер ответит 400
страница по-умолчанию 1
ответ такой:
{
    "item": [
        {
            "name": "Программируемый веб-поддержка сайт Рената ООО ВекторМонтаж",
            "book": {
                "id": 818,
                "author": {
                    "name": "Филипп Овчинников"
                }
            }
        },
        {
            "name": "Программируемый ориентированный на решение вызов ПАО МоторТех Алиса",
            "book": {
                "id": 828,
                "author": {
                    "name": "Донат Суворова"
                }
            }
        },
    ...
    ],
    "itemsinset": 5,
    "page": 4,
    "itemsperpage": 5,
    "itemstotal": 21,
    "exp": {
        "last": 5,
        "current": 4,
        "numItemsPerPage": 5,
        "first": 1,
        "pageCount": 5,
        "totalCount": 21,
        "pageRange": 5,
        "startPage": 1,
        "endPage": 5,
        "previous": 3,
        "next": 5,
        "pagesInRange": [
            1,
            2,
            3,
            4,
            5
        ],
        "firstPageInRange": 1,
        "lastPageInRange": 5,
        "currentItemCount": 5,
        "firstItemNumber": 16,
        "lastItemNumber": 20
    }
}
нижний блок немного помогает клиенту с пейджинацией

5. http://aab.clouds.today/{lang}/book/{id}/ - для GET запросов по выдаче названия книги с id по языку lang(ru|en)
если книги нет - будет код 400
{
    "errors": "Item not found: 234501"
}

если книга есть и для нее есть название на языке:
http://aab.clouds.today/ru/book/501/
response:
{
    "id": 501,
    "Name": "София Универсальный объектно-ориентированный ядро ПАО CибКазаньБашкир"
}

http://aab.clouds.today/en/book/501/
response:
{
    "id": 501,
    "Name": "Culpa."
}
