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
8.2 [Получить комиссию](#trade_commission)<br/>
8.3 [Получить стакан по рынку](#trade_depth)<br/>
8.4 [Получить историю торговых операций](#trade_operations)<br/>
8.5 [Получить торговую операцию по id](#trade_id_operations)<br/>
8.6 [Выставить ордер](#place_order)<br/>
8.7 [Отменить торговую заявку](#cancel_order)<br/>

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

### <a id="trade_commission">Получить комиссии</a>
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
curl --location 'https://api.beribit.com/commission?Timestamp=2024-01-26T15:33:41' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": {
        "LimitOrder": 0.001,
        "MarketOrder": 0.0015
    },
    "Time": "2024-01-26T15:33:41.3995218Z"
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
### <a id="trade_depth">Получить стакан по рынку</a>

Авторизация: **Не требуется**

Тип запроса: **GET**

Путь: **/depth/{marketName}**

Описание: **данная функция отвечает за получение данных стакана по рынку**

Результат успешного выполнения: **возвращается данные по стакану для bids и asks, каждая запись содержит в себе информацию: цене, объеме, сумме, ставке, тип ордера (limit/factor)**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| marketName | string | обязательный | - | название (ид) рынка |

Пример запроса:
```
curl --location 'https://api.beribit.com/depth/USDT_RUB'
```

Пример ответа:
```json
{
    "Success": true,
    "Result": {
        "timestamp": 1706466621,
        "asks": [
            {
                "price": 91.15,
                "volume": 16147686.4535,
                "amount": 177155.09,
                "factor": 1.0157120570537107198573657232,
                "type": "Limit"
            },
            {
                "price": 91.16,
                "volume": 15316533.6424,
                "amount": 168018.14,
                "factor": 1.0158234900824604412747938489,
                "type": "Limit"
            },
            {
                "price": 91.17,
                "volume": 4273072.2576,
                "amount": 46869.28,
                "factor": 1.0159349231112101626922219746,
                "type": "Limit"
            },
            {
                "price": 91.18,
                "volume": 8531562.153,
                "amount": 93568.35,
                "factor": 1.0160463561399598841096501003,
                "type": "Limit"
            },
            {
                "price": 91.19,
                "volume": 738639,
                "amount": 8100,
                "factor": 1.016157789168709605527078226,
                "type": "Limit"
            },
            {
                "price": 91.2,
                "volume": 6840000,
                "amount": 75000,
                "factor": 1.0162692221974593269445063517,
                "type": "Limit"
            },
            {
                "price": 91.21,
                "volume": 1338597.96,
                "amount": 14676,
                "factor": 1.0163806552262090483619344774,
                "type": "Limit"
            },
            {
                "price": 91.22,
                "volume": 19733.6226,
                "amount": 216.33,
                "factor": 1.0164920882549587697793626031,
                "type": "Limit"
            },
            {
                "price": 91.23,
                "volume": 54738,
                "amount": 600,
                "factor": 1.0166035212837084911967907288,
                "type": "Limit"
            },
            {
                "price": 91.25,
                "volume": 117065.5375,
                "amount": 1282.91,
                "factor": 1.0168263873412079340316469802,
                "type": "Limit"
            },
            {
                "price": 91.26,
                "volume": 18768768.876,
                "amount": 205662.6,
                "factor": 1.0169378203699576554490751059,
                "type": "Limit"
            },
            {
                "price": 91.27,
                "volume": 54762,
                "amount": 600,
                "factor": 1.0170492533987073768665032316,
                "type": "Limit"
            },
            {
                "price": 91.29,
                "volume": 54774,
                "amount": 600,
                "factor": 1.017272119456206819701359483,
                "type": "Limit"
            },
            {
                "price": 91.31,
                "volume": 54786,
                "amount": 600,
                "factor": 1.0174949855137062625362157343,
                "type": "Limit"
            },
            {
                "price": 91.33,
                "volume": 54798,
                "amount": 600,
                "factor": 1.0177178515712057053710719857,
                "type": "Limit"
            },
            {
                "price": 91.34,
                "volume": 171262.5,
                "amount": 1875,
                "factor": 1.0178292845999554267885001114,
                "type": "Limit"
            },
            {
                "price": 91.35,
                "volume": 54810,
                "amount": 600,
                "factor": 1.0179407176287051482059282371,
                "type": "Limit"
            },
            {
                "price": 91.37,
                "volume": 54822,
                "amount": 600,
                "factor": 1.0181635836862045910407844885,
                "type": "Limit"
            },
            {
                "price": 91.39,
                "volume": 54834,
                "amount": 600,
                "factor": 1.0183864497437040338756407399,
                "type": "Limit"
            },
            {
                "price": 91.41,
                "volume": 54846,
                "amount": 600,
                "factor": 1.0186093158012034767104969913,
                "type": "Limit"
            },
            {
                "price": 91.43,
                "volume": 54858,
                "amount": 600,
                "factor": 1.0188321818587029195453532427,
                "type": "Limit"
            },
            {
                "price": 91.45,
                "volume": 54870,
                "amount": 600,
                "factor": 1.0190550479162023623802094941,
                "type": "Limit"
            },
            {
                "price": 91.47,
                "volume": 977840.8263,
                "amount": 10690.29,
                "factor": 1.0192779139737018052150657455,
                "type": "Limit"
            },
            {
                "price": 91.49,
                "volume": 912612.75,
                "amount": 9975,
                "factor": 1.0195007800312012480499219969,
                "type": "Limit"
            },
            {
                "price": 91.5,
                "volume": 3431250,
                "amount": 37500,
                "factor": 1.0196122130599509694673501226,
                "type": "Limit"
            },
            {
                "price": 91.51,
                "volume": 82140.2911,
                "amount": 897.61,
                "factor": 1.0197236460887006908847782483,
                "type": "Limit"
            },
            {
                "price": 91.52,
                "volume": 13543.1296,
                "amount": 147.98,
                "factor": 1.019835079117450412302206374,
                "type": "Limit"
            },
            {
                "price": 91.53,
                "volume": 6982242.4845,
                "amount": 76283.65,
                "factor": 1.02,
                "type": "Factor"
            },
            {
                "price": 91.54,
                "volume": 152352.7682,
                "amount": 1664.33,
                "factor": 1.0200579451749498551370626254,
                "type": "Limit"
            },
            {
                "price": 91.55,
                "volume": 203792.131,
                "amount": 2226.02,
                "factor": 1.0201693782036995765544907511,
                "type": "Limit"
            },
            {
                "price": 91.56,
                "volume": 85304.6208,
                "amount": 931.68,
                "factor": 1.0202808112324492979719188768,
                "type": "Limit"
            },
            {
                "price": 91.57,
                "volume": 83737.1022,
                "amount": 914.46,
                "factor": 1.0203922442611990193893470025,
                "type": "Limit"
            },
            {
                "price": 91.58,
                "volume": 98141.707,
                "amount": 1071.65,
                "factor": 1.0205036772899487408067751281,
                "type": "Limit"
            },
            {
                "price": 91.59,
                "volume": 565353.9294,
                "amount": 6172.66,
                "factor": 1.0206151103186984622242032538,
                "type": "Limit"
            },
            {
                "price": 91.6,
                "volume": 420081.264,
                "amount": 4586.04,
                "factor": 1.0207265433474481836416313795,
                "type": "Limit"
            },
            {
                "price": 91.61,
                "volume": 98598.0108,
                "amount": 1076.28,
                "factor": 1.0208379763761979050590595052,
                "type": "Limit"
            },
            {
                "price": 91.62,
                "volume": 6957468.8784,
                "amount": 75938.32,
                "factor": 1.021,
                "type": "Factor"
            },
            {
                "price": 91.63,
                "volume": 245074.5143,
                "amount": 2674.61,
                "factor": 1.0210608424336973478939157566,
                "type": "Limit"
            },
            {
                "price": 91.64,
                "volume": 61578.4144,
                "amount": 671.96,
                "factor": 1.0211722754624470693113438823,
                "type": "Limit"
            },
            {
                "price": 91.65,
                "volume": 86895.198,
                "amount": 948.12,
                "factor": 1.021283708491196790728772008,
                "type": "Limit"
            },
            {
                "price": 91.66,
                "volume": 117388.962,
                "amount": 1280.7,
                "factor": 1.0213951415199465121462001337,
                "type": "Limit"
            },
            {
                "price": 91.67,
                "volume": 252824.0266,
                "amount": 2757.98,
                "factor": 1.0215065745486962335636282594,
                "type": "Limit"
            },
            {
                "price": 91.68,
                "volume": 151787.2416,
                "amount": 1655.62,
                "factor": 1.0216180075774459549810563851,
                "type": "Limit"
            },
            {
                "price": 91.69,
                "volume": 13886702.6475,
                "amount": 151452.75,
                "factor": 1.0217294406061956763984845108,
                "type": "Limit"
            },
            {
                "price": 91.7,
                "volume": 6909614.257,
                "amount": 75350.21,
                "factor": 1.0218408736349453978159126365,
                "type": "Limit"
            },
            {
                "price": 91.71,
                "volume": 112367.6775,
                "amount": 1225.25,
                "factor": 1.0219523066636951192333407622,
                "type": "Limit"
            },
            {
                "price": 91.72,
                "volume": 91168.7628,
                "amount": 993.99,
                "factor": 1.0220637396924448406507688879,
                "type": "Limit"
            },
            {
                "price": 91.73,
                "volume": 55038,
                "amount": 600,
                "factor": 1.0221751727211945620681970136,
                "type": "Limit"
            },
            {
                "price": 93.7,
                "volume": 3777.047,
                "amount": 40.31,
                "factor": 1.0441274793848896813015377758,
                "type": "Limit"
            },
            {
                "price": 200,
                "volume": 200,
                "amount": 1,
                "factor": 2.2286605749944283485625139291,
                "type": "Limit"
            }
        ],
        "bids": [
            {
                "price": 91.14,
                "volume": 79341.0156,
                "amount": 870.54,
                "factor": 1.0156006240249609984399375975,
                "type": "Limit"
            },
            {
                "price": 91.13,
                "volume": 1864063.2387,
                "amount": 20454.99,
                "factor": 1.0154891909962112770225094718,
                "type": "Limit"
            },
            {
                "price": 91.12,
                "volume": 349999.2096,
                "amount": 3841.08,
                "factor": 1.0153777579674615556050813461,
                "type": "Limit"
            },
            {
                "price": 91.11,
                "volume": 249999.4623,
                "amount": 2743.93,
                "factor": 1.0152663249387118341876532204,
                "type": "Limit"
            },
            {
                "price": 91.1,
                "volume": 300999.866,
                "amount": 3304.06,
                "factor": 1.0151548919099621127702250947,
                "type": "Limit"
            },
            {
                "price": 91.09,
                "volume": 131872.8148,
                "amount": 1447.72,
                "factor": 1.015043458881212391352796969,
                "type": "Limit"
            },
            {
                "price": 91.07,
                "volume": 220733.6446,
                "amount": 2423.78,
                "factor": 1.0148205928237129485179407176,
                "type": "Limit"
            },
            {
                "price": 91.06,
                "volume": 203624.7296,
                "amount": 2236.16,
                "factor": 1.0147091597949632271005125919,
                "type": "Limit"
            },
            {
                "price": 91.05,
                "volume": 440000.0355,
                "amount": 4832.51,
                "factor": 1.0145977267662135056830844662,
                "type": "Limit"
            },
            {
                "price": 91.04,
                "volume": 264016,
                "amount": 2900,
                "factor": 1.0144862937374637842656563405,
                "type": "Limit"
            },
            {
                "price": 91.03,
                "volume": 309999.9341,
                "amount": 3405.47,
                "factor": 1.0143748607087140628482282148,
                "type": "Limit"
            },
            {
                "price": 91.01,
                "volume": 54606,
                "amount": 600,
                "factor": 1.0141519946512146200133719634,
                "type": "Limit"
            },
            {
                "price": 90.99,
                "volume": 54594,
                "amount": 600,
                "factor": 1.0139291285937151771785157121,
                "type": "Limit"
            },
            {
                "price": 90.98,
                "volume": 3787212.6326,
                "amount": 41626.87,
                "factor": 1.0138176955649654557610875864,
                "type": "Limit"
            },
            {
                "price": 90.97,
                "volume": 54582,
                "amount": 600,
                "factor": 1.0137062625362157343436594607,
                "type": "Limit"
            },
            {
                "price": 90.96,
                "volume": 1541.772,
                "amount": 16.95,
                "factor": 1.013594829507466012926231335,
                "type": "Limit"
            },
            {
                "price": 90.95,
                "volume": 40460.017,
                "amount": 444.86,
                "factor": 1.0134833964787162915088032093,
                "type": "Limit"
            },
            {
                "price": 90.94,
                "volume": 8699999.7218,
                "amount": 95667.47,
                "factor": 1.0133719634499665700913750836,
                "type": "Limit"
            },
            {
                "price": 90.93,
                "volume": 54558,
                "amount": 600,
                "factor": 1.0132605304212168486739469579,
                "type": "Limit"
            },
            {
                "price": 90.92,
                "volume": 2153550.2132,
                "amount": 23686.21,
                "factor": 1.0131490973924671272565188322,
                "type": "Limit"
            },
            {
                "price": 90.91,
                "volume": 3006239.153,
                "amount": 33068.3,
                "factor": 1.0130376643637174058390907065,
                "type": "Limit"
            },
            {
                "price": 90.9,
                "volume": 9179596.494,
                "amount": 100985.66,
                "factor": 1.0129262313349676844216625808,
                "type": "Limit"
            },
            {
                "price": 90.89,
                "volume": 464183.4101,
                "amount": 5107.09,
                "factor": 1.0128147983062179630042344551,
                "type": "Limit"
            },
            {
                "price": 90.88,
                "volume": 204749.9136,
                "amount": 2252.97,
                "factor": 1.0127033652774682415868063294,
                "type": "Limit"
            },
            {
                "price": 90.87,
                "volume": 122771.7309,
                "amount": 1351.07,
                "factor": 1.0125919322487185201693782037,
                "type": "Limit"
            },
            {
                "price": 90.85,
                "volume": 6336853.8205,
                "amount": 69750.73,
                "factor": 1.0123690661912190773345219523,
                "type": "Limit"
            },
            {
                "price": 90.83,
                "volume": 54498,
                "amount": 600,
                "factor": 1.0121462001337196344996657009,
                "type": "Limit"
            },
            {
                "price": 90.82,
                "volume": 2859.0136,
                "amount": 31.48,
                "factor": 1.0120347671049699130822375752,
                "type": "Limit"
            },
            {
                "price": 90.81,
                "volume": 54486,
                "amount": 600,
                "factor": 1.0119233340762201916648094495,
                "type": "Limit"
            },
            {
                "price": 90.8,
                "volume": 68249.82,
                "amount": 751.65,
                "factor": 1.0118119010474704702473813238,
                "type": "Limit"
            },
            {
                "price": 90.79,
                "volume": 254473.4752,
                "amount": 2802.88,
                "factor": 1.0117004680187207488299531981,
                "type": "Limit"
            },
            {
                "price": 90.77,
                "volume": 122636.6239,
                "amount": 1351.07,
                "factor": 1.0114776019612213059950969467,
                "type": "Limit"
            },
            {
                "price": 90.75,
                "volume": 54450,
                "amount": 600,
                "factor": 1.0112547359037218631602406953,
                "type": "Limit"
            },
            {
                "price": 90.73,
                "volume": 54438,
                "amount": 600,
                "factor": 1.0110318698462224203253844439,
                "type": "Limit"
            },
            {
                "price": 90.71,
                "volume": 54426,
                "amount": 600,
                "factor": 1.0108090037887229774905281926,
                "type": "Limit"
            },
            {
                "price": 90.7,
                "volume": 2043815.66,
                "amount": 22533.8,
                "factor": 1.0106975707599732560731000669,
                "type": "Limit"
            },
            {
                "price": 90.69,
                "volume": 54414,
                "amount": 600,
                "factor": 1.0105861377312235346556719412,
                "type": "Limit"
            },
            {
                "price": 90.68,
                "volume": 1537.026,
                "amount": 16.95,
                "factor": 1.0104747047024738132382438155,
                "type": "Limit"
            },
            {
                "price": 90.67,
                "volume": 54402,
                "amount": 600,
                "factor": 1.0103632716737240918208156898,
                "type": "Limit"
            },
            {
                "price": 90.65,
                "volume": 54390,
                "amount": 600,
                "factor": 1.0101404056162246489859594384,
                "type": "Limit"
            },
            {
                "price": 90.63,
                "volume": 54378,
                "amount": 600,
                "factor": 1.009917539558725206151103187,
                "type": "Limit"
            },
            {
                "price": 90.61,
                "volume": 154365.9143,
                "amount": 1703.63,
                "factor": 1.0096946735012257633162469356,
                "type": "Limit"
            },
            {
                "price": 90.59,
                "volume": 54354,
                "amount": 600,
                "factor": 1.0094718074437263204813906842,
                "type": "Limit"
            },
            {
                "price": 90.57,
                "volume": 54342,
                "amount": 600,
                "factor": 1.0092489413862268776465344328,
                "type": "Limit"
            },
            {
                "price": 90.55,
                "volume": 54330,
                "amount": 600,
                "factor": 1.0090260753287274348116781814,
                "type": "Limit"
            },
            {
                "price": 90.53,
                "volume": 54318,
                "amount": 600,
                "factor": 1.00880320927122799197682193,
                "type": "Limit"
            },
            {
                "price": 90.51,
                "volume": 54306,
                "amount": 600,
                "factor": 1.0085803432137285491419656786,
                "type": "Limit"
            },
            {
                "price": 90.5,
                "volume": 4838789.745,
                "amount": 53467.29,
                "factor": 1.0084689101849788277245375529,
                "type": "Limit"
            },
            {
                "price": 90.49,
                "volume": 54294,
                "amount": 600,
                "factor": 1.0083574771562291063071094272,
                "type": "Limit"
            },
            {
                "price": 90.47,
                "volume": 54282,
                "amount": 600,
                "factor": 1.0081346110987296634722531758,
                "type": "Limit"
            },
            {
                "price": 90.46,
                "volume": 339749.668,
                "amount": 3755.8,
                "factor": 1.0080231780699799420548250501,
                "type": "Limit"
            },
            {
                "price": 90.45,
                "volume": 54270,
                "amount": 600,
                "factor": 1.0079117450412302206373969244,
                "type": "Limit"
            },
            {
                "price": 90.44,
                "volume": 3790615.3376,
                "amount": 41913.04,
                "factor": 1.0078003120124804992199687988,
                "type": "Limit"
            },
            {
                "price": 90.43,
                "volume": 54258,
                "amount": 600,
                "factor": 1.0076888789837307778025406731,
                "type": "Limit"
            },
            {
                "price": 90.41,
                "volume": 55778.4495,
                "amount": 616.95,
                "factor": 1.0074660129262313349676844217,
                "type": "Limit"
            },
            {
                "price": 90.4,
                "volume": 14999.168,
                "amount": 165.92,
                "factor": 1.007354579897481613550256296,
                "type": "Limit"
            },
            {
                "price": 90.39,
                "volume": 393983.7969,
                "amount": 4358.71,
                "factor": 1.0072431468687318921328281703,
                "type": "Limit"
            },
            {
                "price": 90.37,
                "volume": 54222,
                "amount": 600,
                "factor": 1.0070202808112324492979719189,
                "type": "Limit"
            },
            {
                "price": 90.35,
                "volume": 54210,
                "amount": 600,
                "factor": 1.0067974147537330064631156675,
                "type": "Limit"
            },
            {
                "price": 90.34,
                "volume": 1531.263,
                "amount": 16.95,
                "factor": 1.0066859817249832850456875418,
                "type": "Limit"
            },
            {
                "price": 90.17,
                "volume": 4318.2413,
                "amount": 47.89,
                "factor": 1.0047916202362380209494094049,
                "type": "Limit"
            },
            {
                "price": 90.11,
                "volume": 184999.4344,
                "amount": 2053.04,
                "factor": 1.0041230220637396924448406508,
                "type": "Limit"
            },
            {
                "price": 89.83,
                "volume": 0.8873999999999999999999999972,
                "amount": 0.0098786596905265501502838695,
                "factor": 1.001,
                "type": "Factor"
            },
            {
                "price": 89.3,
                "volume": 200001.638,
                "amount": 2239.66,
                "factor": 0.9950969467350122576331624694,
                "type": "Limit"
            },
            {
                "price": 89,
                "volume": 400900.5,
                "amount": 4504.5,
                "factor": 0.9917539558725206151103186985,
                "type": "Limit"
            },
            {
                "price": 88.78,
                "volume": 49999.1204,
                "amount": 563.18,
                "factor": 0.9893024292400267439268999331,
                "type": "Limit"
            },
            {
                "price": 88.38,
                "volume": 49999.2174,
                "amount": 565.73,
                "factor": 0.9848451080900378872297749053,
                "type": "Limit"
            },
            {
                "price": 88,
                "volume": 2672272.24,
                "amount": 30366.73,
                "factor": 0.9806106529975484733675061288,
                "type": "Limit"
            },
            {
                "price": 87.78,
                "volume": 49999.488,
                "amount": 569.6,
                "factor": 0.9781591263650546021840873635,
                "type": "Limit"
            },
            {
                "price": 87.38,
                "volume": 49999.7098,
                "amount": 572.21,
                "factor": 0.9737018052150657454869623356,
                "type": "Limit"
            },
            {
                "price": 86.78,
                "volume": 49999.1648,
                "amount": 576.16,
                "factor": 0.9670158234900824604412747938,
                "type": "Limit"
            },
            {
                "price": 86.38,
                "volume": 49999.3354,
                "amount": 578.83,
                "factor": 0.962558502340093603744149766,
                "type": "Limit"
            },
            {
                "price": 81.33,
                "volume": 299.2944,
                "amount": 3.68,
                "factor": 0.9062848228214842879429462893,
                "type": "Limit"
            },
            {
                "price": 80,
                "volume": 800.8,
                "amount": 10.01,
                "factor": 0.8914642299977713394250055717,
                "type": "Limit"
            },
            {
                "price": 0.1,
                "volume": 782.999,
                "amount": 7829.99,
                "factor": 0.001114330287497214174281257,
                "type": "Limit"
            },
            {
                "price": 0.09,
                "volume": 0.09,
                "amount": 1,
                "factor": 0.0010028972587474927568531313,
                "type": "Limit"
            },
            {
                "price": 0.08,
                "volume": 0.08,
                "amount": 1,
                "factor": 0.0008914642299977713394250056,
                "type": "Limit"
            },
            {
                "price": 0.07,
                "volume": 0.07,
                "amount": 1,
                "factor": 0.0007800312012480499219968799,
                "type": "Limit"
            },
            {
                "price": 0.06,
                "volume": 0.06,
                "amount": 1,
                "factor": 0.0006685981724983285045687542,
                "type": "Limit"
            },
            {
                "price": 0.05,
                "volume": 0.05,
                "amount": 1,
                "factor": 0.0005571651437486070871406285,
                "type": "Limit"
            },
            {
                "price": 0.04,
                "volume": 0.04,
                "amount": 1,
                "factor": 0.0004457321149988856697125028,
                "type": "Limit"
            },
            {
                "price": 0.03,
                "volume": 0.03,
                "amount": 1,
                "factor": 0.0003342990862491642522843771,
                "type": "Limit"
            },
            {
                "price": 0.02,
                "volume": 0.02,
                "amount": 1,
                "factor": 0.0002228660574994428348562514,
                "type": "Limit"
            },
            {
                "price": 0.01,
                "volume": 1.5099,
                "amount": 150.99,
                "factor": 0.0001114330287497214174281257,
                "type": "Limit"
            }
        ]
    },
    "Time": "2024-01-28T18:30:21.704762Z"
}
```

### <a id="trade_operations">Получить историю торговых операций</a>
Тип запроса: **GET**

Авторизация: **Требуется**

Путь: **/orders/{marketName}**

Описание: **данная функция отвечает за получение данных по ордерам**

Результат успешного выполнения: **возвращается данные по ордерам**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| OrderState | string | необязательный | Статус ордера (необязательный, по умолчанию none): none - все статусы; wait - активные; done - закрытые; cancel - отмененные |
| OrderBy | string | необязательный | Сортировка (необязательный, по умолчанию descending): descending - последний по времени оредний первый; ascending - последний по времени оредний последний |
| Limit | int | необязательный | Кол-во ордеров в выгрузке (необязательный, по умолчанию 10, максимум 50) |
| Offset | int | необязательный | Кол-во ордеров которые необходимо пропустить (необязательный, по умолчанию 0) |

Пример запроса:
```
curl --location 'https://api.beribit.com/orders/USDT_RUB?Timestamp=2024-01-28T18:30:21&OrderState=none&OrderBy=Descending&Limit=10&Offset=0' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": [
        {
            "id": 60548627,
            "market": "USDT_RUB",
            "remaining_volume": 0.892,
            "executed_volume": 98.427,
            "volume": 99.319,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 1.088365,
            "funds_fee": 0.001635,
            "trades_count": 1,
            "created_at": "2024-01-25T01:43:36.049047",
            "updated_at": "2024-01-25T01:43:36.049047"
        },
        {
            "id": 60548615,
            "market": "USDT_RUB",
            "remaining_volume": 0.9029,
            "executed_volume": 0,
            "volume": 0.9029,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Cancel",
            "funds_received": 0,
            "funds_fee": 0,
            "trades_count": 0,
            "created_at": "2024-01-25T01:43:08.396915",
            "updated_at": "2024-01-25T01:43:08.396915"
        },
        {
            "id": 59683033,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 2.733,
            "volume": 2.733,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 0.029955,
            "funds_fee": 0.000045,
            "trades_count": 1,
            "created_at": "2024-01-17T14:35:23.921405",
            "updated_at": "2024-01-17T14:35:23.921405"
        },
        {
            "id": 59127345,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 5,
            "volume": 5,
            "side": "Sell",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 453.768325,
            "funds_fee": 0.681675,
            "trades_count": 1,
            "created_at": "2024-01-12T23:05:57.026569",
            "updated_at": "2024-01-12T23:05:57.026569"
        },
        {
            "id": 59127344,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 454.5,
            "volume": 454.5,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 4.9925,
            "funds_fee": 0.0075,
            "trades_count": 1,
            "created_at": "2024-01-12T23:05:54.913525",
            "updated_at": "2024-01-12T23:05:54.913525"
        },
        {
            "id": 59124303,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 5,
            "volume": 5,
            "side": "Sell",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 453.568625,
            "funds_fee": 0.681375,
            "trades_count": 1,
            "created_at": "2024-01-12T22:15:41.83801",
            "updated_at": "2024-01-12T22:15:41.83801"
        },
        {
            "id": 59124302,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 454.3,
            "volume": 454.3,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 4.9925,
            "funds_fee": 0.0075,
            "trades_count": 1,
            "created_at": "2024-01-12T22:15:37.486923",
            "updated_at": "2024-01-12T22:15:37.486923"
        },
        {
            "id": 59030447,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 1,
            "volume": 1,
            "side": "Sell",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 91.093155,
            "funds_fee": 0.136845,
            "trades_count": 1,
            "created_at": "2024-01-11T22:14:18.123674",
            "updated_at": "2024-01-11T22:14:18.123674"
        },
        {
            "id": 59030437,
            "market": "USDT_RUB",
            "remaining_volume": 0,
            "executed_volume": 91.24,
            "volume": 91.24,
            "side": "Buy",
            "ord_type": "Market",
            "state": "Done",
            "funds_received": 0.9985,
            "funds_fee": 0.0015,
            "trades_count": 1,
            "created_at": "2024-01-11T22:13:58.360604",
            "updated_at": "2024-01-11T22:13:58.360604"
        },
        {
            "id": 57103479,
            "market": "USDT_RUB",
            "remaining_volume": 10,
            "executed_volume": 0,
            "volume": 10,
            "side": "Sell",
            "ord_type": "Limit",
            "state": "Cancel",
            "price": 92,
            "funds_received": 0,
            "funds_fee": 0,
            "trades_count": 0,
            "created_at": "2023-12-28T09:21:14.849764",
            "updated_at": "2023-12-28T09:21:14.849764"
        }
    ],
    "Time": "2024-01-28T18:31:52.7489334Z"
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

### <a id="trade_id_operations">Получить торговую операцию по id</a>
Тип запроса: **GET**

Авторизация: **Требуется**

Путь: **/order/{orderId}**

Описание: **данная функция отвечает за получение данных по ид ордера**

Результат успешного выполнения: **возвращается данные по ордеру**

Пример запроса:
```
curl --location 'https://api.beribit.com/order/59127345?Timestamp=2024-01-28T18:30:21' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
```

Пример ответа (HTTP Code 200):
```json
{
    "Success": true,
    "Result": {
        "id": 59127345,
        "market": "USDT_RUB",
        "remaining_volume": 0,
        "executed_volume": 5,
        "volume": 5,
        "side": "Sell",
        "ord_type": "Market",
        "state": "Done",
        "funds_received": 453.768325,
        "funds_fee": 0.681675,
        "trades_count": 1,
        "created_at": "2024-01-12T23:05:57.026569",
        "updated_at": "2024-01-12T23:05:57.026569"
    },
    "Time": "2024-01-28T18:57:32.8012835Z"
}
```
Пример ошибки (HTTP Code 400):
```json
{
    "Success": false,
    "Error": {
        "Message": "Order not found"
    },
    "Time": "2024-01-28T18:57:55.3340153Z"
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
### <a id="place_order">Выставить ордер</a>
Тип запроса: **POST**

Путь: **/orders**

Описание: **запрос создаёт завку на покупку / продажу для выбранного рынка.**

Результат успешного выполнения: **возвращается данные по выставленной заявке**

Пример запроса:
```
curl --location 'https://api.beribit.com/orders?Timestamp=2024-01-28T18:30:21' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
--header 'Content-Type: application/json' \
--data '{
    "Market": "USDT_RUB",
    "Volume": 100.0,
    "Price": 92.0, 
    "OrderSide": "buy", 
    "OrderType": "limit" 
}'
```


При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| Market | string | обязательный | - | название (ид) рынка |
| Volume | decimal | обязательный | - | Cумма в валюте продажи |
| OrderSide | string | обязательный | - | Направление заявки, buy или sell |
| OrderType | string | обязательный | - | Тип заявки (limit/market) |
| Price | decimal | обязательный для типа limit | - | Фикcированная цена |

Пример ответа:
```json
{
    "Success": true,
    "Result": {
        "id": 61938327,
        "market": "USDT_RUB",
        "remaining_volume": 99.36,
        "executed_volume": 0,
        "volume": 99.36,
        "side": "Buy",
        "ord_type": "Market",
        "state": "Wait",
        "funds_received": 0,
        "funds_fee": 0,
        "trades_count": 0,
        "created_at": "2024-01-28T19:02:07.9552322Z",
        "updated_at": "2024-01-28T19:02:07.9552322Z"
    },
    "Time": "2024-01-28T19:02:07.9685794Z"
}
```
### <a id="cancel_order">Отменить торговую заявку</a>

Тип запроса: **POST**

Путь: **/orders/delete**

Описание: **запрос позволяет отменить активную торговую заявку.**

Результат успешного выполнения: **возвращается заявка до отмены**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Значение по умолчанию | Примечание |
| -- | -- | -- | -- | -- |
| Ид заявки | int | обязательный | - | ид заявки |

Пример запроса:
```
curl --location 'https://api.beribit.com/order/delete/61939228?Timestamp=2024-01-28T18:30:21' \
--header 'UID: uid-here' \
--header 'Signature: signature-here'
--header 'Content-Type: application/json' \
--data '{
    "Market": "USDT_RUB",
    "Volume": 100.0,
    "Price": 92.0, 
    "OrderSide": "buy", 
    "OrderType": "limit" 
}'
```

Пример ответа:
```json
{
    "Success": true,
    "Result": {
        "id": 61939228,
        "market": "USDT_RUB",
        "remaining_volume": 99.68,
        "executed_volume": 0,
        "volume": 99.68,
        "side": "Buy",
        "ord_type": "Limit",
        "state": "Cancel",
        "price": 89,
        "funds_received": 0,
        "funds_fee": 0,
        "trades_count": 0,
        "created_at": "2024-01-28T19:10:10.506162",
        "updated_at": "2024-01-28T19:10:10.506162"
    },
    "Time": "2024-01-28T19:10:26.192541Z"
}
```
