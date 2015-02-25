# Партнерский Программный Интерфейс Adaperio.ru

## Для работы с Adaperio через партнерский API нужно:

     1) Обратиться на support@adaperio.ru с просьбой выдать логин/пароль для партерской программы.

     2) Оплатить нужное кол-во отчетов. 

     3) Запустить тесты и убедиться, что они завершились без ошибок (тестовый номер а999му199 не приводит к снятию денег).

     4) Встроить API в свою систему.

## Для тестирования нужно:

     1) Добавить полученный от нас логин/пароль в файл test/adaperio_api_tst.js

     2) Установить нужные для тестирования пакеты: 
     npm install https fs crypto assert

     3) Установить нужный для тестирования фреймворк: 
     npm install -g mocha

     4) Запустить все тесты (с таймаутом 30 секунд): 
     mocha reporter --spec -t 30000

В результате выполнения тестов - вы произведете тестовую покупку и получите ссылку на отчет об автомобиле **а999му199**

## Описание API
Для работы используются REST-подобное API с JSON в качестве передаваемых данных. 
Адреса наших backend серверов: **partner.api.adaperio.ru**.
ВНИМАНИЕ: запрещено кэшировать IP адреса серверов. 

### 1. Получить данные о наличии информации об автомобиле

```javascript
GET /v1/data_for_cars/:num

Выполняется следующее: 
    1. Проверяется наличие данных об автомобиле;
    2. Возвращается тело в формате JSON c расшифровкой того, какие именно данные найдены.

Возвращается: 200 (OK), если данные были найдены; 
              404 (Not Found), если нет данных.
Внимание: метод может выполняться долгое время (вызов "блокирующий"), установите таймаут минимум 60 секунд.
```

Тело ответа:
```javascript
[{
     "num":"а001кк12",
     "vin":"XTA2*********1931",
     "carModel":"ВАЗ 21102 ЛЕГКОВОЙ",
     "year":"2000",

     // Если false - отчет "дешевый" (100 руб), если true - отчет "дорогой" (более 100 руб)
     "extendedReport":true,

     // ДТП (у автомобилей Мск региона ДТП может быть найдено после покупки отчета) 
     "accidentFound":false,

     // Найдены ли данные с аукционов
     “auctionsFound”:true,

     // Найдены ли фотографии ДТП/повреждений
     “picsFound":false,

     // Использование в качестве такси
     "taxiFound":false,

     // Найден ли пробег
     "milleageFound":true,

     // Фотографии автомобиля
     "autoNomerPics":[],

     // Найдены ли сведения из таможни
     "customsFound":false,

     // В розыске ли автомобиль
     "gibddWanted":false,

     // Есть ли ограничения
     "gibddRestricted":false,

     // Есть ли информация о комплектации
     "equipInfoFound":false,

     // CARFAX отчет
     "carfaxFound": true,

     // Данные о ремонтных работах
     "repairsFound": true

     ...
}
]
```

### 2. Получить данные о наличии информации об автомобиле по VIN

```javascript
GET /v1/data_for_cars_by_vin/:vin

Выполняется следующее: 
    1. Проверяется наличие данных об автомобиле;
    2. Возвращается тело в формате JSON c расшифровкой того, какие именно данные найдены.

Возвращается: 200 (OK), если данные были найдены; 
              404 (Not Found), если нет данных.
Внимание: метод может выполняться долгое время (вызов "блокирующий"), установите таймаут минимум 60 секунд.

Формат возвращаемых данных и поведение метода польностью идентичны методу #1.
```


### 3. Получить отчет

```javascript
GET /v2/partners/:login/report_by_num/:num?password=12345

Выполняется следующее:
    1. Проверяет логин и пароль партнера.
    2. Проверяет остаток баланса на счету партнера.
    3. Уменьшает остаток баланса на 1.
    4. Возвращает статус 200 + ссылку на отчет.

Возвращается: 200 (OK), в случае успеха.
              401 (Unauthorized), если введен неверный логин/пароль.
              402 (Payment Required), если необходимо пополнить баланс.
              404 (Not Found), если произошла ошибка.

Внимание: метод может выполняться долгое время (вызов "блокирующий"), установите таймаут минимум 60 секунд.
```

+ login - ваш логин;
+ num - гос.номер автомобиля в кодировке utf8 (кириллица), закодирован в 'URL encoding';
+ password - ваш пароль.

В случае успеха - тело ответа будет содержать JSON вида:
```javascript
{ 
     link: 'http://www.adaperio.ru/engine.html#/success?InvId=3269587&OutSum=100.000000&SignatureValue=1813ba713a5ee12abf0b7bb3e669d072',
     signature: 1813ba713a5ee12abf0b7bb3e669d072,
     invId: 3269587
}
```

**link** и будет являться ссылкой, которую можно передать пользователю. Время жизни такой ссылки - месяц. 

ВНИМАНИЕ: вызов этого метода возможен только из защищенного модуля (вашего backend’а), так как секретный пароль не должны быть доступен простому конечному пользователю. Если вызывать этот метод (даже с учетом https) из незащищенного приложения - пользователь сможет дизассемблировать его или просто перехватить пароль в памяти.

### 4. Получить отчет по VIN

```javascript
GET /v2/partners/:login/report_by_vin/:vin?password=12345

Выполняется следующее:
    1. Проверяет логин и пароль партнера.
    2. Проверяет остаток баланса на счету партнера.
    3. Уменьшает остаток баланса на 1.
    4. Возвращает статус 200 + ссылку на отчет.

Возвращается: 200 (OK), в случае успеха.
              401 (Unauthorized), если введен неверный логин/пароль.
              402 (Payment Required), если необходимо пополнить баланс.
              404 (Not Found), если произошла ошибка.

Внимание: метод может выполняться долгое время (вызов "блокирующий"), установите таймаут минимум 60 секунд.
```

+ login - ваш логин;
+ vin - VIN номер автомобиля (17 символов);
+ password - ваш пароль.

В случае успеха - тело ответа будет содержать JSON вида:
```javascript
{ 
     link: 'http://www.adaperio.ru/engine.html#/success?InvId=3269587&OutSum=100.000000&SignatureValue=1813ba713a5ee12abf0b7bb3e669d072',
     signature: 1813ba713a5ee12abf0b7bb3e669d072,
     invId: 3269587
}
```

**link** и будет являться ссылкой, которую можно передать пользователю. Время жизни такой ссылки - месяц. 

ВНИМАНИЕ: вызов этого метода возможен только из защищенного модуля (вашего backend’а), так как секретный пароль не должны быть доступен простому конечному пользователю. Если вызывать этот метод (даже с учетом https) из незащищенного приложения - пользователь сможет дизассемблировать его или просто перехватить пароль в памяти.


### 5. (ОПЦИОНАЛЬНО) Послать письмо c pdf отчетом на e-mail (возможно несколько) по номеру заказа

```javascript
GET /v2/partners/:login/orders/:invId/email_report/:emails?password=12345

Выполняется следующее:
    1. Проверяет логин и пароль партнера.
    2. Находит заказ, который должен был сформирован методом №2.
    3. Формирует pdf отчет.
    4. Посылает его на указанные адреса.
    5. Возвращает статус 200.

Возвращается: 200 (OK), в случае успеха.
              401 (Unauthorized), если введен неверный логин/пароль.
              404 (Not Found), если произошла ошибка.
```

+ login - ваш логин;
+ invId - значение получено из ссылки, которую вернул метод №2;
+ emails - список e-mail адресов получателей (разделенный запятыми), закодирован в 'URL encoding';
+ password - ваш пароль.

### 6. (ОПЦИОНАЛЬНО) Получить конечный JSON по номеру заказа
Если вы не хотите переходить на наш сайт (см. ссылку из метода №2) или хотите разобрать/обработать данные самостоятельно - такая возможность имеется. Вы можете получить вместо html отчета только "сырые" данные в формате JSON. 

```javascript
GET /v1/cars_by_order/:invId?signature=b76c0883ca7c4c623315183f6ab2cb0e

Выполняется следующее:
    1. Находит заказ, который должен был сформирован методом №2.
    2. Проверяет signature.
    4. Возвращает тело в формате JSON с полными данными об автомобиле.

Возвращается: 200 (OK), в случае успеха.
              404 (Not Found), если произошла ошибка.
```

+ login - ваш логин;
+ invId - значение получено из ссылки, которую вернул метод №2;
+ signature - значение получено из ссылки, которую вернул метод №2.

Описание формата данных отсутсвует.
