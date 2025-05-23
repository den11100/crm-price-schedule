# 📘 API: ПРОЦЕДУРНЫЕ КАБИНЕТЫ
Сгенерируй документацию API в формате Markdown, в одном цельном блоке без разбивки по частям.

## 📦 Особенности обработки ошибок

Все ответы от API приходят с HTTP статусом **200 OK**, вне зависимости от результата выполнения запроса.  
Информацию об успешности операции необходимо проверять по полю `status` в теле ответа:

- `"status": "success"` — операция выполнена успешно.
- `"status": "fail"` — произошла ошибка, описание находится в полях `error` и `code`.


## 🔐 Аутентификация

Все методы требуют заголовок:

X-Api-Key: {your_api_key}

При отсутствии или неправильном API-ключе:
```json
{
  "status": "fail",
  "error": "Не верный API ключ",
  "code": 2
}
```

# Получить список процедурных кабинетов.

`Метод: POST /api/price/schedule/procedure-rooms`

**Описание:**  
Отдаёт список активных процедурных кабинетов. (если в оответе перестает появлятся кабинет - значит кабинет помечен как удален. Или перестал быть процедурным кабинетом crm.room.is_procedure = 0)<br>
Если в теле запроса передан массив `crm_room_ids`, возвращаются только кабинеты с указанными ID; иначе — все активные процедурные кабинеты.

## Запрос

**Заголовки:**  
- `Content-Type: application/json`

**Можно пустое тело запроса или пустой массив**<br>
**Тело запроса (application/json):**  
```json
{
  "crm_room_ids": [3]
}
```

| Параметр     | Тип     | Обязательный | Описание                                                                 |
|--------------|---------|--------------|--------------------------------------------------------------------------|
| crm_room_ids | array   | нет          | Массив ID кабинетов для фильтрации. Если не указан — возвращаются все.   |

#### Ответ 200 OK

~~~json
{
  "status": "success",
  "data": [
    {
      "id": 3,
      "office_id": 1,
      "name": "Процедурный кабинет Тест (какойто текст)",
      "comment": "",     
      "min_time": "08:00:00",
      "max_time": "19:00:00",
      "grid_pitch": 10,    
      "fio": "Процедурный кабинет Тест",   
      "office": {
        "id": 1,
        "name": "МО Москва",
        "address": "Москва, ул. Улица, дом 28",
        "phones": "доб. 4017,4023",
        "comment": "г. Москва, ул. Улица, дом 28",
        "city": "Москва",
        "time_zone": "+03:00",
        "is_deleted": 0,
        "erp_medoffice_id": 1
      },
      "isActiveApiPrice": true
    },
    {...}
  ]
}
~~~

#### Ответ 200 OK
Если запросили кабинет по [id] но пришел пустой ответ, значит кабинет помечен как удален. Или перестал быть процедурным кабинетом crm.room.is_procedure = 0
~~~json
{
    "status": "success",
    "data": []
}
~~~

#### Ответ 200 ОК
Если какие либо ошибки
```json
{
  "status": "fail",
  "error": "Текст ошибки на русском",
  "code": (int) код ошибки
}
```


#### Поля объекта кабинета

| Поле                        | Тип       | Описание                                                               |
|-----------------------------|-----------|------------------------------------------------------------------------|
| `id`                        | integer   | Уникальный идентификатор кабинета (crm.room.id)                        |
| `office_id`                 | integer   | ID офиса (crm.office.id)                                               |
| `name`                      | string    | Название кабинета                                                      |
| `comment`                   | string    | Комментарий  может быть больше 255 символов                            |
| `min_time`                  | time      | Время начала работы (`HH:MM:SS`)                                       |
| `max_time`                  | time      | Время окончания работы (`HH:MM:SS`)                                    |
| `fio`                       | string    | Чистое название (в name бывают добавляют доп инфу - для crm)           |
| `isActiveApiPrice`          | boolean   | Активированно ли API для PRICE !!!                                     |
| `office`                    | object    | Объект офиса (см. ниже)                                                |

#### Поля объекта `office`

| Поле               | Тип       | Описание                                                   |
|--------------------|-----------|------------------------------------------------------------|
| `id`               | integer   | Уникальный идентификатор офиса (crm.office.id)             |
| `name`             | string    | Название офиса                                             |
| `address`          | string    | Адрес                                                      |
| `phones`           | string    | Телефоны                                                   |
| `comment`          | string    | Комментарии может быть больше 255 символов                 |
| `city`             | string    | Город                                                      |
| `time_zone`        | string    | Часовой пояс (`+03:00`)                                    |
| `is_deleted`       | integer   | Флаг удаления (0/1)                                        |
| `erp_medoffice_id` | integer   | ID медофиса в ERP                                          |



## 📘 Получение расписания ячеек процедурных кабинетов

### 📡 Метод запроса
`POST /api/price/schedule/procedure-cells`

---

### 📝 Описание
Метод возвращает расписание временных ячеек (свободных и занятых) для процедурных кабинетов на ближайшие 14 дней, начиная с текущей даты.<br>
При необходимости можно ограничить выборку конкретными кабинетами с помощью параметра `crm_room_ids`.

---

### 🔧 Тело запроса (JSON)

| Параметр       | Тип    | Обязательный | Описание |
|----------------|--------|--------------|----------|
| `crm_room_ids` | array  | Нет          | Массив ID процедурных кабинетов. Если не указан, возвращаются все процедурные кабинеты по которым есть расписание. |

#### 📥 Пример запроса

```json
{
  "crm_room_ids": [3, 5, 7]
}
```

#### Ответ
```json
{
  "status": "success",
  "data": {
    "1": {
      "3": {
        "fio": "Процедурный кабинет Москва",
        "crm_room_id": 3,
        "cells": [
          {
            "dt": "2025-05-21",
            "time_start": "08:00:00",
            "time_end": "08:10:00",
            "free": 1,
            "type": 1
          },
          {
            "dt": "2025-05-21",
            "time_start": "08:10:00",
            "time_end": "08:20:00",
            "free": 1,
            "type": 1
          }
        ]
      }
    }
  }
}
```

 "1" - это crm.office.id<br>
 "3" - это crm.room.id<br>
 
Придут все ячейки (свободные и занятые)<br>
free = 1 - свободна<br>
free = 0 - занята<br>

type = 1 - офлайн приём. (Для процедурных неаверное всегда 1)<br>
type = 3 - онлайн приём.<br>


#### Ответ 200 ОК, если приходит пустой массив значит не активно api у кабинета или он помечен как удаленный или не задано рассписание.
```json
{
    "status": "success",
    "data": []
}
```

#### Ответ 200 ОК
Если какие либо ошибки
```json
{
  "status": "fail",
  "error": "Текст ошибки",
  "code": (int) код ошибки
}
```


# Бронирование слота

### POST `/api/price/schedule/procedure-reserve-slot`

Бронирование слота на 20 минут в процедурном кабинете.

> ⚠️ Неподтвержденные слоты автоматически удаляются через 20 минут

### Описание

Позволяет забронировать слот. Валидация обязательных параметров, проверка существования процедурного кабинета и создание записи о бронировании. Валидация времени<br>
В случаи успешного бронирования нужно будет сохранить crm_appointment_id - он понадобится при подтверждении.

### Обязательные параметры (JSON тело запроса)

| Параметр       | Тип     | Описание                                 |
|----------------|---------|------------------------------------------|
| crm_room_id    | int     | Уникальный идентификатор кабинета (crm.room.id) |
| analysis_id    | int     | ID теста erp.analysis.id                 |
| dt             | string  | Дата процедуры (в формате Y-m-d)         |
| time_start     | string  | Время начала (формат H:i:s)                |
| time_end       | string  | Время окончания (формат H:i:s)             |

### Пример запроса

```json
{
  "crm_room_id": 3,
  "analysis_id": 25,
  "dt": "2025-05-21",
  "time_start": "08:00:00",
  "time_end": "08:10:00"
}
```

#### Ответ 200 ОК
```json
{
    "status": "success",
    "data": {
        "crm_appointment_id": 98435
    }
}
```



### Ошибки Ответ 200 ОК
```json
{
    "status": "fail",
    "error": "Длительность приёма не соответствует тесту!",
    "code": 13
}
```

```json
{
    "status": "fail",
    "error": "Данное время уже занято.",
    "code": 13
}
```



## 📘 API: Получение продолжительности приёма по тестам


### Описание:
Возвращает продолжительность приёма (в минутах) для указанных тестов по их `analysis_id`.  
Если список `analysis_ids` не передан — вернутся все записи.
Если для теста нет информации, значит приём длится 10 минут.


### URL: `/api/price/schedule/duration-analysis`
### 📥 Тело запроса может быть пустым или [] или [erp.analysis.id, ...]

```json
{
  "analysis_ids": [25, 30]
}
```

#### Ответ 200 ОК
```json
{
    "status": "success",
    "data": {
        "25": 20,
        "30": 20,
        "317": 20,
        "426": 40,
        "439": 20
    }
}
```

"25" - это erp.analysis.id


#### Ответ 200 ОК (значит нет информации по тесту)
```json
{
    "status": "success",
    "data": []
}
```


# Подтверждение слота после оплаты.
### URL: `/api/price/schedule/procedure-confirm-appointment`


#### Обязательные параметры:

| Ключ                | Тип     | Описание                                           |
|---------------------|---------|----------------------------------------------------|
| `lk_id`             | int     | Идентификатор личного кабинета                     |
| `crm_appointment_id`| int     | crm.appointment.id                                 |
| `price_order_uid`   | string  | Уникальный идентификатор заказа из прайса          |
| `p_email`           | string  | Email пациента                                     |
| `p_family`          | string  | Фамилия пациента                                   |
| `p_name`            | string  | Имя пациента                                       |
| `p_sex`             | int     | Пол пациента (1 - М; 2 - Ж)                        |
| `p_phone`           | string  | (только цифры)                                     |

#### Необязательные параметры:

| Ключ                | Тип     | Описание                                           |
|---------------------|---------|----------------------------------------------------|
| `p_patronymic`      | string  | Отчество пациента                                  |
| `p_birth`           | string  | Дата рождения в формате `Y-m-d`                    |

---

## Логика выполнения

1. По `crm_appointment_id` ищется слот в таблице `crm.appointment`.
   - Если не найден — возвращается сообщение:  
     `"Бронирование слота не найдено"`

2. Вызывается метод `Patient::getOrCreatePatientFromPrice($data)`:
   - При неудаче возвращается сообщение:  
     `"Ошибка создания пациента: <текст_ошибки>"`

4. Сохраняется модель Appointment:
   - Если сохранение не удалось — возвращается сообщение:  
     `"Ошибка сохранения записи: <список_ошибок>"`


---
### Пример запроса
```json
{
  "lk_id": 123,
  "crm_appointment_id": 98435,
  "price_order_uid": "622F2BCDC6C31", 
  "p_email": "patient@example.com",
  "p_family": "Иванов",
  "p_name": "Иван", 
  "p_patronymic": "Иванович",
  "p_sex": 1,
  "p_phone": "79001234567",
  "p_birth": "1980-12-30",
}
```

#### Ответ 200 ОК
```json
{
    "status": "success",
    "data": [
        "Бронирование подтверждено"
    ]
}
```


#### Ошибки 200 ОК 
```json
{
    "status": "fail",
    "error": "Не переданы обязательные параметры: crm_appointment_id",
    "code": 10
}
```

```json
{
    "status": "fail",
    "error": "Бронирование слота не найдено",
    "code": 14
}
```

```json
{
    "status": "fail",
    "error": "Ошибка создания пациента: Значение «Email заказчика» не является правильным email адресом.",
    "code": 14
}
```


## 🧾 Коды ошибок API

Описание всех возможных кодов ошибок, возвращаемых в поле `code` при статусе `"fail"`:

| Код | Константа                     | Описание                                                         |
|-----|-------------------------------|------------------------------------------------------------------|
| 2   | `CODE_BAD_API`                | Неверный API-ключ. Клиент не прошёл проверку авторизации.        |
| 8   | `CODE_NOT_FOUND`              | Запрашиваемый объект  не найден.                                 |
| 9   | `CODE_INVALID_FORMAT`         | Ошибка формата данных — например, тело запроса невалидный JSON.  |
| 10  | `CODE_REQUIRED_PARAM`         | Не переданы обязательные параметры запроса.                      |
| 11  | `CODE_ALREADY_EXIST`          | Объект с таким значением уже существует .                        |
| 12  | `CODE_VALIDATION`             | Ошибка валидации модели — некорректные или пустые значения.      |
| 13  | `CODE_VALIDATION_OR_NO_VACANT`| Ошибка валидации модели или слот занят                           |
| 14  | `CODE_ERROR_CONFIRM`          | Ошибка подтверждения слота.                                      |


