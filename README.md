* [Скачать актуальную версию прошивки 7.3 ](https://github.com/martinhol221/SIM800C_ESP8266/tree/master/firmware) обязательно перезалить дашборд
После перепрошивки зайти в меню прошивки через Web-интерфейс и сделать сброс настроек.

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/old/CkAAAgLhoeA-960.jpg)

Приложение CarMQTT Control

* ![](https://github.com/martinhol221/SIM800C_ESP8266/raw/master/old/CarMQTTcontrol2.2.2.jpg)

* ![](https://github.com/martinhol221/SIM800C_ESP8266/raw/master/old/CarMQTT_JSON.png)

## Коротко о протоколе MQTT в конкретной прошивке 

Приложение подписывается на топик `car/c5/pub` - и отправляет команды в топик устройства.
Устройство подписывается на топик `car/c5/sub` и реагирует на следующие сообщения в этом топике:

* `prog0`...`prog9` - исполняет одну из 10 програм храниящихся в настройках ESP8266
* `wifion`      - включает WI-FI точку доступа `Webasto_123456` с паролем `martinhol221`
* `wifioff`     - отключает WI-FI точку доступа
* `control1=1`  - активирует контроль за каналом IN1 (педаль тормоза, или другие функции)
* `control1=0`  - деактивирует контроль за каналом IN1
* `control2=1`  - активирует контроль за каналом IN2 (датчик удара, объема, или другие функции)
* `control2=0`  - деактивирует контроль за каналом IN2
* `termostat=1` - активирует контроль вилкой температур заданных в настройках (термостат, автопрогрев)
* `termostat=1` - деактивирует контроль температурой
* `termostat=0` - деактивирует контроль температурой
* `reboot` -  перезагрузка ESP8266
* `timer=15` - установка таймера обратного отсчета на 15 минут 
* `starter=1600` - установка максимального времени прокрутки стартера в 1.600 сек. 
* `ussd`  - публикует в топик `car/c5/pub/ussd` баланс	`Balans=5.00r MB=454` (только GSM кодировка)
* `debug` - публикует в топик `car/c5/pub/debug` JSON строку статистики соединения `{"debug":[1,0,0,0,5,5,1]}`
* `update`- публикует в топик `car/c5/pub`	JSON строку с данными текущего состояния 
`{"pin":[13.73,1,1,0,0,0,1,0],"temp":[14.81,14.50,14.13],"time":[45,3455,4],"control":[1,0,1,0]}`, извлечь необходимый параметр можно с помощью [JSON pach](https://github.com/json-path/JsonPath) доступного в большинстве MQTT приложений

* `$.pin[0]` - вернёт напряжение питания устройства (13.73)
* `$.pin[1]` - вернёт состояние реле `K1` (1) - включено
* `$.pin[2]` - вернёт состояние реле `K2` (1) - включено
* `$.pin[3]` - вернёт состояние реле `K3` (0) - отключено 
* `$.pin[4]` - вернёт состояние реле `K4` (0) - отключено
* `$.pin[5]` - вернёт состояние реле `K5` (0) - отключено 
* `$.pin[6]` - вернёт состояние входа `IN1` (0) - нет напряжения
* `$.pin[7]` - вернёт состояние входа `IN2` (1) - есть напряжение
* `$.temp[0]` - вернёт температуру датчика с индексом `0`  (14.81)
* `$.temp[1]` - вернёт температуру датчика с индексом `1`  (14.50)
* `$.temp[2]` - вернёт температуру датчика с индексом `2`  (14.13)
* `$.time[0]` - вернёт значение таймера обратного отсчета `45` минут 
* `$.time[1]` - вернёт время работы устройства `3455`
* `$.control[0]` -  контроль за каналом IN1 `1` активен 
* `$.control[1]` -  контроль за каналом IN1 `0` не активен 
* `$.control[3]` -  контроль за вилкой температур `0` не активен 



## Редактор програм

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/old/DkAAAgLhoeA-1920.jpg)
  
## Контроль включения стартера

В настройках интерфейса можно задать условие из трех пунктов для разрешение включения стартера в зависмости он напряжения АКБ  (при работающем двигателе оно будет выше 12,90V), и состояние каналов  `IN1` или `IN2`. обычно, они подключается на выход концевика педали тормоза (для АКПП), или на датчик включенной передачи (МКПП).

Длительность работы стартера задается в милисекундах (1 / 1000 сек.)

Поконтролю зарядки АКБ после запуска устройство понимает что двигатель запущен, контроль можно отключить.

## Контроль по аппаратному прерыванию каналов IN1 и IN2 и кнопке

* Cмена состояния каналов `IN1` или `IN2` с `0` на `12V` вызывает программы `prog0`... `prog9` заданные в настройках для этого канала
* Cмена состояния Кнопки с `3.3V` на `0` вызывает программы `prog0`... `prog9` или активации Wi-Fi

**********
## Управление по термостату

В настройках устройства задается:

* минимальная температура при падении ниже которой происходит вызов програм `prog0`... `prog9`
* максимальная температура при  превышении которой происходит вызов програм `prog0`... `prog9`
* индекс подключенного датчика с которого контролируется температура
* флаг активациии или деактивации режима

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/old/AkAAAgLhoeA-1920.jpg)

## Активация WI-FI точки доступа

Устройство генерирует точку доступа SSID `Webasto_123456` на 10 минут с паролем `martinhol221` , к которой можно подключиться с телефона для обновления прошивки или внесения изменений в настройки, страница настроек доступна по адресу `http://192.168.4.1`.
Активация WIFI возможна несколькими способами

* Запрос из приложения `wifion`
* Замыкание `SCL` на `GND` вызывает включение WiFi если это настроено в пункте контроля каналов.
* Снятие и подача питания на устройство

![](https://github.com/martinhol221/SIM800C_ESP8266/blob/master/dashbord/web.jpg)

По прошествии 10 минут WiFi отключается и устройство уходит в энергосберегающий режим потребляя ток 17-22 мА.

## Управление по звонку с вводом DTMF команд

При звонке на устройство модем снимет вызов только с телефона хозяина, после "снятия трубки" будет ожидать ввода команд

* `#0*1234` - вызовет программу `prog0`
* `#1*1234` - вызовет программу `prog1`
* `#2*1234` - вызовет программу `prog2`
* `#3*1234` - вызовет программу `prog3`
* `#555`    - Отправит SMS через 10 сек. 
* `#777`    - Перезагрузит модем. 
* `#888`    - активирует вайфай. 
* `#999`    - Перезагрузит ESP. 

где `1234`     - пинкод для управления   

****
## управление GET-запросом
http://192.168.4.1/sensor?name1=TEMP4&val1=-12.24   // Передача на narodmon.ru значения датчика -12.24 с именем TEMP4  

http://192.168.4.1/set?PROG=0                       // Выполнить программу 0 - (СТОП)

http://192.168.4.1/set?PROG=6                       // Выполнить программу 6 

http://192.168.4.1/set?AT_CMD=ATD%2B37512345678%3B // Звоним на нужный номер +37512345678  

http://192.168.4.1/set?AT_CMD=ATH0                  // Завершаем вызов

http://192.168.4.1/sms?sms_phone=%2B3751234567&sms_text=Trevoga.Datchik1 // отправка sms 


## Выбор MQTT приложения в телефон для управления по GPRS

Для Android
* [Car MQTT Control (специально написанное)](https://github.com/martinhol221/SIM800C_ESP8266?files=1)
* [MQTT Dash Google Play](https://play.google.com/store/apps/details?id=net.routix.mqttdash&hl=ru) рекомендую
* [IoT MQTT Panel](https://play.google.com/store/apps/details?id=snr.lab.iotmqttpanel.prod)
* [IoT MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard)
* [Linear MQTT Dashboard](https://play.google.com/store/apps/details?id=com.ravendmaster.linearmqttdashboard)

Для IOS: 
[IoT OnOff](https://itunes.apple.com/be/app/iot-onoff/id1267226555?mt=8) не на всех версиях ОС, может не коннекстится с брокером

## Выбор MQTT брокера (далее сервера)

**********************************

MQTT broker

https://www.cloudmqtt.com

https://mqtt.4api.ru

https://myqtthub.com

http://flyhub.org

**********************************

**Главное окно программы MQTT Dash** 
![](https://github.com/martinhol221/SIM800L_MQTT/raw/master/other/17.jpg)

* [Дашборд для MQTT Dasch](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/Daschbord.json) альтернативная

**Настройка приложения к серверу**

![](https://github.com/martinhol221/SIM800L_MQTT/raw/master/other/mqtt-9.jpg)

**Загрузка настроек интерфейса**

Открыть в приложении ***Импорт/экспорт*** (1)

Нажать ***Подписаться и ждать метрики*** (2) 

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/setupMqtt.JPG)

В вебсокете https://api.cloudmqtt.com/console/________/websocket  заполнить  и отправить настройки в телефон

![](https://github.com/martinhol221/SIM800C_ESP8266/raw/master/old/UzAAAgCyMeA-960.jpg)

***Topic***  metrics/exchange

Сcылки на конфиг файлы выше, если не понятно смотрим нудное видео без звука 

[Видео по регистрации на youtube.com](https://www.youtube.com/watch?v=xgZZ417HFFQ)

************************************

### Потребление трафика:




Один пакет GPRS данных состоит в среднем из 100 - 110 байт, отправляется устройством каждую минуту, 100 байт х 60 минут х 24 часа х 31 день = около 4.5 MB/мес, однако на практике оператор трафик считают больший, у меня набегает 9.5 Мб в месяц.
![](https://github.com/martinhol221/SIM800C_ESP8266/raw/master/old/normal_log.JPG)

Стоит отметить что сотовые операторы по разному округляют трафик в конце сессии, в результате чего, списывать с баланса могут в разы больше фактически затраченного.

![](https://github.com/martinhol221/SIM800L_MQTT/blob/master/other/trafic.JPG)
