# Аналитическая модель

Выбирается схема **'Звезда'**.

### Гранулярность

Одна запись fact_transactions = одна финансовая транзакция

| Тип | Таблица | Основные данные |
| --- | --- | --- |
| Fact | fact_transactions | transaction_id, amount, transaction_datetime, customer_key, account_key, product_key, channel_key, currency_key, date_key |
| Dimension | dim_customer | corporate_customer_id, segment, customer_type |
| Dimension | dim_account | account_id, account_type, status |
| Dimension | dim_product | product_id, product_name, product_type |
| Dimension | dim_channel | channel_name, channel_type |
| Dimension | dim_currency | currency_code, currency_name |
| Dimension | dim_date | day, month, quarter, year |

### Схема

```plantuml
@startuml

entity "fact_transactions" as fact {
    * transaction_key
    --
    transaction_id
    transaction_datetime
    customer_key
    account_key
    product_key
    channel_key
    currency_key
    date_key
    amount
}

entity "dim_customer" as customer {
    * customer_key
    --
    corporate_customer_id
    customer_type
    segment
}

entity "dim_account" as account {
    * account_key
    --
    account_id
    account_type
    status
}

entity "dim_product" as product {
    * product_key
    --
    product_id
    product_name
    product_type
}

entity "dim_channel" as channel {
    * channel_key
    --
    channel_name
    channel_type
}

entity "dim_currency" as currency {
    * currency_key
    --
    currency_code
    currency_name
}

entity "dim_date" as date {
    * date_key
    --
    day
    month
    quarter
    year
}

customer ||--o{ fact
account ||--o{ fact
product ||--o{ fact
channel ||--o{ fact
currency ||--o{ fact
date ||--o{ fact

@enduml
```

### Почему подходит для OLAP-нагрузки

Для BI требуется анализировать транзакции:
- по клиенту и сегменту
- по счёту
- по продукту
- по каналу
- по валюте
- по периоду

'Звезда' уменьшает количество связей в пользовательских запросах и делает модель понятнее для BI. Она проще для потребителей и снижает вероятность ошибок при построении отчётов.
