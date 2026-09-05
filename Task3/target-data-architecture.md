# Целевая архитектура данных

### Архитектурный подход

В качестве целевой выбрана гибридная архитектура:
- Operational Systems: выполнение банковских операций и хранение актуального операционного состояния
- MDM: золотая запись клиента, единые идентификаторы и справочники
- Integration Layer: синхронные и асинхронные интеграции
- Kafka + CDC: доставка событий и изменений
- PostgreSQL DWH: согласованные структурированные данные, регламентированная и управленческая отчётность
- S3 + Apache Iceberg: Data Lakehouse для большой истории и событий
- Trino: SQL доступ к Lakehouse
- Airflow: оркестрация batch ETL/ELT
- BI: отчёты, дашборды и аналитика
- DataHub: каталог, владельцы, метаданные и lineage
- Data Products: публикация согласованных данных по бизнес доменам

Такое разделение соответствует принципу: операционный контур оптимизирован для проведения операций, а аналитический — для массового чтения, агрегаций и исторического анализа.

### Диаграмма To-Be

```plantuml
@startuml
title Target Data Architecture (12 месяцев)

left to right direction

package "Операционный контур" {
    rectangle "CRM" as CRM
    rectangle "Core Banking" as Core
    rectangle "Card Processing" as Cards
    rectangle "Loan System" as Loans
    rectangle "MDM\nCustomer Hub" as MDM
}

package "Интеграция и обработка" {
    rectangle "Integration Layer" as Integration
    queue "Kafka + CDC" as Kafka
    rectangle "Apache Airflow" as Airflow
    rectangle "Data Quality" as DQ
}

package "Платформа данных" {
    database "PostgreSQL DWH" as DWH

    package "Data Lakehouse" {
        cloud "S3" as S3
        rectangle "Apache Iceberg" as Iceberg
        rectangle "Trino" as Trino

        S3 --> Iceberg
        Iceberg --> Trino
    }

    rectangle "DataHub\nCatalog + Lineage" as DataHub
}

package "Data Products" {
    rectangle "Customer 360" as Customer360
    rectangle "Accounts & Payments" as PaymentsDP
    rectangle "Cards & Transactions" as CardsDP
    rectangle "Credit Portfolio" as CreditDP
}

package "Потребители" {
    rectangle "BI" as BI
    rectangle "Risk / Antifraud" as Risk
    rectangle "Регуляторная\nотчётность" as Regulatory
}

CRM --> Integration
Core --> Integration
Cards --> Integration
Loans --> Integration
MDM <--> Integration

Core --> Kafka
Cards --> Kafka
CRM --> Kafka
Loans --> Kafka

Integration --> Airflow
MDM --> Airflow

Airflow --> DQ
Kafka --> DQ

DQ --> DWH
DQ --> S3

DWH --> Customer360
DWH --> PaymentsDP
Trino --> CardsDP
Trino --> CreditDP

Customer360 --> BI
PaymentsDP --> BI
CardsDP --> BI
CreditDP --> BI

DWH --> Regulatory
Trino --> Risk

DWH ..> DataHub : metadata / lineage
Iceberg ..> DataHub : metadata / lineage
Customer360 ..> DataHub
PaymentsDP ..> DataHub
CardsDP ..> DataHub
CreditDP ..> DataHub
@enduml
```
