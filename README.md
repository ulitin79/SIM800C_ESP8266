# Анатомия автозапуска 7.0
Anatomy of autorun 7.0

[Анатомия автозапуска 7.0](https://www.drive2.ru/c/518657502959632604/)

**************************

# Выбор MQTT приложения в телефон для управления по GPRS

Для Android

[MQTT Dash Google Play](https://play.google.com/store/apps/details?id=net.routix.mqttdash&hl=ru) recommend

[IoT MQTT Panel](https://play.google.com/store/apps/details?id=snr.lab.iotmqttpanel.prod)

[IoT MQTT Dashboard](https://play.google.com/store/apps/details?id=com.thn.iotmqttdashboard)

[Linear MQTT Dashboard](https://play.google.com/store/apps/details?id=com.ravendmaster.linearmqttdashboard)

Для IOS: 

[IoT OnOff](https://itunes.apple.com/be/app/iot-onoff/id1267226555?mt=8) 

# Выбор MQTT брокера (далее сервера)

***********************************

MQTT broker

https://www.cloudmqtt.com

http://flyhub.org

**********************************

# Настройка приложений сводится к трем этамам:

**Регистрация на сервере

*Cloudmqtt.com > Log in> + Create Nwe Intance > 

`Name`: Любое имя

`Plan`: Cute Cat(Free)

`Tags`: Любое имя

`Data center`: EU-West-1 (Ireland) > Confirm > Create Nwe Intance

**Внесение настроек сервера `Server`,`User`,`Password`,`Port` приложение

**Конфигурация интерфейса приложения (кнопочек, ползунков, индикаторов и др.)

[Видео по регистрации на youtube.com](https://www.youtube.com/watch?v=xgZZ417HFFQ)

************************************

# Готовая приборная панель программы MQTT Dasch, интерфейс (далее daschbord)

![](https://github.com/martinhol221/SIM800C_ESP8266/raw/master/dashbord/MQTT_Dash.jpg)

**Дашборд можно настроить двумя способами:

1. отправить [этот конфиг](https://raw.githubusercontent.com/martinhol221/SIM800C_ESP8266/master/daschbord.txt) в тпик `metrics/exchange`, предварительно подписав на него MQTT Dasch

2. Натроить приложение вручную по принципу, устройство подписано на топики `car/c5/sub` и  `car/c5/sub/webasto`, смартфон на	`car/c5/pub`, `car/c5/pub/gps`, `car/c5/pub/ussd` и `car/c5/pub/rssi`.

запрос `where?` в топик `car/c5/sub`	 возвращает в топик `car/c5/pub/gps` значение 52.834681,21.694131

запрос  `money?` в топик `ar/c5/sub`	 возвращает в топик `car/c5/pub/ussd` значение 2.87rub

запрос  `signal?` в топик `car/c5/sub`	  возвращает в топик `car/c5/pub/rssi` значение	29 (уровень GSM сигнала сети в попугаях)

запрос  `wifion` в топик `car/c5/sub`	  включает на устройстве точку доступ `Webasto_123456` на 10 минут

запрос  `25`  в топик `car/c5/sub/webasto`	  включает реле K1  на 25 минут

запрос  `estart`в топик `car/c5/sub`	      включает реле K4,K1,K2, кратковременно K3, устаноавливает таймер обратного отсчета 30 мин.

запрос  `relay4on` в топик `car/c5/sub`	    включает реле K4  на 10 минут

запрос  `estop` в топик `car/c5/sub`	      отключает все реле

запрос  `ref` в топик `car/c5/sub`	     возвращает все параметры устройства в формате JSON:

`{"pin":[13.73,1,1,0,0,0,1,0],"temp":[14.81,14.50,14.13],"time":[8,2day.18:28:24],"loss":[0,0,0]}`

распарсить строку можно с помощью [JSON pach](https://github.com/json-path/JsonPath)

`$.pin[0]` - вернет напряжение питания устройства (11.73)

`$.pin[1]` - вернет состояние реле `K1` (1) - включенное реле на АСС

`$.pin[2]` - вернет состояние реле `K2` (1) - включенное реле на зажигние

`$.pin[3]` - вернет состояние реле `K3` (0)  

`$.pin[4]` - вернет состояние реле `K4` (0)

`$.pin[5]` - вернет состояние реле `K5` (0)   

`$.pin[6]` - вернет состояние входа `IN1` (0) - состояние педали СТОП

`$.pin[7]` - вернет состояние входа `IN2` (1) - контроль включенного зажигания

`$.temp[0]` - вернет температуру датчика с индексом `0`  (14.81)

`$.temp[1]` - вернет температуру датчика с индексом `1`  (14.50)

`$.temp[2]` - вернет температуру датчика с индексом `2`  (14.13)

`$.time[0]` - вернет значение таймера обратного отсчета `8` минут 

`$.time[1]` - вернет время работы устройства `2day.18:28:24`

секция `loss` - это контроль пропуска коннекта модема и сети, для сбора статистики стабильности.


********

**Управления по звонку:**

При звонке с номера 1 происходит "снятие трубки" и ожидание ввода DTMF команд:

* `1*XXXX` - запускает двигатель на 10 минут и "вешает трубку"

* `2*XXXX` - запускает вебасто   на 30 минут и "вешает трубку"

* `#` - затирает ошибочно введенный символ

* `0` - останавливает все и "вешает трубку"

где `XXXX` пинкод задаваемый в настройках

При звонке с номера 2 происходит сброс вызова и запуск двигатель на 10 минут

***********

