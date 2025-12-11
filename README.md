# Data Platform Sandbox: Airflow + MinIO + Medallion Architecture

Этот репозиторий содержит учебно-тестовую инфраструктуру для реализации пайплайнов данных, построения медальонной архитектуры (staging → silver → gold), а также для последующей интеграции инструментов Data Lineage.

Архитектура построена на следующих компонентах:

- **Apache Airflow** — оркестратор задач (ETL/ELT).
- **MinIO** — S3-совместимое объектное хранилище для слоёв данных.
- **PostgreSQL** — метахранилище Airflow.
- **Docker Compose** — инфраструктурный рантайм для локального и облачного стендов.
- **Yandex Cloud VM** — целевая среда деплоя.
- **GitHub Actions** — CI/CD для доставки DAG-ов и конфигурации.

---

## 1. Архитектура

### 1.1. Общая схема

GitHub → GitHub Actions → Yandex Cloud VM → Docker Compose
↘→ Airflow Scheduler
↘→ MinIO (staging/silver/gold)


### 1.2. Компоненты

#### Airflow
Используется для оркестрации ETL/ELT-пайплайнов.  
В директории `airflow/` находятся:
- `dags/` — DAG-и с задачами
- `plugins/` — кастомные операторы
- `requirements.txt` — зависимости Airflow

#### MinIO
Хранилище для трёх слоёв данных:
- **Staging** — сырые данные  
- **Silver** — очищенные, нормализованные  
- **Gold** — аналитические витрины  

Работает в Docker и полностью повторяет логику AWS S3.

#### Docker Compose
Файл `docker-compose.yaml` поднимает:
- airflow-webserver  
- airflow-scheduler  
- airflow-worker (если включён)  
- postgres  
- minio  
- minio-console  

#### Yandex Cloud VM
Продовый аналог локального стенда.  
ВМ рекомендуется конфигурации:
- 4 vCPU  
- 8 GB RAM  
- 50–100 GB NVMe  
На ней крутится тот же `docker-compose.yaml`, что и локально.

---

## 2. Структура репозитория

repo/
├── airflow/
│ ├── dags/
│ ├── plugins/
│ ├── requirements.txt
│ ├── Dockerfile
│ └── README.md
├── infrastructure/
│ ├── terraform/ (optional)
│ └── ansible/ (optional)
├── minio/
│ ├── config/
│ └── README.md
├── docker-compose.yaml
├── .gitignore
└── README.md


---

## 3. Деплой в Yandex Cloud

1. Создаётся виртуальная машина (Ubuntu 22.04).  
2. Устанавливаются:
   - docker
   - docker-compose
   - ssh-доступ и ключи
3. Репозиторий клонируется командой:

git clone https://github.com/<your_username>/<your_repo>.git

4. Поднимаются сервисы:

docker compose up -d

---

## 4. CI/CD с GitHub Actions

CI/CD реализует:
- автоматическую доставку DAG-ов на VM
- автоматическое обновление Airflow Scheduler

Процесс:
1. При `git push` в ветку `main`  
2. GitHub Actions запускает workflow  
3. DAG-и копируются через SSH/SCP на VM  
4. Scheduler перезапускается

Workflow лежит в:

.github/workflows/deploy-dags.yml

---

## 5. Как добавить новый DAG

1. Создай файл в `airflow/dags/`  
2. Закоммить изменения  
3. Выполни `git push`  
4. GitHub Actions автоматически доставит DAG на сервер
