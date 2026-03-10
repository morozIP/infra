# BigData «Lakehouse» demo

Этот каталог содержит небольшой учебный проект, в котором на базе
Docker‑compose разворачивается стек для **Data Lakehouse** (Iceberg +
Nessie + MinIO) и простейший Airflow‑pipeline, загружающий CSV‑данные
через Dremio.

## Структура

```
bigData/
├── docker-compose.yml          # весь стек: Nessie, MinIO, mc, Dremio, Airflow, Postgres
├── data/
│   └── sample.csv              # исходный CSV
└── airflow/
    ├── Dockerfile              # кастомный образ airflow
    ├── requirements.txt        # psycopg2-binary & dremio-simple-query
    └── dags/
        ├── dag_lakehouse.py    # единственный DAG
        └── sql/
            └── insert_queries.sql  # создаётся во время выполнения
```

## Что запускается

* **MinIO** — S3‑совместимое хранилище; `mc` одномоментно создаёт бакет
  `warehouse`.
* **Nessie** — каталог метаданных для Iceberg.
* **Dremio** — портал/движок SQL, подключён к каталогу и хранилищу.
* **Airflow** — оркестратор: webserver, scheduler, init‑контейнер.
* **Postgres** — база Airflow.
* **mc** — контейнер‑клиент для настройки MinIO.

## Как проверить

1. **Собрать/запустить**:

   ```bash
   docker-compose build airflow-webserver
   docker-compose up -d
   ```

2. Убедиться, что сервисы доступы:

   * MinIO → http://localhost:9000  (`admin/password`)
   * Nessie → http://localhost:19120
   * Dremio → http://localhost:9047  (`admin/admin` по умолчанию)
   * Airflow → http://localhost:8080  (`admin/admin`)

3. В Dremio добавить источник **Nessie** (URL `http://catalog:19120/api/v2`)
   с параметрами S3 (endpoint `storage:9000`, root `warehouse`,
   `admin`/`password`, `fs.s3a.path.style.access=true`,
   `dremio.s3.compat=true`).

4. В Airflow UI запустить DAG `dremio_query_dag`.  
   *Create table → generate queries → insert queries*.

5. Проверить результат:

   * в Dremio: `SELECT * FROM catalog.orders` (5 строк);
   * в MinIO: папка `warehouse/orders_*` с `.parquet` и `metadata/…`;
   * в Nessie UI: таблица в ветке `main` (можно создать/мерджить ветки).

6. Для повторного теста изменить `data/sample.csv` и перезапустить DAG.

7. Очистка:

   ```bash
   docker-compose down -v
   rm -rf airflow/__pycache__ airflow/dags/sql/insert_queries.sql
   ```

## Примечания

* Код DAG‑а можно параметризовать через переменные/Connections Airflow.
* В `docker-compose.yml` уже есть healthchecks для Postgres/Airflow;
  при желании добавьте их и для Dremio/MinIO и используйте
  `depends_on.condition: service_healthy`.
* Пустые строки в CSV вызывают ошибку; либо используйте `olaptests/data`
  версию без пустых строк, либо добавьте проверку в DAG.

---

Этот набор файлов — готовый пример pipeline‑а: из CSV → Airflow/Dremio →
Nessie/Iceberg → S3‑совместимое хранилище. Используйте как шаблон для
экспериментов с “данными как код” и ветвлением в Nessie.
