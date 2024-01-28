# Руководство подключению к API 

## Навигация
1. [Введение](#introduction)
2. [Тестирование](#testing)
3. [HTTP-коды возврата](#httpcodes)
4. [Авторизация](#auth)
5. [Аккаунт](#account)<br/>
5.1 [Получение баланса по всем валютам](#balance_all)<br/>
5.2 [Функция получения баланса по конкретной валюте](#balance_token)
6. [Криптовалюта](#crypto)<br/>
6.1 [Сгенерировать депозитный адрес](#generate_deposit)<br/>
6.2 [Получить историю пополнений криптовалюты](#history_deposit)<br/>
7. [Код BERIBIT](#codes)<br/>
7.1 [Перевод внутри платформы](#transfer_code)<br/>
7.2 [Применить код](#deposit_code)<br/>
8. [Торговые операции](#trade)<br/>
8.1 [Получить рынки](#trade_markets)<br/>
---

## <a id="introduction">Введение</a>

---

### Для получение API-ключей необходимо:
1. Быть зарегистрированным на <a href="https://beribit.com/register" target="_blank">beribit.com</a>
2. В <a href="https://beribit.com/profile">настройках профиля</a> установить Google Authenticator
3. Cвязаться с <a href="https://t.me/beribitbot" target="_blank">технической поддержкой</a> и попросить выдать доступ

---

### Основная информация
- Базовый адрес: https://api.beribit.com
- API-ключи **чувствительны к регистру**.
- Все поля связанные со временем, указываются с нулевым смещением (UTC+0) в формате _YYYY-MM-DD'T'hh:mm:ss_ <br>
>**Если время сейчас 2023-09-14 15:44:31 (UTC+3), то в запросе оно передается в следующем формате:**
>```
> 2023-09-14T12:44:31
>```

---

## <a id="testing">Тестирование</a>

> Версия API на тестовом сервере может быть более новой и в некоторых моментах отличаться от описанной в документации.
> 
Для тестирования работы с API можно использовать тестовый сервер, https://test.beribit.com. Тестовый сервер во всём аналогичен рабочему, кроме исполнения депозитов/выводов, и использует 
копию базы с продакшна. Если для тестов необходимы дополнительные лимиты на тестовом сервера, они могут быть получены через техническую поддержку.

---

## <a id="httpcodes">HTTP-коды возврата</a>
|Код возврата| Описание |
|--|--|
| 200 | Операция выполнена успешно, возвращается объект JSON |
| 4XX | Используются для некорректно сформированных запросов; проблема на стороне отправителя |
| 401 | Используется при ошибке авторизации |
| 408 | Используется если превышен интервал запроса |
| 5XX | Используются для внутренних ошибок, проблема на стороне BERIBIT. **Важно НЕ рассматривать это как неудачную операцию; состояние выполнения НЕИЗВЕСТНО и могло быть выполнено успешно** |

---


## <a id="auth">Авторизация</a>
<b>**Обязательно:** Для каждого запроса ВНЕ ЗАВИСИМОСТИ ОТ ТИПА (GET/POST/DELETE) добавляется штамп времени (текущее время в формате UTC), передается в параметре запроса timestamp.</b>
>```
> GET https://api.beribit.com/accounts?timestamp=2023-08-20T13:51:00
>```
>```
> POST https://api.beribit.com/deposit/generate_address?timestamp=2023-08-20T13:51:00
>```

### Для авторизации необходимо поместить в следущие заголовоки запроса:
|Заголовок| Содержание |
|--|--|
| UID | **Персональный ключ (находится в API-ключах)** |
| SIGNATURE | **HMACSHA256-хеш (поле расчитываемое на каждый запрос)** |

---

### Примеры получения SIGNATURE
На ключах показаных в данном разделе можно проверить правильность генерации подписи в вашем приложении.

---
#### GET ЗАПРОС 
```
GET https://api.beribit.com/deposit/history?Timestamp=2023-08-20T13:51:00&Limit=10
```
|Параметр| Описание | Пример|
|--|--|--|
|PRIVATE_KEY|Приватный ключ HMACSHA256|ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==|
|QUERYSTRING|Все Query-параметры из HTTP запроса |?Timestamp=2023-08-20T13:51:00&Limit=10|

> Подписью является HMACSHA256(QUERYSTRING), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для GET-запроса:

    ?Timestamp=2023-08-20T13:51:00&Limit=10

Полученая подпись которую необходимо поместить в заголовок SIGNATURE

    45d8011a090e13502bcc1397650119ea4f37d369b3c9cdd64af2e92dbd493ad7

---
#### POST ЗАПРОС 
```
POST https://api.beribit.com/orders?Timestamp=2023-08-20T13:51:00
{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }
```
|Параметр| Описание | Пример|
|--|--|--|
|PRIVATE_KEY|Приватный ключ|ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==|
|PARAMS|Все Query-параметры из HTTP запроса |?Timestamp=2023-08-20T13:51:00|
|PAYLOAD|Тело запроса|{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }|

Подписью является HMACSHA256(PARAMS + ":" + PAYLOAD), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для POST-запроса:

    ?Timestamp=2023-08-20T13:51:00:{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }

Полученая подпись которую необходимо поместить в заголовок SIGNATURE

    15786f9f487c2ed8bcc6ddbe4f107f9d8dde0b26179e35de94b21665706637ed

---

## <a id="account">Аккаунт</a>

### <a id="balance_all">Получение баланса по всем валютам</a>

Авторизация: **Требуется**

Тип запроса: **GET**

Путь: **/accounts**

Описание: **данная функция отвечает за возвращение списка криптовалютных балансов.**

Результат успешного выполнения: **возвращается название криптовалюты, сумма на открытом счету, сумма на закрытом счету и таймштамп.**

Пример запроса:
```
curl --location 'https://api.beribit.com/accounts?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{
  "Success": true,
  "Result": [
    {
      "CurrencyLabel": "RUB",
      "Balance": 1005778.652637,
      "Locked": 0,
      "Timestamp": 1706283222
    },
    {
      "CurrencyLabel": "BTC",
      "Balance": 13.6344413325,
      "Locked": 0,
      "Timestamp": 1706283222
    },
    {
      "CurrencyLabel": "ETH",
      "Balance": 1,
      "Locked": 0,
      "Timestamp": 1706283222
    },
    {
      "CurrencyLabel": "USDT",
      "Balance": 8743.061909158,
      "Locked": 1,
      "Timestamp": 1706283222
    },
    {
      "CurrencyLabel": "BNB",
      "Balance": 999.959595,
      "Locked": 0,
      "Timestamp": 1706283222
    },
    {
      "CurrencyLabel": "TRX",
      "Balance": 884.9998251,
      "Locked": 0,
      "Timestamp": 1706283222
    }
  ],
  "Time": "2024-01-26T15:33:41.0658698Z"
}
```

Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

---

### <a id="balance_token">Функция получения баланса по конкретной валюте</a>

Авторизация: **Требуется**

Тип запроса: **GET**

Путь: **/account/{currency}**

Описание: **данная функция отвечает за возвращение баланса заданной валюты.**

Результат успешного выполнения: **возвращается сумма на открытом счету, сумма на закрытом счету и таймштамп.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Currency | string | обязательный | валюта, баланс которой требуется получить |

Пример запроса:
```
curl --location 'https://api.beribit.com/account/USDT?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{{
    "Success": true,
    "Result": {
        "Balance": 8743.061909158,
        "Locked": 1,
        "Timestamp": 1706284266
    },
    "Time": "2024-01-26T15:33:41.3607729Z"
}
```

Пример ошибки (HTTP Code 400):
```json
{
    "Success": false,
    "Error": {
        "Message": "Invalid currency"
    },
    "Time": "2024-01-26T15:33:41.1856961Z"
}
```
Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

---


## <a id="crypto">Криптовалюта</a>

---

### <a id="generate_deposit">Сгенерировать депозитный адрес</a>

> ! ВНИМАНИЕ ! НА ДАННЫЙ МОМЕНТ ЗАПРОС РАБОТАЕТ ТОЛЬКО ДЛЯ СЕТИ **TRC20** ! 

Авторизация: **Требуется**

Тип запроса: **POST**

Путь: **deposit/generate_address**

Описание: **генерация адреса для депозита**

Результат успешного выполнения: **возвращается json с идентификатором адреса, криптовалютным адресом и датой выполнения данной функции.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Blockchain | string | обязательный | название сети |

Пример запроса:
```
curl --location 'https://api.beribit.com/deposit/generate_address?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid_insert_here' \
--header 'Signature: signature-here' \
--header 'Content-Type: application/json' \
--data '{
    "Blockchain": "TRC20"
}'
```

Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": {
        "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
        "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
        "Time": "2023-09-15T09:49:11.8341287Z"
    }
}
```

Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

---

### <a id="history_deposit">Получить историю пополнений криптовалюты</a>

! ВАЖНО: НА ДАННЫЙ МОМЕНТ ЗАПРОС РАБОТАЕТ ТОЛЬКО ДЛЯ СЕТИ **TRC20** ! 

Авторизация: **Требуется**

Тип запроса: **GET**

Путь: **deposit/history**

Описание: **данная функция отвечает за получение истории пополнения, также можно задать опциональные параметры для получения информации за конкретные периоды**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе информацию: об адресе, об идентификаторе адреса, о хеше транзакции, о сети, о названии валюты, о сумме, о статусе и о времени**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| AddressId | string | обязательный | - | идентификатор адреса |
| Blockchain | string | обязательный | - | название сети |
| Limit | int? | опциональный | 100 | количество выгружаемых операций |
| Offset | int? | опциональный | 0 | параметр, отвечающий за пропуск некоторого числа первых сделок |
| FromDate | DateTime? | опциональный | DateTime.MinValue | нижний лимит выборки по времени создания операции |
| ToDate | DateTime? | опциональный | DateTime.MaxValue | верхний лимит выборки по времени создания операции |

Статусы заявки:

- Pending (ожидание исполнения операции);
- Executed (операция исполнена);
- Cancelled (операция отменена).

Пример запроса:
```json
curl --location 'https://api.beribit.com/deposit/history?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid_insert_here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": [
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": "6d58e075ff11c423a533a0b986238a36e60a41ef7716ef8393384f3b955e5a04",
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 12950.59,
            "Status": "Executed",
            "Time": "2023-09-15T09:49:40.3404432Z"
        },
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": "431498142770f180c5f1b808c9c070656461ea766496da8321507100b35a5776",
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 501.42,
            "Status": "Pending",
            "Time": "2023-09-15T09:49:40.3404895Z"
        },
        {
            "Address": "TMTwMMhmZKz6Ay1TnzTMdDzAxDV5H66666",
            "AddressId": "6164815f-2440-408c-a613-d8a839cab2d5",
            "Txid": null,
            "Blockchain": "TRC20",
            "Currency": "USDT",
            "Amount": 1792.64,
            "Status": "Cancelled",
            "Time": "2023-09-15T09:49:40.3404899Z"
        }
    ]
}
```

Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

---

## <a id="codes">Код BERIBIT</a>

---

### <a id="transfer_code">Перевод внутри платформы</a>

Авторизация: **Требуется**

Тип запроса: **POST**

Путь: **code/withdraw**

Описание: **данная функция отвечает за перевод внутри платформы между клиентами**

Результат успешного выполнения: **возвращается код перевода (уникальный номер)**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| UserToId | string | обязательный | - | Beribit ID клиента (Доступен в разделе 'внутренние переводы' (https://beribit.com/wallet/codes)|
| Amount | string | обязательный | - | Количество валюты |
| Token | string | обязательный | - | Токен |

|Список токенов |
| -- |
| RUB | 
| USDT |
| BTC |
| ETH |
| BNB |
| TRX |

Пример ответа (HTTP Code 200):
```
curl --location 'https://api.beribit.com/code/withdraw?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid_insert_here' \
--header 'Signature: signature-here' \
--header 'Content-Type: application/json' \
--data '{
    "Token": "RUB",
    "UserToId": "BeribitID_Here",
    "Amount": 0
}'
```
Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": "32e2e57c-1e02-4b7c-77ff-bfb680d9374c",
    "Time": "2024-01-26T15:33:41.9614616Z"
}
```

Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

Пример ошибки (HTTP Code 400):
```json
{
    "Success": false,
    "Error": {
        "Message": "UserToId not found"
    },
    "Time": "2024-01-26T15:33:41.9324992Z"
}
```

---

### <a id="deposit_code">Применить код</a>

Авторизация: **Требуется**

Тип запроса: **POST**

Путь: **code/withdraw**

Описание: **данная функция отвечает за применение крла**

Результат успешного выполнения: **возвращается сумму принятого RUB-кода**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| Code | string | обязательный | - | Beribit Code |

Пример ответа (HTTP Code 200):
```
curl --location 'https://api.beribit.com/code/deposit?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid-here' \
--header 'Signature: signature-here' \
--header 'Content-Type: application/json' \
--data '{
    "Code": "beribitcode00000000000000000000000000000000"
}'
```
Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": 10,
    "Time": "2024-01-28T18:14:44.3713744Z"
}
```

Пример ошибки (HTTP Code 401):
```json
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2024-01-26T15:33:41.0658698Z"
  }
}
```

Пример ошибки (HTTP Code 400):
```json
{
    "Success": false,
    "Error": {
        "Message": "Code not found"
    },
    "Time": "2024-01-26T15:33:41.0658698Z"
}
```

---

## <a id="trade">Торговые операции</a>

---

### <a id="trade_markets">Получить рынки</a>
Тип запроса: **GET**

Авторизация: **не требуется**

Путь: **/markets**

Описание: **данная функция отвечает за получение данных по рынкам**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе информацию: об названии (ид) рынка, код валюты цены, код валюты лота, минимальный размер лота**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Currency | string | обязательный | валюта, баланс которой требуется получить |

Пример запроса:
```
curl --location 'https://api.beribit.com/markets'
```

Пример ответа (HTTP Code 200):
```json
{
  "Success": true,
  "Result": [
    {
      "id": "USDT_RUB",
      "price_unit": "RUB",
      "price_step": 0.01,
      "lot_unit": "USDT",
      "lot_min": 0.01,
      "size_step": 0.01
    },
    {
      "id": "ETH_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "ETH",
      "lot_min": 0.001,
      "size_step": 0.001
    },
    {
      "id": "BTC_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "BTC",
      "lot_min": 0.0001,
      "size_step": 0.0001
    },
    {
      "id": "BNB_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "BNB",
      "lot_min": 0.0001,
      "size_step": 0.0001
    },
    {
      "id": "TRX_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "TRX",
      "lot_min": 0.0001,
      "size_step": 0.0001
    }
  ],
  "Time": "2024-01-28T18:21:49.8592526Z"
}
```

### <a id="trade_markets">Получить комиссии</a>
Тип запроса: **GET**

Авторизация: **Требуется**

Путь: **/markets**

Описание: **данная функция отвечает за получение данных по рынкам**

Результат успешного выполнения: **возвращается список данных, каждая запись содержит в себе информацию: об названии (ид) рынка, код валюты цены, код валюты лота, минимальный размер лота**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Currency | string | обязательный | валюта, баланс которой требуется получить |

Пример запроса:
```
curl --location 'https://api.beribit.com/markets'
```

Пример ответа (HTTP Code 200):
```json
{
  "Success": true,
  "Result": [
    {
      "id": "USDT_RUB",
      "price_unit": "RUB",
      "price_step": 0.01,
      "lot_unit": "USDT",
      "lot_min": 0.01,
      "size_step": 0.01
    },
    {
      "id": "ETH_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "ETH",
      "lot_min": 0.001,
      "size_step": 0.001
    },
    {
      "id": "BTC_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "BTC",
      "lot_min": 0.0001,
      "size_step": 0.0001
    },
    {
      "id": "BNB_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "BNB",
      "lot_min": 0.0001,
      "size_step": 0.0001
    },
    {
      "id": "TRX_USDT",
      "price_unit": "USDT",
      "price_step": 0.01,
      "lot_unit": "TRX",
      "lot_min": 0.0001,
      "size_step": 0.0001
    }
  ],
  "Time": "2024-01-28T18:21:49.8592526Z"
}
```
