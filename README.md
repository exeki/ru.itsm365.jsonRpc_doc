Скриптовый модуль jsonRpc реализован в компании ITSM365 Максимом Демьяновым. Данная документация не является официальной.
Данный проект опубликован здесь только ради публикации документации (неофициальной) с целью ознакомления комьюнити с возможностями модуля.

Для получения модуля вы можете [обратится в техподдержку ITSM365 через портал](https://portal.itsm365.com/sd/), если вы являетесь ее клиентом, или же в техподдержку Naumen, если вы клиент Naumen, что бы они в свою очередь связались в ITSM365.
# Общее описание

jsonRpc - это модуль NSD (Naumen Service Desk), направленный на взаимодействие с NSD по одноименной технологии. Протокол
JSON-RPC (сокр. от англ. JavaScript Object Notation Remote Procedure Call — JSON-вызов удалённых процедур) — протокол
удалённого вызова процедур, использующий JSON для кодирования сообщений.

Наглядное описание протокола JSON-RPC [тут](https://habr.com/ru/post/441854/).

Основным преимуществом данного модуля при взаимодействии с NSD помощи него является возможность воспроизведения [условных операций NSMP API](https://www.naumen.ru/docs/sd/NSD_manual.htm#script/api.htm#op) (штатный [REST API](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm)не поддерживает условные операции, только прямое сравнение на равенство).
# Структура запроса к модулю

## Общие положения

Все запросы к модулю jsonRpc имеют общие черты:

1. Выполнятся через POST http метод;
2. Адресуются по URL:
   'https://mysd.ru/sd/services/rest/exec-post?accessKey={accessKey}&func=modules.jsonRpc.process&params=request,response,user&raw=true'
3. Имеют одинаковую структуру ключей верхнего уровня в body запроса и одинаковую структуру ключей верхнего уровня в body
   ответа;
4. Имеют один обязательный header 'Content-Type: application/json';

**Фактически, всё регулирование взаимодействия с модулем происходит на уровне body запроса.**

## Структура body запроса

### Описание body запроса

| Ключ    | Тип данных | Описание                                                                          | Обязательно |      |
|---------|------------|-----------------------------------------------------------------------------------|-------------|------|
| jsonrpc | string     | Версия протокола. Модуль умеет только в 2.0.                                      | ❌           | 2.0  |
| method  | string     | Код вызываемого метода.                                                           | ✔           |      |
| params  | object     | Параметры вызываемого метода. Набор параметров диктуется вызываемым методом.      | ✔           |      |
| id      | int        | ID запроса, дублируется в отчет. Если не указан, ответ вернется с id равным null. | ❌           | null |

### Пример body запроса

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "number": 634534
    },
    "view": [
      "number",
      "creationDate",
      "agreement",
      "category"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

## Структура body ответа

### Описание body ответа

| Ключ          | Тип данных         | Описание                                                                                                        | Обязательно | 
|---------------|--------------------|-----------------------------------------------------------------------------------------------------------------|-------------| 
| jsonrpc       | string             | Версия протокола. Возвращает переданную при запросе версию. Если версия не была передана, вернет "2.0".         | ✔           |  
| id            | int?               | ID запроса. Возвращает переданный при запросе id. Если id не был передан, вернет null.                          | ✔           |
| result        | list?/ или object? | Результат выполняемой операции. Значение зависит от вызванного в запросе метода.                                | ✔           |
| error         | object?            | Ошибка при выполнении операции. Если ошибки не было - вернет null, иначе вернет object с ключами code и message | ✔           |
| error.code    | int                | Код ошибки выполнения. Если ошибки не было - ключ не будет объявлен.                                            | ❌           |
| error.message | int                | Сообщение ошибки выполнения. Если ошибки не было - ключ не будет объявлен.                                      | ❌           |

### Пример body ответа (успешно)

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": [
    {
      "number": 634534,
      "creationDate": "2021.09.05 06:31:04",
      "agreement": {
        "title": "Услуги для аптечной сети",
        "UUID": "agreement$2283201"
      },
      "category": {
        "title": "Расхождение по кол-ву товара (Недовоз)",
        "UUID": "directory$147597706"
      }
    }
  ],
  "error": null
}
```

### Пример body ответа (ошибка)

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": null,
  "error": {
    "code": 500,
    "message": "Cannot invoke method collectEntries() on null object"
  }
}
```

# Условные операторы

## Общее описание

Условные операции используются в body запросов jsonRpc для расширенного сравнения значений атрибутов, при этом jsonRpc
не исключает прямое сравнение значений атрибутов объектов с значением-условием, к примеру:

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "number": 635124
    }
  },
  "view": [
    "UUID"
  ],
  "limit": 10,
  "offset": 0,
  "id": 1
}
```

Все условные операции объявляются в запросе как json объекты и должны находится по ключу params.query в body запроса.
Типовая структура json объекта (а не всего body запроса) условной операции:

```JSON
{
  "method": "like",
  "params": {
    "args": [
      "%наклад%"
    ]
  }
}
```

| Ключ        | Ожидаемый тип данных | Описание                                                                                                                           |
|-------------|----------------------|------------------------------------------------------------------------------------------------------------------------------------|
| method      | string               | Наименование вызываемого метода.                                                                                                   |
| params      | object               | Параметры метода. Используется не для всех методов. Если используется, то должен в обязательно порядке содержать в себе ключ args. |
| params.args | list                 | Передает аргуметы выполнения условной операции.                                                                                    |

## Форматы данных

jsonRpc поддерживает стандартные типы данных JSON (строка, число, объект, массив, boolean, null).
При сравнении со ссылочными атрибутами необходимо передавать строку с UUID.
При сравнении атрибутов с типом date необходимо передавать строку с датой в формате 'yyyy.MM.dd'.
При сравнении атрибутов с типом dateTime аргуметом необходимо передавать строку с датой в формате 'yyyy.MM.dd HH:mm:ss'.

## Условные операции

### like

### Назначение

Поиск по вхождению в строке (тексте) значения атрибута, который соответствует поисковому шаблону. Для формирования
поискового шаблона допустимо использование спецсимвола '%'.

### Ограничения

Ограничение: используется для атрибутов типа "Строка", "Текст", "Текст в формате RTF". Иначе генерируется исключение.

### Пример

Поиск заявок с темой, содержащей подстроку "наклад".

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "shortDescr": {
        "method": "like",
        "params": {
          "args": [
            "%наклад%"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

## isNotNull

#### Назначение

Поиск объектов, значение целевого атрибута которых не null или не пустой список.

#### Ограничения

ОТСУТСТВУЮТ

#### Пример

Поиск заявок с не пустой категорией.

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "category": {
        "method": "isNotNull"
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### isNull

#### Назначение

Поиск объектов, значение целевого атрибута которых null или пустой список.

#### Ограничения

ОТСУТСТВУЮТ

#### Пример

Поиск заявок с пустой категорией.

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "category": {
        "method": "isNull"
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### orEq

#### Назначение

Поиск объектов, значение целевого атрибута которых совпадает хотя бы с одним из значений из списке параметров.

#### Ограничения

Минимальное количество параметров: 2, иначе генерируется исключение.

#### Пример

Поиск заявок с услугой "slmService$67224875" или "slmService$81142345" или "slmService$74322343".

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "service": {
        "method": "orEq",
        "params": {
          "args": [
            "slmService$67224875",
            "slmService$81142345",
            "slmService$74322343"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### not

#### Назначение

Поиск объектов, значение целевого атрибута которых не совпадает ни с одним из значений из списка параметров. Допустимо
использование с одним параметром и с несколькими параметрами.

#### Ограничения

ОТСУТСТВУЮТ

#### Пример body запроса

Поиск заявок статус которых не "registered", и не "inprogress", и не "onNeg".

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "state": {
        "method": "not",
        "params": {
          "args": [
            "registered",
            "inprogress",
            "onNeg"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### eq

#### Назначение

Поиск объектов, значение целевого атрибута которых совпадает со значением параметра.

#### Ограничения

ОТСУТСТВУЮТ

#### Пример

Поиск заявок с услугой "slmService$67224875".

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "service": {
        "method": "eq",
        "params": {
          "args": [
            "slmService$67224875"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### in

#### Назначение

Поиск объектов, значение целевого атрибута которых содержится в списке параметров.

#### Ограничения

ОТСУТСТВУЮТ

#### Пример body запроса JSON-RPC запроса

Поиск заявок со статусом "registered" или "inprogress" или "onNeg".

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "state": {
        "method": "in",
        "params": {
          "args": [
            "registered",
            "inprogress",
            "onNeg"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### between

#### Назначение

Поиск объектов, значение целевого атрибута которых находится в рамках между value1 и value2, включая граничные
значения (value1 <= value <= value2).

#### Ограничения

Используется для атрибутов типа "Дата", "Дата/время" и числовых атрибутов, иначе генерируется исключение.

#### Пример body запроса JSON-RPC запроса

Поиск заявок с датой создания меж 2021.04.15 и 2022.05.23 (включительно).

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "creationDate": {
        "method": "between",
        "params": {
          "args": [
            "2021.04.15 00:00:00",
            "2022.05.23 00:00:00"
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### gt

#### Назначение

Поиск объектов, значение целевого атрибута которых больше значения указанного параметра.

#### Ограничения

Используется для атрибутов типа "Дата", "Дата/время" и числовых атрибутов, иначе генерируется исключение.

#### Пример

Поиск заявок с номером больше 642758.

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "number": {
        "method": "gt",
        "params": {
          "args": [
            642758
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

### lt

#### Назначение

Поиск объектов, значение целевого атрибута которых меньше значения указанного параметра.

#### Ограничения

Используется для атрибутов типа "Дата", "Дата/время" и числовых атрибутов, иначе генерируется исключение.

#### Пример

Поиск заявок с номером меньше 642758.

```JSON
{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "number": {
        "method": "lt",
        "params": {
          "args": [
            642758
          ]
        }
      }
    },
    "view": [
      "UUID"
    ],
    "limit": 10,
    "offset": 0
  },
  "id": 1
}
```

# Методы  модуля

Текущая реализация JSON-RPC поддерживает 4 метода:

- get - получить/найти один объект
- find - найти несколько объектов по критериям поиска
- create - создать объект
- edit - отредактировать объект

Примеры использования можно получить в postman-коллекции (во вложении).

Перечисленные методы являются аналогами штатных методов REST
API  [get](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm#get), [find](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm#find), [create](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm#create), [edit](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm#edit),
но обладают расширенным синтаксисом.
Метод [delete](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm#delete) и остальные методы
штатного REST API в данном модуле не реализованы, при необходимости их можно использовать в соответствии с
нативным [REST API](https://www.naumen.ru/docs/sd/NSD_manual.htm#RESTful/REST_API_method.htm).

## Метод get

### Общее описание

Метод **get** jsonRps выполнят поиск одного объекта по коду метакласса объекта и его атрибутам.
Метод является оберткой утилитарного метода get NSMP API.
Поддерживает условные операторы.

> Возвращает ошибку если по критериям поиска найдено несколько объектов.

### Структура body запроса

#### Описание

| Ключ         | Ожидаемый тип данных | Описание                                                                                                                                        | Обязательно | Значение по умолчанию |
|--------------|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|-------------|-----------------------|
| params.fqn   | string               | Метакласс искомых объектов                                                                                                                      | ✔           |                       |
| params.query | object               | Ассоциативный массив, где ключ - это код атрибута объекта, значение - это искомое значение или [условная операция](/assets/reks/sd/jsonRpc/op). | ✔           |                       |
| params.view  | list                 | Перечень кодов атрибутов, значения которых должны быть отданы в ответе.                                                                         | ❌           | \["UUID"]             |

#### Пример

```CURL
curl --location --request POST 'https://mysd.ru/sd/services/rest/exec-post?accessKey={someKey}&func=modules.jsonRpc.process&params=request,response,user&raw=true' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc": "2.0",
  "method": "get",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "UUID": "serviceCall$861450586"
    },
    "view": [
      "number",
      "creationDate",
      "agreement",
      "wayAddressing"
    ]
  },
  "id": 1
}'
```

### Структура result в body ответа

#### Описание

Если искомый объект успешно найден, то результатом выполнения операции в ключе result будет json объект - отражение
объекта базы NSD, каждый ключ в объекте - это код атрибута, значение - значение атрибута объекта в базе NSD.

Возвращает null в ключе result, если объект не найден.

#### Пример body ответа (объект успешно найден)

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "number": 1183017,
    "creationDate": "2022.10.25 18:55:08",
    "agreement": {
      "title": "Услуги для аптечной сети",
      "UUID": "agreement$2283201"
    },
    "wayAddressing": {
      "title": "Личный кабинет",
      "UUID": "wayAdressing$647202"
    }
  },
  "error": null
}
```

#### Пример body ответа (объект не найден)

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": null,
  "error": null
}
```

## Метод find

### Общее описание

Метод **find** выполняет поиск нескольких объектов по коду метакласса объектов и их атрибутам.
Метод является оберткой утилитарного метода find NSMP API.

### Структура body  запроса

#### Описание

| Ключ          | Ожидаемый тип данных | Описание                                                                                                                                       | Обязательно | Значение по умолчанию |
|---------------|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------|-------------|-----------------------|
| params.fqn    | string               | Код метакласса искомых объектов                                                                                                                | ✔           |                       |
| params.query  | object               | Ассоциативный массив, где ключ - это код атрибута объекта, значение - это искомое значение или [условная операция](/assets/reks/sd/jsonRpc/op) | ✔           |                       |
| params.view   | list                 | Перечень кодов атрибутов, значения которых должны быть отданы в ответе.                                                                        | ❌           | \["UUID"]             |
| params.limit  | int                  | Лимит объектов в ответе.                                                                                                                       | ❌           | 1000                  |
| params.offset | int                  | Отступ объект при формировании выборки для ответа.                                                                                             | ❌           | 0                     |

#### Пример

```CURL
curl --location --request POST 'https://mysd.ru/sd/services/rest/exec-post?accessKey={someKey}&func=modules.jsonRpc.process&params=request,response,user&raw=true' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc": "2.0",
  "method": "find",
  "params": {
    "fqn": "serviceCall",
    "query": {
      "creationDate": {
        "method": "between",
        "params": {
          "args": [
            "2021.04.15 00:00:00",
            "2022.05.23 00:00:00"
          ]
        }
      },
      "state": {
        "method": "in",
        "params": {
          "args": [
            "registered"
          ]
        }
      }
    },
    "view": [
      "number",
      "creationDate",
      "agreement",
      "wayAddressing"
    ],
    "limit": 2,
    "offset": 0
  },
  "id": 1
}'
```

### Структура body ответа

#### Описание

Результатом выполнения операции будет массив json объектов в ключе result, каждый объект в массиве является отражением
объекта базы NSD, каждый ключ в объекте - это код атрибута, значение - значение атрибута объекта в базе NSD.
Если не найдено ни одного объекта в виде результата вернется null.

#### Пример

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": [
    {
      "number": 642758,
      "creationDate": "2021.09.10 09:45:18",
      "agreement": {
        "title": "Услуги для центрального офиса",
        "UUID": "agreement$18542801"
      },
      "wayAddressing": {
        "title": "Личный кабинет",
        "UUID": "wayAdressing$647202"
      }
    },
    {
      "number": 671067,
      "creationDate": "2021.10.01 06:01:53",
      "agreement": {
        "title": "Услуги для центрального офиса",
        "UUID": "agreement$18542801"
      },
      "wayAddressing": {
        "title": "Личный кабинет",
        "UUID": "wayAdressing$647202"
      }
    }
  ],
  "error": null
}
```

## Метод create

### Общее описание

Метод **create** выполняет создание объекта базы NSD по коду типа объекта и атрибутам.
Метод является оберткой утилитарного метода create NSMP API.

### Структура body запроса

#### Описание

| Ключ         | Ожидаемый тип данных | Описание                                                                                                         | Обязательно | Значение по умолчанию |
|--------------|----------------------|:-----------------------------------------------------------------------------------------------------------------|-------------|-----------------------|
| params.fqn   | string               | Метакласс создаваемого объекта                                                                                   | ✔           |                       |
| params.attrs | object               | Ассоциативный массив, где ключ - это код атрибута объекта, значение - это значение атрибута создаваемого объекта | ✔           |                       |
| params.view  | list                 | Перечень кодов атрибутов, значения которых должны быть отданы в ответе.                                          | ❌           | \["UUID"]             |

#### Пример запроса

```CURL
curl --location --request POST 'https://mysd.ru/sd/services/rest/exec-post?accessKey={someKey}&func=modules.jsonRpc.process&params=request,response,user&raw=true' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc": "2.0",
  "method": "create",
  "params": {
    "fqn": "serviceCall$request",
    "attrs": {
        "clientName" : "Казанцев Егор",
        "clientEmail" : "myemail@somedomain.com",
        "clientPhone" : "8(800)555-35-35",
        "service" : "slmService$846227001",
        "agreement" : "agreement$181422901",
        "shortDescr" : "Какая то тема",
        "descriptionRTF" : "бр<br>бр"
    },
    "view": [
      "number",
      "UUID",
      "state"
    ]
  },
  "id": 1
}'
```

### Структура body успешного ответа

#### Описание

Если объект будет успешно создан, то результатом выполнения операции в ключе result будет json объект - отражение
созданного объекта базы NSD, каждый ключ в объекте - это код атрибута, значение - значение атрибута созданного объекта в
базе NSD.

#### Пример

```JSON
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "number": 1183021,
    "UUID": "serviceCall$861450589",
    "state": "registered"
  },
  "error": null
}
```

## Метод edit

### Общее описание

Метод **edit** выполняет редактирование объекта базы NSD.
Метод является оберткой утилитарного метода edit NSMP edit.

### Структура body запроса

#### Описание

| Ключ         | Ожидаемый тип данных | Описание                                                                                                                                                                 | Обязательно | Значение по умолчанию |
|--------------|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|-----------------------|
| params.uuid  | string               | UUID редактируемого объекта.                                                                                                                                             | ❌           |                       |
| params.fqn   | string               | Код метакласса искомого объекта, если используется поиск объекта по ключу query. Не нужно указывать если используется поиск по идентификатору по ключу uuid.             | ❌           |                       |
| params.query | object               | Ассоциативный массив, где ключ - это код атрибута объекта, значение - это искомое значение или условная операция. Используется для идентификации редактируемого объекта. | ❌           |                       |
| params.attrs | object               | Ассоциативный массив, где ключ - это код атрибута объекта, значение - это указываемое в атрибуте значение. Используется для указания параметров редактирования.          | ✔           |                       |
| params.view  | list                 | Перечень кодов атрибутов, значения которых должны быть отданы в ответе.                                                                                                  | ❌           | \["UUID"]             |

> При использовании метода базой предварительно выполняется идентификация объекта. Для этого ей **обязательно** нужно
> передать либо заполненный ключ "uuid", либо заполненный ключ "query". В случае, если будет использован query, база
> выполнит поиск объектов к редактированию. Если в результате поиска объектов будет найдено несколько - база вернет
> ошибку. **Настоятельно рекомендуется выполнять метод с использования идентификации объекта по ключу "uuid".**

#### Пример запроса

```CURL
curl --location --request POST 'https://mysd.ru/sd/services/rest/exec-post?accessKey={someKey}&func=modules.jsonRpc.process&params=request,response,user&raw=true' \
--header 'Content-Type: application/json' \
--data-raw '{
  "jsonrpc": "2.0",
  "method": "edit",
  "params": {
    "uuid": "serviceCall$861538701",
    "attrs": {
        "shortDescr" : "новая тема",
        "descriptionRTF" : "новое описание"
    },
    "view": [
      "shortDescr",
      "descriptionRTF",
      "UUID"
    ]
  },
  "id": 1
}'
```

### Структура result в body ответа

#### Описание

Если объект будет успешно отредактирован, то результатом выполнения операции в ключе result будет json объект -
отражение отредактированного объекта базы NSD, каждый ключ в объекте - это код атрибута, значение - значение атрибута
созданного объекта в базе NSD. Набор пар "ключ-значение" в объекте зависит от ключ "view" в запросе.

#### Пример

```JSON
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "shortDescr": "новая тема",
        "descriptionRTF": "новое описание",
        "UUID": "serviceCall$861538701"
    },
    "error": null
}```

