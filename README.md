### Парсер общей библиотеки docx/postgres/manticore

Для запуска необходимы docker windows, git windows, postman, dbeaver

1. Установить Docker для Windows https://docs.docker.com/desktop/install/windows-install/

2. Установить git для windows https://git-scm.com/downloads При установке можно оставить все по умолчанию

3. Установить программу Postman https://www.postman.com/downloads/

4. Установить программу DBeaver Community, необходима для просмотра данные postgres БД в графическом интерфейсе
https://dbeaver.io/

После установки необходимых программ, нажмите меню пуск, найдите git консоль Git CMD, запустите

Выберите или создайте папку с проектами, например c:\Users<username>\projects,

Для создания наберите в консоли Git CMD`mkdir ./projects`

Выбрать созданную папку, набрать в консоли Git CMD: `cd ./projects`

Для клонируйте репозитория terratensor/common_library_parser, наберите в консоли Git CMD:

`git clone https://github.com/terratensor/common_library_parser.git`

После наберите в консоли Git CMD

`cd ./common_library_parser`

Далее запустите windows docker (через меню пуск)

После запуска необходимо проверить, что нет запущенных контейнеров с manticoresearch (меню containers), если есть остановить их или удалить, иначе будет конфликт доступа к портам.

В консоли Git CMD, где ранее создали папку и клонировали репозиторий для запуска контейнера с парсером common_library_parser набрать:

`docker-compose down --remove-orphans`

`docker compose up --build -d`

запустится база данных postgres и manticore search

- Будет создана локальная БД с именем common-library
- Имя пользователя app
- Пароль secret
- Порт для подключения к БД 54322

Эти настройки можно увидеть или изменить в файле docker-compose.yml
```
    environment:
      APP_ENV: dev
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: common-library
```

Запускаем book-parser-common.exe файл, папка по умолчанию, где должны быть размещены docx для парсинга — `process`, с помощью параметра `-o` можно указать путь нужный путь к папке, в конце пути обязательно поставить слэш:

`book-parser-pg.exe -o ./militera/mt/`

Будет произведена обработка docx файлов и запись их в таблицы БД:
```
books
    id, name, filename, created_at, updated_at, deleted_at

book_paragraphs
    id, book_id, text, position, length, created_at, updated_at, deleted_at 
```

Посмотреть данные в БД можно, например, с помощью программы DBeaver Community https://dbeaver.io/

--------------------------

**Остановка контейнеров book-parser-common**

`docker-compose down --remove-orphans`

**Запуск контейнеров book-parser-common из папки проекта** 

`docker compose up --build -d`

### Первый запуск manticore indexer, если еще не создана таблица common_library

```
docker exec -it book-parser-manticore indexer common_library
```

### Повторный запуск manticore indexer, если таблица существует, для переиндексации

```
docker exec -it book-parser-manticore indexer common_library --rotate
```

#### Бэкап postgres БД:
```
docker exec -it book-parser-postgres bash

pg_dump --dbname=book-parser --username=app --host=postgres-book-parser | gzip -9 > book-parser-backup-filename.gz
```

Скопировать файл бэкапа в контейнер докера для восстановления БД, распаковать из gz, запустить загрузку бэкапа в БД

```
cp book-parser-backup-filename.gz book-parser-postgres:app/book-parser-backup-filename.gz

gzip -d book-parser-backup-filename.gz

psql -U app -d lib < book-parser-backup-filename.sql
```

#### Остановка и удаление БД. ВНИМАНИЕ ДАННЫЕ БУДУТ УДАЛЕНЫ.

`docker compose down -v --remove-orphans`

### Опции командной строки

```
-h --help помощь
-b, --batchSize int   размер пакета по умолчанию (default batch size) (default 3000)
-o, --output string   путь хранения файлов для обработки (default "./process/")
```

##### Сборка бинарника, нужно для разработки:

`GOOS=windows GOARCH=amd64 go build -o ./book-parser-gorm.exe ./common/cmd/main.go`
