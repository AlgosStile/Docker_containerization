# Введение в Docker
## Задание:
## 1. запустить контейнер с БД, отличной от mariaDB, используя инструкции на сайте: https://hub.docker.com/
## 2. добавить в контейнер hostname такой же, как hostname системы через переменную
## 3. заполнить БД данными через консоль
## 4. запустить phpmyadmin (в контейнере) и через веб проверить, что все введенные данные доступны

### Решение:

Для выполнения задания понадобится установленный Docker и доступ к интернету для загрузки образов.

Запуск контейнера с БД Postgres:
docker run --name some-postgres -e POSTGRES_PASSWORD=test123 -d postgres

где:

--name - имя контейнера
-e POSTGRES_PASSWORD - переменная окружения с паролем для пользователя по умолчанию postgres
-d postgres - имя образа, используемого для запуска контейнера
Добавление hostname в контейнер:
docker exec some-postgres bash -c 'echo "127.0.0.1 $(hostname)" >> /etc/hosts'
где:

some-postgres - имя запущенного контейнера
Заполнение БД данными через консоль (SQL):
docker exec -i some-postgres psql -U postgres << EOF
CREATE DATABASE testdb;
\connect testdb
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(50) NOT NULL UNIQUE
);
INSERT INTO users (name, email) VALUES ('John Doe', 'johndoe@example.com'), ('Jane Doe', 'janedoe@example.com');
EOF
где:

some-postgres - имя запущенного контейнера
-i - флаг для передачи SQL-запросов вводом
psql - команда для подключения к PostgreSQL
-U postgres - имя пользователя базы данных, здесь используется пользователь по умолчанию postgres
<< EOF - начало передачи SQL-запросов
\connect testdb - переключение на базу данных testdb
CREATE TABLE - создание таблицы users с полями id, name и email
INSERT INTO - добавление строк в таблицу users
Запуск pgAdmin в контейнере:
docker run --name some-pgadmin -p 5050:80 -e PGADMIN_DEFAULT_EMAIL=test@test.com -e PGADMIN_DEFAULT_PASSWORD=test123 -d dpage/pgadmin4
где:

--name - имя контейнера
-p 5050:80 - проброс портов, где 5050 - порт хоста, 80 - порт контейнера
-e PGADMIN_DEFAULT_EMAIL - переменная окружения для указания email при первом запуске pgAdmin
-e PGADMIN_DEFAULT_PASSWORD - переменная окружения для указания пароля при первом запуске pgAdmin
-d dpage/pgadmin4 - имя образа, используемого для запуска контейнера

Подключение к БД через pgAdmin:
Откройте браузер и перейдите по адресу http://localhost:5050/
Введите логин (email) и пароль, указанные при запуске контейнера pgAdmin
Далее, создайте новое подключение, нажав на кнопку "Add New Server" в окне "Quick Links" или на панели меню "Object".
В открывшемся окне "Create - Server" введите следующие данные:
Name - название вашего подключения (может быть любым)
Hostname/address - имя хоста или IP-адрес контейнера с БД, в данном случае - имя хоста контейнера some-postgres
Port - порт, на котором запущен PostgreSQL, по умолчанию - 5432
Maintenance database - имя базы данных, в данном случае - testdb
Username - имя пользователя базы данных, в данном случае - postgres
Password - пароль для пользователя базы данных, в данном случае - test123
Нажмите на кнопку "Save" для сохранения настроек подключения.
Выберите ваше подключение из списка серверов в окне "Object".
Чтобы проверить доступность данных, выполните запрос SQL, например, нажмите правой кнопкой мыши на таблице "users", выберите "Query Tool" и выполните запрос "SELECT * FROM users;".