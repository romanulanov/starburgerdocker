# Сайт доставки еды Star Burger

Это сайт сети ресторанов Star Burger. Здесь можно заказать превосходные бургеры с доставкой на дом.

![скриншот сайта](https://dvmn.org/filer/canonical/1594651635/686/)

Сеть Star Burger объединяет несколько ресторанов, действующих под единой франшизой. У всех ресторанов одинаковое меню и одинаковые цены. Просто выберите блюдо из меню на сайте и укажите место доставки. Мы сами найдём ближайший к вам ресторан, всё приготовим и привезём.

На сайте есть три независимых интерфейса. Первый — это публичная часть, где можно выбрать блюда из меню, и быстро оформить заказ без регистрации и SMS.

Второй интерфейс предназначен для менеджера. Здесь происходит обработка заказов. Менеджер видит поступившие новые заказы и первым делом созванивается с клиентом, чтобы подтвердить заказ. После оператор выбирает ближайший ресторан и передаёт туда заказ на исполнение. Там всё приготовят и сами доставят еду клиенту.

Третий интерфейс — это админка. Преимущественно им пользуются программисты при разработке сайта. Также сюда заходит менеджер, чтобы обновить меню ресторанов Star Burger.


## Важное ! Файл `.env`
Для работы сайта необходимо создать файл `.env`, вида

- `DEBUG` — дебаг-режим. Поставьте `False`.
- `SECRET_KEY` — секретный ключ проекта. Он отвечает за шифрование на сайте. Например, им зашифрованы все пароли на вашем сайте.
- `YANDEX_API_KEY` — API ключ для подключения [геокодера](https://developer.tech.yandex.com/services)
- `ROLLBAR_TOKEN_KEY` — API ключ для контроля ошибок вашего сайта подробнее на [Rollbar](https://app.rollbar.com/)
- `ALLOWED_HOSTS` — [см. документацию Django](https://docs.djangoproject.com/en/3.1/ref/settings/#allowed-hosts)
- `DATABASE_URL` = "postgresql://starburger_db_user:тут_указать_свой_пароль@localhost/starburger"
и установить дополнение в своем окружении

```sh
SECRET_KEY=djang...f93n4
YANDEX_API_KEY=0ea7...75f45a1
ROLLBAR_TOKEN_KEY=218a...6e92
ENVIRONMENT="Production"
DATABASE_URL="postgresql://имя пользователя в БД:Пароль@Адрес сайта БД:Порт-5454/имя БД"
```

## Как запустить dev-версию сайта

Для запуска сайта нужно использовать команду, предварительно переименовав файл
`docker-compose.local-dev.yaml` в `docker-compose.yaml`
```sh
docker-compose -f docker-compose.yaml up -d
```
Сайт запускается локально по адресу http://0.0.0.0:8000

## Как запустить prod-версию сайта
Для запуска сайта нужно использовать команду:
```sh
docker-compose -f docker-compose.local-dev.yaml up -d --build
```
Сайт запускается по вашему адресу, который вы укажите в конфигурационном файле nginx
Важно! на сайте необходимо установить сертификаты для вашего доменного [имени](https://letsencrypt.org/ru/getting-started/)

## Просмотр логов работы докера
Для просмотра логов можно использовать команду:
```sh
docker-compose logs
```

## Скрипт обнобления
Непосредственно в каталоге данного репозитория находится скрипт, упрощающий обновление сайта
Для файла `deploy_star_burger.sh` необходимо сделать его исполняемым и запустить
```sh
chmod ugo+x deploy_star_burger.sh
```

Сервер NGINX будет запущен и сконфигурирован при запуске `docker-compose.prod.yaml`
```sh
upstream star_burger {server django-web:8000;}
server {
    listen 80;
    server_name ulanovroman.ru;
    charset utf8;
    autoindex off;
    location / {
      return 301 https://$host$request_uri;
    }
}
server {
    listen 443 ssl;
    server_name ulanovroman.ru;
    charset utf8;
    autoindex off;
    add_header Strict-Transport-Security "max-age=31536000";
    ssl_certificate /etc/letsencrypt/live/ulanovroman.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ulanovroman.ru/privkey.pem;
    ssl_ciphers TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-256-GCM-SHA384:ECDHE:!COMPLEMENTOFDEFA>
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    location / {
        proxy_pass http://star_burger;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_redirect off;
        location /media/ {
            alias /webapp/media/;
        }
        location /static/ {
            alias /webapp/static/;
        }
    }
    location /.well-known/acme-challenge/ {
    }
}
```

## Пример работы
Пример работы сайта можно посмотреть по [адресу](https://ulanovroman.ru.ru).


## Цели проекта
Код написан в учебных целях — это урок в курсе по Python и веб-разработке на сайте [Devman](https://dvmn.org). За основу был взят код проекта [FoodCart](https://github.com/Saibharath79/FoodCart).
