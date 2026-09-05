# Пайплайны данных

| Пайплайн | Источник | Приёмник | Тип | Инструменты |
| --- | --- | --- | --- | --- |
| Единая отчётность — 3 месяца | FU/RB CRM, Core, Cards, Loans, MDM | PostgreSQL DWH | ETL, batch | Airflow, SQL |
| Клиентские мастер-данные | CRM, клиентские каналы | MDM → DWH/Lakehouse | ETL / CDC | Integration Layer, Kafka/CDC, Airflow |
| Финансовые операции | Core Banking | DWH + Lakehouse | CDC | Kafka/CDC |
| Карточные события | Card Processing | Data Lakehouse | Streaming | Kafka, S3, Iceberg |
| Кредитный портфель | Loan System | DWH + Lakehouse | Batch / CDC | Airflow, Kafka/CDC |
| Data Products | DWH / Lakehouse | Доменные наборы | ELT | Airflow, SQL, Trino |

Airflow используется именно как оркестратор: он запускает задачи, управляет зависимостями, ретраями и повторной обработкой, но сам не является движком обработки данных.

### Пайплайн промежуточного DWH

FU/RB Sources > Extract > Staging > Validate / Mapping / Deduplicate > Core > Data Marts > BI

Для PostgreSQL DWH используется ETL: данные проходят преобразование перед публикацией в согласованные слои. Для Lakehouse допустим ELT — исходные данные сначала сохраняются, затем преобразуются уже внутри аналитического контура.

### Сценарии больших данных

#### Сценарий 1. История карточных операций

| Параметр | Решение |
| ---| --- |
| Источник | Card Processing |
| Характер | Большая история + событийные данные |
| Целевое хранилище | Data Lakehouse |
| Способ обработки | Streaming / CDC + ELT |
| Потребители | Антифрод, риск-менеджмент, аналитика |

Поток:
Card Processing > Kafka / CDC > S3 (Bronze) > Iceberg (Silver/Gold)> Trino > Risk / Antifraud / BI

#### Сценарий 2. Исторические данные RetailBank

| Параметр | Решение |
| --- | --- |
| Источник | RB DWH, RB Core, RB Cards, RB Loans, архивные выгрузки |
| Характер | Исторические, архивные, частично полуструктурированные |
| Целевое хранилище | S3 + Data Lakehouse |
| Способ обработки | Batch / ELT |
| Потребители | BI, риск-менеджмент, аналитика |

Сначала данные сохраняются без потери исходной информации в Bronze, затем очищаются и сопоставляются с корпоративными идентификаторами в Silver, а согласованные наборы публикуются в Gold. 
