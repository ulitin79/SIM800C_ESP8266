   Прошивка 7.2 начала принимать входящие команды в формате **JSON** и предусматривает работу модема на скорости **57600**, предыдущая работала на скорости 9600;

Перед обновлением любой версии старше 31.12.2018 до 7.2 необходимо находясь с старой прошивке, в консоли модема отправить команду `AT+IPR=57600;&W`. Если мониторе порта после отправки команды появятся артефакты,что свидетельствует о смене скорости модемом, затем можно обновляться через *.bin файл. Не лишним будет сделать полный сброс настроек `http://192.168.4.1/hardreset`

* [Скачать актуальную версию прошивки 7.2 ](https://github.com/martinhol221/SIM800C_ESP8266/tree/master/firmware) управление JSON, обязательно перезалить дашборд
* [FULL - версия дашборд для MQTT Dasch](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/daschbord_full.txt) максимум информации
* [LITE  - версия дашборд для MQTT Dasch](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/daschbord_lite.txt) минимум кнопок
* [WEBASTO  - версия дашборд для MQTT Dasch](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/daschbord_lite.txt) 
управление реле К1

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/Daschbord.jpg)


## Архитектура обмена командами:

Все команды для **устройства** поступают в JSON формате в топик `car/c5/sub`, на который подписывается устройство, 
**Приложение** в смартфоне подписывается на сообщения приходящие в топик `car/c5/pub/...`

**********
## Контроль уровня сигнала

Запрос `{"cmd":20}` возвращает в топик `car/c5/pub/debug` сообщение `{"rssi":[24,0,0,0]}`, где 24 уровень GSM сигнала, возможные значения:

* 0 -115 дБм или меньше (отобразится только в консоли)
* 1 -111 дБм  (отобразится только в консоли)
* 2 ... 30 = -110 ... -54 дБм
* 31 - 52 дБм или больше
* 99 неизвестно или не обнаружено (отобразится только в консоли)


**********
## Контроль баланса

* Запрос `{"cmd":15}` отправит USSD запрос баланса  #100#
* Запрос `{"cmd":35}` отправит USSD запрос баланса  *100#
* Запрос `{"cmd":36}` отправит USSD запрос баланса  *102#
* Запрос `{"cmd":37}` отправит USSD запрос баланса  *105#

ответ вернется через 10 сек в топик `car/c5/pub/ussd` в DPU или GSM кодировке

**********
## Контроль времени

Запрос `{"timer":600}` устанавливает значение таймера обратного отсчета на 600 секунд, 
на последней секунде будут отключены все реле. Если не устанавливать таймер или установить его `{"timer":0}` , то отключение реле по таймеру не наступит.

**********
## Контроль напряжения

Запрос `{"run":[41],"det":21}` включит реле К4 и через 21 секунду выключит его, в случае если напряжение питания будет меньше порогового, в противном случае реле останется включенным. `"det":30` - таймер 30 сек и т. д. 

## Контроль нейтрали или состояния педали тормоза

Вход `IN1` подключается на выход концевика педали тормоза (для АКПП), или на датчик включенной передачи (МКПП), при появлении высокого логического уровня на этом вхоте все реле отключаются, а таймер сбрасывается в нулевое состояние.

**********
## Контроль температуры

Запрос `{"temp_min":4.25,"temp_max":18.50,"termostat":X}` устанавливает значение минимальной температуры для включения реле `4.25` град, максимальной температуры для отключения реле `18.50`, где  `X` - режим работы термостата

* `0` - контроль температуры отключен
* `1` - включено управление на реле K1
* `4` - включено управление на реле K4
**********

## Контроль местоположения

Запрос `{"cmd":5}`  возвращает в топик `car/c5/pub/gps` значение 52.834681,21.694131.

Модуль не имеет собственного GPS приемника, координаты рассчитываются триангуляцией по базовым станциям, подобно локации в смартфонах, и имеют точность 50 - 800 м. в условиях города. В целях защиты от несанкционированного использования функция не доступна в готовых устройствах, и становится доступной только после обновления прошивки с Github, при условии отсутствия злого умысла в ее применении, и недопустимости слежения за кем-то.

## Активация WI-FI точки доступа

Запрос `{"wifi":600}` включает на устройстве точку доступа SSID `Webasto_123456` на 600 секунд с паролем `martinhol221` , страница настроек доступна по адресу `http://192.168.4.1`

**********
## Управление центаральным замком

Запрос `{"run":[41,5,40]}` или `{"run":[41,5,4,40]}` сгенерирует импульс 0.5 или 0.75 сек на реле `К4` которое может быть подключено к кнопке блокировки дверей салона


**********
## SMS уведомления 

Запрос `{"sms":45}` вернет СМС сообщение через 45 секунд

**********
## Обновление показаний

Запрос `{"cmd":7}` возвращает все параметры устройства в формате JSON:

`{"pin":[13.73,1,1,0,0,0,1,0],"temp":[14.81,14.50,14.13],"time":["8:54",2day.18:28:24]}`

распарсить строку можно с помощью [JSON pach](https://github.com/json-path/JsonPath) доступного в большинстве MQTT приложений
* `$.pin[0]` - вернёт напряжение питания устройства (13.73)
* `$.pin[1]` - вернёт состояние реле `K1` (1) - включенное реле на АСС
* `$.pin[2]` - вернёт состояние реле `K2` (1) - включенное реле на зажигание
* `$.pin[3]` - вернёт состояние реле `K3` (0)  
* `$.pin[4]` - вернёт состояние реле `K4` (0)
* `$.pin[5]` - вернёт состояние реле `K5` (0)   
* `$.pin[6]` - вернёт состояние входа `IN1` (0) - состояние педали СТОП
* `$.pin[7]` - вернёт состояние входа `IN2` (1) - контроль включенного зажигания
* `$.temp[0]` - вернёт температуру датчика с индексом `0`  (14.81)
* `$.temp[1]` - вернёт температуру датчика с индексом `1`  (14.50)
* `$.temp[2]` - вернёт температуру датчика с индексом `2`  (14.13)
* `$.time[0]` - вернёт значение таймера обратного отсчета `8:54` минут 
* `$.time[1]` - вернёт время работы устройства `2day.18:28:24`

**********
## Управление реле

* Запрос `{"run":[21]}` - включит реле К2
* Запрос `{"run":[20]}` - отключит реле К2
* Запрос `{"run":[11],"cmd":7}` - включит реле К1 и отправит параметры в приложение
* Запрос `{"run":[10],"cmd":7}` - отключит реле К1 и отправит параметры в приложение
* Запрос `{"run":[51,2,50]}` - импульс 50 миллисекунд на К5  
* Запрос `{"run":[51,3,50]}` - импульс 150 миллисекунд  на К5  
* Запрос `{"run":[41,3,3,40]}` - импульс 200 миллисекунд  на К4 
* Запрос `{"run":[41,5,40,XX]}` - запускает цепочку действий 

где **XX** - код действия:
* `1` пауза 0.03 секунды
* `2` пауза 0.05 секунды
* `3` пауза 0.1 секунды
* **`4` пауза 0.25 секунды**
* **`5` пауза 0.5 секунды**
* **`6` пауза 1 секунда**
* **`7` пауза 3 секунды**
* `8` пауза 5 секунд
* `9` пауза 10 секунд
* **`10` Отключить реле К1 (ACC, потребители)**
* `11` Включить реле К1 при любых условиях
* **`12` Включить реле К1 в случае, если на IN1 низкий логический уровень (не включена передача)**
* `13` Включить реле К1 в случае, если на IN2 низкий логический уровень
* `14` Включить реле К1 в случае, если на IN1 и IN2 низкий логический уровень
* `15` Включить реле К1 в случае, если на IN1 или IN2 низкий логический уровень
* **`20` Отключить реле К2 (ЗАЖИГАНИЯ)**
* `21` Включить реле К2 при любых условиях
* **`22` Включить реле К2 в случае, если на IN1 низкий логический уровень (не включена передача)**
* `23` Включить реле К2 в случае, если на IN2 низкий логический уровень
* `24` Включить реле К2 в случае, если на IN1 и IN2 низкий логический уровень
* `25` Включить реле К2 в случае, если на IN1 или IN2 низкий логический уровень
* **`30` Отключить реле К3 (СТАРТЕРА)**
* `31` Включить реле К3 при любых условиях
* `32` Включить реле К3 в случае, если на IN1 низкий логический уровень
* `33` Включить реле К3 в случае, если на IN2 низкий логический уровень
* `34` Включить реле К3 в случае, если на IN1 и IN2 низкий логический уровень
* `35` Включить реле К3 в случае, если на IN1 или IN2 низкий логический уровень
* **`36` Включить реле К3 в случае, если измеренное напряжение ниже порогового (нет зарядки)**
* `37` Включить реле К3 в случае, если выполнены пункты 36 и 32
* `40` Отключить реле К4 (внешнее) 
* `41` Включить реле К4 при любых условиях
* `42` Включить реле К4 в случае, если на IN1 низкий логический уровень
* `43` Включить реле К4 в случае, если на IN2 низкий логический уровень
* `44` Включить реле К4 в случае, если на IN1 и IN2 низкий логический уровень
* `45` Включить реле К4 в случае, если на IN1 или IN2 низкий логический уровень
* **`50` Отключить реле К5 (реле обходчика иммобилайзера)** 
* `51` Включить реле К5 при любых условиях
* **`52` Включить реле К5 (обходчика иммобилайзера) в случае, если не включена пердача
* `53` Включить реле К5 в случае, если на IN2 низкий логический уровень
* `54` Включить реле К5 в случае, если на IN1 и IN2 низкий логический уровень
* `55` Включить реле К5 в случае, если на IN1 или IN2 низкий логический уровень

******
## Пример для формирования своего сценария запуска двигателя автомобиля

* Реле K1 - АСС (потребители) или на обманку педали стоп (характерно для роботов)
* Реле К2 - зажигание
* Реле К3 - стартер
* Реле К4 - реле на кнопку блокировки салона
* Реле К5 - внешенее обходчика иммобилайзера

Шаблоны команд: 

* `{"timer":600,"run":[22,7,36,6,4,30,5],"cmd":7,"det":30}` - запуск (только реле К2 и К3), время прогрева 10 мин. 
* `{"timer":600,"run":[52,5,22,7,36,6,4,30,6,50,5],"cmd":7,"det":30}` - запуск (реле К5, К2 и К3)
* `{"timer":600,"run":[52,5,12,5,22,7,36,6,4,30,6,50,5],"cmd":7,"det":30}`- запуск (реле К5, К1, К2 и К3)
* `{"timer":600,"run":[52,5,12,5,22,7,36,6,4,30,6,10,2,50,5],"cmd":7,"det":30}`- запуск (реле К5, К1, К2 и К3)
* `{"timer":600,"run":[52,5,22,7,36,6,4,30,6,12,5,50,5],"cmd":7,"det":30}`- запуск (реле К5, К2, К3 и К1)
* `{"timer":600,"run":[52,5,12,5,22,7,10,4,36,6,4,30,4,12,7,50,4],"cmd":7,"det":30}`- запуск (реле К5, К2, К1 и К3)
* `{"timer":600,"run":[52,5,22,5,12,7,36,6,4,30,7,50,5,10,5],"cmd":7,"det":30}`- запуск (реле К5, К2, К1 и К3)
* `{"timer":1800,"run":[12,4],"cmd":7}` - вебасто на реле К1 на 30 минут
* `{"timer":1200,"run":[22,4],"cmd":7}` - вебасто на реле К2 на 20 минут
* `{"timer":0,"run":[50,3,40,3,30,3,20,3,10,3],"cmd":7}` - стоп на все        
* `{"run":[41,5,40]}` - импульс на ЦЗ

Расшифровка:

*  `"timer":600` - установит таймер обратного отсчета на 600 сек (10 минут)
* `"run":[` - запустит сценарий по кодам :
* `52` включит К5, если авто не на передаче (контроль клеммы IN1)
* `5`  пауза 0,5 сек.
* `22` включит реле К2, если авто не на передаче (контроль клеммы IN1)
* `5`  пауза 0,5 сек. 
* `12` включит реле К1, если авто не на передаче (контроль клеммы IN1)
* `7` пауза 3 сек.
* `36` включит реле K3,  если измеренное напряжение ниже порогового
* `6` пауза 1,0 сек.
* `4` дополнительная пауза 0,25 сек. 
* `30` отключит реле К3 - важно не забыть
* `7` пауза 3 сек.
* `50` отключит реле К5
* `5` пауза 0,5 сек.
* `10`отключит реле К1
* `5` пауза 0,5 сек.
* `]` - конец сценария
* `"cmd":7` - отправит параметры  в смартфон
* `"det":30` установит время в 30 сек. через которое произойдет проверка запуска двигателя, путем сравнения измерянного напряжений с заданным напряжением зарядки при заведенном моторе (обычно 13-13.5 вольт.)

## Управленние по звонку с вводом DTMF команд

При звонке на устройство модем мнимет выхов только с телефона хозяина, после "снятия трубки" будет ожидать ввода команд

* `1*1234` - запустит сценарий `{"timer":600,"run":[52,5,12,5,22,7,36,6,4,30,7,50,5],"det":30}`
* `2*1234` - запустит сценарий `{"timer":600,"run":[52,5,22,5,12,7,36,6,4,30,7,50,5,10,5],"det":30}`
* `3*1234` - запустит сценарий `{"timer":1200,"run":[52,5,12,5,22,7,36,6,4,30,7,50,5],"sms":45,"det":30}`
* `4*1234` - запустит сценарий `{"timer":1200,"run":[52,5,22,5,12,7,36,6,4,30,7,50,5,10,5],"sms":45,"det":30}`
* `5*1234` - запустит сценарий `{"timer":600,"run":[52,5,12,5,22,7,10,4,36,6,4,30,4,12,7,50,4],"sms":45,"det":30}`
* `666`    - пришлет СМС через 20 сек `{"sms":20}`
* `777`    - перезагрузит модем
* `#`      - удалит ошибочно нажатые кнопки
* `0`      - отключит все реле  `{"timer":0,"run":[50,4,40,4,30,4,20,4,10,4]}`

где `1234`     - пинкод для управления   

****

## Выбор MQTT приложения в телефон для управления по GPRS

Для Android

* [MQTT Dash Google Play](https://play.google.com/store/apps/details?id=net.routix.mqttdash&hl=ru) recommend
* [IoT MQTT Panel](https://play.google.com/store/apps/details?id=snr.lab.iotmqttpanel.prod)
* [IoT MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard)
* [Linear MQTT Dashboard](https://play.google.com/store/apps/details?id=com.ravendmaster.linearmqttdashboard)

Для IOS: 
[IoT OnOff](https://itunes.apple.com/be/app/iot-onoff/id1267226555?mt=8) 

## Выбор MQTT брокера (далее сервера)

***********************************

MQTT broker

https://www.cloudmqtt.com

http://flyhub.org

**********************************

## Настройка приложений сводится к трем этамам:

### Регистрация на сервере

Cloudmqtt.com > Log in> + Create Nwe Intance > 

`Name`: Любое имя

`Plan`: Cute Cat(Free)

`Tags`: Любое имя

`Data center`: EU-West-1 (Ireland) > Confirm > Create Nwe Intance

Внесение настроек сервера `Server`,`User`,`Password`,`Port` приложение

Конфигурация интерфейса приложения (кнопочек, ползунков, индикаторов и др.)

[Видео по регистрации на youtube.com](https://www.youtube.com/watch?v=xgZZ417HFFQ)

************************************

## Готовая приборная панель программы MQTT Dasch (далее daschbord)

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/daschbord1.jpg)


**********
## Настройки плиток в MQTT Dash в ручном режиме

* Тип `Текст`
* Имя `Температура`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.temp[0]` - для первого термодатчика
* Другие настройки `QoS(0)`, цвет и размер плитки по вкусу.

***

* Тип `Диапазон/прогресс`
* Имя `Напряжение`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[0]` - для чтения напряжения
* Минимум `0`, Максимум `15`, Постфикс `V`, Точность `2`
* Плитка мигает для привлечения внимания... `val > 13.2`
* Другие настройки `QoS(0)`, цвет и размер плитки по вкусу.

**********

* Тип `Переключатель/кнопка`
* Имя `СТАРТ`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[2]` - контроль по состоянию реле зажигания
* Вкл: `1`, Выкл: `0`, другие настройки `QoS(0)`
* Во вкладку `ON TAP` должен быть помещен этот код , который исполняется при нажатии

> app.publish('car/c5/sub','{"timer":600,"run":[52,5,22,5,12,7,36,6,4,30,7,50,5,10,5],"cmd":7,"det":30}', false, 0);

**********


* Тип `Переключатель/кнопка`
* Имя `СТОП`
* Топик (SUB) `car/c5/pub` - желательно не изменять
* Извлечь, используя JSON path.... `$.pin[2]` - контроль по состоянию реле зажигания
* Вкл: `1`, Выкл: `0`, другие настройки `QoS(0)`
* Во вкладку `ON TAP` должен быть помещен этот код , который исполняется при нажатии

> app.publish('car/c5/sub','{"timer":0,"run":[50,4,40,4,30,4,20,4,10,4],"cmd":7}', false, 0);



### Потребление трафика:

Один пакет GPRS данных состоит в среднем из 100 байт, отправляется устройством каждую минуту, 100 байт х 60 минут х 24 часа х 31 день = около 4.5 MB/мес, однако на практике оператор трафик считают больший, у меня набегает 9.5 Мб в месяц.
Стоит отметить что сотовые операторы по разному округляют трафик в конце сессии, в результате чего, списывать с баланса могут в разы больше фактически затраченного.

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/trafic.JPG)
