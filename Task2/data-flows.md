# DFD промежуточного состояния через 3 месяца

### Логика диаграммы
1. CRM двух банков продолжают создавать клиентские данные.
2. Integration Layer доставляет изменения в MDM.
3. MDM выполняет сопоставление, дедупликацию и обогащение и формирует единую запись клиента.
4. Корпоративный идентификатор возвращается в системы FinUnion и RetailBank.
5. Финансовые сущности считываются из профильных Core, Card и Loan систем.
6. Через Integration Layer данные проходят контроль и поступают в Unified DWH.
7. BI работает только с DWH, а не с операционными системами.
8. Регуляторная отчётность формируется из DWH.
9. DataHub регистрирует метаданные, владельцев и происхождение данных.

Удаление (Delete) для финансовых фактов в потоках не используется: потребители не должны удалять или исправлять мастер-записи в обход профильной системы.

| Источник | Приёмник | Сущности | Действие | Режим | Зачем |
| --- | --- | --- | --- | --- | --- |
| FU CRM | MDM | Клиент | Create / Update / Validate | Async | Создание единого профиля |
| RB CRM | MDM | Клиент | Create / Update / Validate | Async | Сопоставление клиентов RetailBank |
| MDM | FU CRM, RB CRM | Клиент | Read / Update | Async | Распространение корпоративного ID и золотой записи |
| FU Core | DWH | Счёт, договор, платёж, транзакция  | Read / Validate | CDC / batch | Единая отчётность без аналитической нагрузки на Core |
| RB Core | DWH | Счёт, договор, платёж, транзакция  | Read / Validate | CDC / batch | Включение портфеля RetailBank в общую отчётность |
| FU Cards | DWH | Карта, транзакция | Read / Validate | CDC / batch | Карточная аналитика |
| RB Cards | DWH | Карта, транзакция | Read / Validate | CDC / batch | Объединение карточных данных |
| FU Loans | DWH | Кредит, договор | Read / Validate | Batch | Общий кредитный портфель |
| RB Loans | DWH | Кредит, договор | Read / Validate | Batch | Общий кредитный портфель |
| MDM | DWH | Клиент, продукт, справочник | Read / Enrich | Batch / CDC | Использование согласованных измерений |
| DWH | BI | Клиенты, счета, продукты, операции | Aggregate / Read | Batch | Единые управленческие витрины |
| DWH | Регулятор | Регламентированные данные | Aggregate / Read | Batch | Регуляторная отчётность |
| MDM / Integration / DWH | DataHub | Метаданные | Create / Update | Async / batch | Lineage, каталог, владельцы |

CDC используется там, где требуется передавать изменения из транзакционных БД без выполнения тяжёлой аналитики непосредственно на них. Batch подходит для периодической отчётности и массивных загрузок.

### Код диаграммы

```plantuml
@startuml
title FinUnion + RetailBank — Data Flow Diagram, состояние через 3 месяца

left to right direction

skinparam shadowing false
skinparam defaultFontName Arial
skinparam defaultFontSize 14
skinparam nodesep 65
skinparam ranksep 80

skinparam note {
    BackgroundColor transparent
    BorderColor transparent
}

rectangle "FinUnion" {
    component "FU CRM" as FU_CRM
    component "FU Core Banking" as FU_Core
    component "FU Card Processing" as FU_Cards
    component "FU Loan System" as FU_Loans
}

rectangle "RetailBank" {
    component "RB CRM" as RB_CRM
    component "RB Core Banking" as RB_Core
    component "RB Card Processing" as RB_Cards
    component "RB Loan System" as RB_Loans
}

rectangle "Промежуточный контур данных" {
    component "Integration Layer" as Integration
    database "MDM Customer Hub" as MDM
    database "Unified DWH" as DWH
    component "BI" as BI
    component "DataHub\nData Catalog" as DataHub
}

actor "Регулятор" as Regulator

FU_CRM --> Integration
note on link
Create / Update
Клиент, обращение
end note

RB_CRM --> Integration
note on link
Create / Update
Клиент, обращение
end note

Integration --> MDM
note on link
Validate / Enrich
Сопоставление клиентов
end note

MDM --> Integration
note on link
Read
Golden Customer
corporate_customer_id
end note

Integration --> FU_CRM
note on link
Update
Единый ID клиента
end note

Integration --> RB_CRM
note on link
Update
Единый ID клиента
end note


FU_Core --> Integration
note on link
Read
Счета, договоры,
операции, платежи
end note

RB_Core --> Integration
note on link
Read
Счета, договоры,
операции, платежи
end note

FU_Cards --> Integration
note on link
Read
Карты, карточные операции
end note

RB_Cards --> Integration
note on link
Read
Карты, карточные операции
end note

FU_Loans --> Integration
note on link
Read
Кредиты, договоры
end note

RB_Loans --> Integration
note on link
Read
Кредиты, договоры
end note


Integration --> DWH
note on link
Validate / Aggregate
Клиенты, счета,
продукты, операции
end note

DWH --> BI
note on link
Read / Aggregate
Единые витрины
end note

DWH --> Regulator
note on link
Read
Регуляторная отчётность
end note


MDM --> DataHub
note on link
Read
Метаданные,
владелец, lineage
end note

Integration --> DataHub
note on link
Read
Метаданные потоков
end note

DWH --> DataHub
note on link
Read
Метаданные витрин
end note

@enduml
```


