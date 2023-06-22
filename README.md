### Парсер общей библиотеки docx/postgres/manticore

Для запуска необходимы docker windows, git windows, postman, dbeaver

1. Установить Docker для Windows https://docs.docker.com/desktop/install/windows-install/

2. Установить git для windows https://git-scm.com/downloads При установке можно оставить все по умолчанию

3. Установить программу Postman https://www.postman.com/downloads/

4. Установить программу DBeaver Community, необходима для просмотра данные postgres БД в графическом интерфейсе
https://dbeaver.io/

После установки необходимых программ, нажмите меню пуск, найдите git консоль Git CMD, запустите

Выберите или создайте папку с проектами, например c:\projects,

Для создания наберите в консоли Git CMD 
```
mkdir ./projects
```

Выбрать созданную папку, набрать в консоли Git CMD: 
```
cd ./projects
```

Для клонирования репозитория terratensor/common_library_parser, наберите в консоли Git CMD:

```
git clone https://github.com/terratensor/common_library_parser.git
```

Увидите сообщение в консоли об успешном клонировании:
```
Cloning into 'common_library_parser'...
fatal: helper error (-1): User cancelled dialog.
Username for 'https://github.com':
c:\terratensor>git clone https://github.com/terratensor/common_library_parser.git
Cloning into 'common_library_parser'...
remote: Enumerating objects: 20, done.
remote: Counting objects: 100% (20/20), done.
remote: Compressing objects: 100% (15/15), done.
Receiving objects: 100% (20/20), 6.48 KiB | 6.48 MiB/s, done.
Resolving deltas: 100% (4/4), done.
```

После наберите в консоли Git CMD

```
cd ./common_library_parser
```

Затем наберите в консоли 
```
ls -la
```

Увидите следующую структуру папки:
```
total 32
drwxrwx---+ 1 username Domain Users    0 Jun 22 13:30 .
drwxrwx---+ 1 username Domain Users    0 Jun 22 13:30 ..
drwxrwx---+ 1 username Domain Users    0 Jun 22 13:30 .git
-rwxrwx---+ 1 username Domain Users 8880 Jun 22 13:30 README.md
drwxrwx---+ 1 username Domain Users    0 Jun 22 13:30 docker
-rwxrwx---+ 1 username Domain Users  957 Jun 22 13:30 docker-compose.yml
drwxrwx---+ 1 username Domain Users    0 Jun 22 13:30 process
```

Скачайте последнюю версию парсера book-parser-common.exe.
Ссылка на файл находится в секции Assets страницы описания релиза 

https://github.com/terratensor/book-parser/releases/latest

Сохраните book-parser-common.exe в папке с проектом ./common_library_parser

Далее запустите windows docker (через меню пуск)

После запуска необходимо проверить, что нет запущенных контейнеров с manticoresearch (меню containers), если есть остановить их или удалить, иначе будет конфликт доступа к портам.

В консоли Git CMD, где ранее создали папку и клонировали репозиторий для запуска контейнера с парсером common_library_parser набрать:

```
docker-compose down --remove-orphans
```

```
docker compose up --build -d
```

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

Скопируйте в папку process docx файлы для обработки (8480).

Папка по умолчанию, где должны быть размещены docx для обработки файлов — `process`, с помощью параметра `-o` можно указать любой другой путь к нужной папке, в конце пути обязательно поставить слэш

Запустите book-parser-common.exe, если файлы скопированы в папку process:

```
book-parser-common.exe
```


Или запустите book-parser-common.exe с параметром -o и укажите нужную папку, где расположены файлы для обработки, например, для другой своей папки `/militera/mt/`:

```
book-parser-common.exe -o ./militera/mt/
```

Будет произведена обработка docx файлов и запись их в таблицы БД:
```
db_pg_books
    id, name, filename, created_at, updated_at, deleted_at

db_pg_paragraphs
    id, book_id, book_name, text, position, length, created_at, updated_at, deleted_at 
```

При обработке файлов в консоли после каждой успешно обработанной книги будеп появляться сообщение с наименованием и номером файла:
```
2023/06/22 11:11:52 Яроцкий. На всех фронтах.docx #8469 done
2023/06/22 11:11:52 Ярошенко. Повторение пройденного.docx #8470 done
```
Иногда в процессе на экран могут вылетать большие «портянки» текста с параграфами - это лог медленного запроса, который выполнялся в БД более 200 мс,
не обращайте на это внимание, это информационное сообщение, книга и параграфы в этом случае все равно сохранены в БД.
```
book-parser/common/db/sql/pgGormStore/paragraph_store.go:83 ←[33mSLOW SQL >= 200ms
←[0m←[31;1m[293.917ms] ←[33m[rows:856]←[35m INSERT INTO "db_pg_paragraphs" ("book_id","book_name","text","position","length","created_at","updated_at","deleted_at") VALUES (6664,'Дёниц К. Десять лет и двадцать дней','<p>Вначале все шло по плану. 24 мая 1943 года «U-441» была атакована «Сандерлендом», который был довольно быстро сбит. Однако из-за сбоя в кормовой четрехствольной установке самолет успел сбросить бомбы. Поэтому подлодка «U-441» сама получила повреждения и была вынуждена вернуться на базу.</p>',3000,294,'2023-06-22 11:08:12.521','2023-06-22 11:08:12.521',NULL),(6664,'Дёниц К.
...
,(6664,'Дёниц К. Десять лет и двадцать дней','<p>H</p>',3855,8,'2023-06-22 11:08:12.521','2023-06-22 11:08:12.521',NULL) RETURNING "id"←[0m
```


Время обработки 8480 docx файлов в один поток примерно 20 минут.

После обработки всех файлов в консоли будет завершающее сообщение:

```
...
2023/06/22 11:11:53 all files done
```

Для запуска индексации данных из БД в мантикору запустите manticore indexer:

```
docker exec -it book-parser-manticore indexer common_library
```

Время обработки данных из БД (24 701 003 параграфов) примерно 8-10 минут.

После того как данный будут проиндексированы необходимо перезапустить контейнеры common_library_parser, сделать это можно командами:

```
docker compose down
```

```
docker compose up -d
```

Теперь можно запустить программу postman

В программе Postman сделать вкладку и в адресе указать:
```
POST localhost:9308/search
```
![image](https://github.com/audetv/book-parser/assets/129882753/aad0a4c2-e213-46ac-a615-4ac98a8a7f82)


Ниже выбрать Body, ниже raw, рядом JSON

В поле ниже скопировать запрос:

```
{
    "index": "common_library",
    "highlight": {
        "fields": [
            "text"
        ],
        "limit": 0,
        "no_match_size": 0
    },
    "max_matches": 353545,
    "limit": 50,
    "offset": 0,
    "query": {
        "bool": {
            "must": [
                {
                    "query_string": "концепция"
                }
            ]
        }
    }
```
В match_phrase, вместо "концепция", можно набрать любой поисковый запрос в кавычках, например "бахмут"

"limit" - кол-во записей на странице
"offset" - смещение от начала.

Если надо посмотреть следующие 100 записей, изменить offset, например:
```
"limit": 100
"offset": 100
```

--------------------------
### Дополнительные сведения:

**Остановка контейнеров book-parser-common**

```
docker-compose down --remove-orphans
```

**Запуск контейнеров book-parser-common из папки проекта** 

```
docker compose up --build -d
```

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
```

```
pg_dump --dbname=book-parser --username=app --host=postgres-book-parser | gzip -9 > book-parser-backup-filename.gz
```

Скопировать файл бэкапа в контейнер докера для восстановления БД, распаковать из gz, запустить загрузку бэкапа в БД

```
cp book-parser-backup-filename.gz book-parser-postgres:app/book-parser-backup-filename.gz
```
```
gzip -d book-parser-backup-filename.gz
```
```
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

```
GOOS=windows GOARCH=amd64 go build -o ./book-parser-gorm.exe ./common/cmd/main.go
```
