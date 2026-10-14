# Диаграммы

## Диаграмма последовательности

```mermaid
sequenceDiagram
    actor User as Пользователь / Почта
    participant System as Сервис фильтрации
    participant DB as БД (Белый список)
    participant Model as ML-модель

    User->>System: Поступило входящее письмо
    System->>DB: Проверить адрес отправителя в белом списке
    
    alt Адрес найден в белом списке (Альтернатива 2а)
        DB-->>System: Подтверждение наличия в белом списке
        System-->>User: Доставить письмо во «Входящие»
    else Адрес отсутствует в белом списке
        DB-->>System: Адрес не найден
        System->>Model: Передать текст сообщения для анализа
        
        alt ML-модель доступна (Основной сценарий)
            Model-->>System: Вернуть вердикт (Спам / Не спам)
            
            alt Письмо признано спамом
                System->>User: Переместить в «Корзину» и отправить уведомление
            else Письмо признано легитимным
                System-->>User: Доставить письмо во «Входящие»
            end
            
        else Исключение: ML-модель недоступна
            Model--xSystem: Ошибка соединения / Тайм-аут
            Note over System: Записать ошибку в лог <br/>(перестраховка)
            System-->>User: Доставить письмо во «Входящие»
        end
    end
```

## Диаграмма классов

```mermaid
classDiagram
    direction LR

    class User {
        +int id
        +string email
        +string notificationChannel
        +getProfile()
    }

    class EmailMessage {
        +int id
        +string sender
        +string subject
        +string body
        +string status
        +moveToTrash()
        +restoreToInbox()
    }

    class WhitelistEntry {
        +int id
        +string emailAddress
        +datetime createdAt
        +addAddress()
        +removeAddress()
    }

    class SpamClassification {
        +int id
        +float confidenceScore
        +string verdict
        +analyzeText()
    }

    User "1" --> "*" EmailMessage : владеет
    User "1" --> "*" WhitelistEntry : настраивает
    EmailMessage "1" --> "1" SpamClassification : анализируется
```