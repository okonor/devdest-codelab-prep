# IoT/Firebase Codelab — Smart Office Chair

Пройдя данный codelab вы научитесь основам программирования Arduino микроконтроллеров и поймете как использовать связку WioLink+Firebase для быстрого прототипирования IoT решений. Нас ждет увлекательная работа – соединение электронных компонентов, программирование микроконтроллера, подключение/настройка Firebase и разработка веб-страницы!

Для прохождения codelab вам понадобится ноутбук с MacOS (10.11+) или Windows (7+) и следующие хардварные компоненты:

* Обычное [офисное кресло](http://www.gadgetreview.com/wp-content/uploads/2015/10/best-office-chair-herman-aeron.jpg) или [офисный стул](http://www.lekas74.ru/images/rss/standart_stul.jpg).
* Плата [WioLink](https://www.seeedstudio.com/Wio-Link-p-2604.html) (в комплекте с кабелем USB-microUSB).
* [Grove Led Bar v2.0](https://www.seeedstudio.com/Grove-LED-Bar-v2.0-p-2474.html) (в комплекте с кабелем Grove).
* [Grove Ultrasonic Ranger](https://github.com/Seeed-Studio/Grove_Ultrasonic_Ranger) (в комплекте с кабелем Grove).
* Моток изоленты, лучше всего [синей](http://www.n-sb.ru/uploads/product/sinyaya-izolenta-6000-v-15mm-h-10m-zubr-master-1233-73_z01.jpg)!

## Небольшая история и постановка задачи

Офисное кресло является предметом, с которым человек взаимодействует гигантсткое количество времени. При этом в наши дни стремительно развития «Интернета вещей» наши офисные кресла по-прежнему остаются «глупыми». [Алексей Коровянский](https://github.com/AlexKorvyansky) и [Евгений Комаров](https://github.com/opengamer29) решили это исправить и в рамках [Mobilatorium-а](https://mobilatorium.org) сделали прототип IoT-решения умного офисного кресла. Этот эксперимент лег в основу нашего codelab-а.

Ключевой особенностью умного офисного кресло является отображение «состояние здоровья» пользователя. Для этого используется LED Bar и простая аналогия с зарядом батарейки телефона. Если пользователь сидит на кресле, то «уровень здоровья» уменьшается, разряжаясь полностью за N минут. Если пользователь встал с кресла, то «здоровье» начинает увеличиваться, заряжаясь полностью за M минут.

Читая дальнейший текст **README** и выполняя предложенные **ЗАДАЧИ**, вы шаг за шагом реализуете:

0. Пару разминочных задачек по работе с Arduino.
0. Программно-аппаратное решение для умного кресла.
0. Умное кресло взаимодействующее с Firebase.
0. Умное кресло взаимодействующее с веб-страницей.

Если вы прежде не работали с Arduino или Firebase — не волнуйтесь, мы начнем с самых простых вещей и будем очень плавно двигаться вперед, более того вы в первую секунду почувствуете, что WioLink и Firebase являются очень простыми и мощными средствами.

Codelab рассчитан на 60-90 минут. Если вы справитесь быстрее, мы будем рады увидеть ваши предложения по дальнейшему развитию идеи умного офисного кресла. Пара идей, которые у нас уже есть:

0. Если пользователь очень долго сидит в «красной» зоне (когда уровень здоровья равен нулю), мы можем автоматически отправить SOS-сообщение его другу/коллеге и за дружеской заботы помочь передохнуть и размяться. 
0. Считать среднюю недельную статистику для команды (если вся команда использует умные офисные кресла) — готовы поспорить, что статистика большинства команд будет за пределами «зеленой зоны» (обозначающий здоровый график перерывов и разминок). Выход из «красной» или «желтой» зоны в «зеленую» это отличный челендж для любой команды, которая заботится о здоровье. 

Ждем ваших креативных идей (issue с меткой enhancement), а еще лучше fork-ов и pull request-ов!

Итак, **давайте приступим к работе!**

## Шаг 1. Подготовка окружения и разминка

Первым делом нам нужно [установить драйвер](http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx) для работы с WioLink. 

Дальше нам понадобится [PlatformIO](http://platformio.org/get-started), кроссплатформенная IDE на базе текстового редактора Atom, позволяющая программировать под большое количество микроконтроллеров. После установки и запуска PlatformIO необходимо создать новый проект (меню PlatformIO > Initialize or Update PlatformIO Project). В диалоге создания проекта необходимо указать тип платы микроконтроллера и путь по которому будет размещаться наш проект. WioLink основана на процессоре ESP8266, поэтому среди предложенного перечня плат нам подойдет Espressif ESP8266 ESP-12E. После нажатия кнопки Process PlatformIO скачает все необходимые файлы для работы с нашим микропроцессором.

Помимо служебных файлов во вновь созданном проекте будет содержаться папки `lib` и `src`. Папка `lib` является хранилищем библиотек, подключенных к этому проекту, а папка `src` предназначена для кода проекта. Точками входа в Arduino приложение выступают функции `void setup` (разовый запуск настроек) и `void loop` (бесконечный цикл логики программы). Компилятор ищет эти функции во всех файлах в директории `src`. 

Например, мы можем создать файл `main.cpp` в папке `src`, подключить библиотеку `<Arduino.h>` и объявить функции `setup` и `loop` следующим образом:

```C++
#include <Arduino.h>

void setup() {
	// инициализация Serial порта c baudrate равным 28800
  Serial.begin(28800)
  
  // Для включения GPIO портов на плате WioLink необходимо подать высокий сигнал на 15-ый пин микропроцессора.
	pinMode(15, OUTPUT);
	digitalWrite(15, HIGH);
}

void loop() {
  Serial.println("Hello World! Devfest Siberia 2016 rulezzzz");
  delay(1000);
}

```

Для того, чтобы запустить данную программу на микроконтроллере, нужно подключить его к компьютеру с помощью кабеля USB-microUSB и залить прошивку на контроллер (с помощью меню PlatformIO > Upload или [соответствующей кнопки на тулбаре](https://cl.ly/221R3C022c34/Image%202016-11-20%20at%209.03.04%20AM.png)).

Для отображения информации с последовательного порта в PlatformIO необходимо включить монитор последовательного порта из меню PlatformIO > Serial Monitor, указав baudrate (в нашем случае `28800`) и соответствующий порт (в нашем случае `CP2102 USB to UART Bridge controller`).

**ЗАДАЧА 1.1**: Реализовать программу, выводящую на консоль "Devfest Siberia 2016 rulezzzz" ~ раз в 1000 миллисекунд. Если вы справились с этой задачей значит вы корректно настроили окружение и готовы двигаться вперед к реализации нашего умного кресла!</span>

**Заметка:** Периодически при работе с WioLink после прошивки плата «зависает». В этом случае помогает нажатие кнопки reset, расположенной на WioLink. Если ваша программа делает вид, что работает, но ничего не выводит на консоль значит плата зависла и вам нужно нажать кнопку reset.

**Заметка:** Вы узнали старый добрый синтаксис? Да, наша программа написана на C++. При этом в Arduino сообществе нет четко определенного code style и каждый пишет как умеет. Вы еще успеете посетовать на это подключая сторонние библиотеки в Шаге 2. А пока настройтесь писать несовершенный код, это же codelab!

## Шаг 2. Сборка железа

На WioLink размещены [6 Grove совместимых разъемов](https://raw.githubusercontent.com/SeeedDocument/Wio_Link/master/image/Hardware%20overview.jpg). Из них 3 цифровых (Digital), 1 аналоговый (Analog), 1 UART и 1 I2C. Рядом с разъемами подписаны пины контроллера, подключенные к этим разъемам, а также пины к которым подключены питание (3v3) и земля (GND). Например, разъем Digital 0 включает в себя 14-й и 12-й пины, питание и землю.

Для отображения «уровня здоровья» мы будем использовать [Grove LED Bar](https://www.seeedstudio.com/Grove-LED-Bar-v2.0-p-2474.html). Этот элемент имеет четыре пина: два сигнальных пина (DCKI и DI), пин VCC для подключения питания (поддерживается работа в диапазоне 3.3V - 5V) и пин GND для подключения земли. Как и любой другой Grove совместимый элемент, наш Led Bar легко и надежно подключается к плате с помощью Grove кабеля. Если вы воткнете Led в разъем Digital 1, то dcki займет 13 пин контроллера, а di займет 12-ый. 

Для взаимодействия с LED Bar мы рекомендуем использовать библиотеку https://github.com/Seeed-Studio/Grove_LED_Bar. Нужно скачать репозиторий библиотеки, распаковать архив и полученную папку перенести в `lib` нашего проекта. 

Подключение библиотеки осуществляется строкой `#include <Grove_LED_Bar.h>`. После этого мы можем создавать объект класса `Grove_LED_Bar` с помощью конструктора `Grove_LED_Bar(pinDcki, pinDi, direction)`, где `pinDclk`, `pinDi` - номера соответствующих сигнальных пинов, `direction` - направление загорания диодов (`0` - прямое, `1` - обратное). Также в функции `setup` у объекта `Grove_LED_Bar` необходимо вызывать метод `begin()`. 

Для управления LED Bar-ом рекомендуется использовать метод `setLevel (float level)`. Значение `level` может варьироваться от 0 до 10, включая дробные значения. В зависимости от выбранного `level` будет включаться соответственное количество диодов (от 0 до 10). В случае если указано дробное число, будет меняться яркость свечения последнего диода. 

**ЗАДАЧА 2.1:** Реализовать схему, которая на старте зажигает все десять индикаторов LED Bar-а и дальше постепенно гасит все индикаторы.

**Заметка:** Для перезапуска программы можно использовать кнопку Reset.

Теперь мы можем перейти к реализации бизенс-логики нашего умного кресла. Зарядка и разрядка «здоровья» должна осуществляться за счет определения сидим мы сейчас на стуле или нет. Для этого нам понадобится ультразвуковой дальномер [Grove Ultrasonic Ranger](https://www.seeedstudio.com/Grove-Ultrasonic-Ranger-p-960.html). Этот датчик имеет три функциональных пина: цифровой сигнальный пин (SIG), пин VCC для подключения питания (поддерживается работа в диапазоне 3.3V - 5V) и пин GND для подключения земли. Оставшийся пин (NC) Groove разъема не используется. 

Для взаимодействия с Ultrasonic Ranger мы рекомендуем использовать библиотеку https://github.com/Seeed-Studio/Grove_Ultrasonic_Ranger. Подключение осуществляется строкой `#include <Ultrasonic.h>`, после этого мы можем создать объект класса `Ultrasonic` с помощью конструктора `Ultrasonic(pinUltrasonicRangeFinder)` и измерять расстояние методом `MeasureInCentimeters()`.

**ЗАДАЧА 2.2:** Реализовать схему умного офисного кресла. Когда мы сидим на кресле «уровень здоровья» должен уменьшаться, разряжаясь полностью за 60 секунд. Если мы встали с кресла, то «здоровье» должно увеличиваться, заряжаясь полностью за 20 секунд. Схема должна быть собрана на реальном стуле и работать в боевых условиях. Для крепления датчиков и платы используйте синюю изоленту.

Когда вы получите работающую версию, поаплодируйте себе, ведь теперь вы знаете как легко и просто собирать умные контроллеры!

## Шаг 3. Подключение к WiFi и Firebase

У нас уже есть умное офисное кресло, но оно еще не подключено к возможностям "Интернета вещей". Для того, чтобы это исправить нам нужно научиться подключать нашу плату к WiFi сети.

Плата WioLink имеет встроенный WiFi модуль, для работы с ним нам потребуется добавить библиотеку ESP8266WiFi к проекту. Эта библиотека загрузилась PlatformIO вместе с инструментарием для разработки под esp8266. Она является глобальной, поэтому достаточно просто подключить ее строкой `#include <ESP8266WiFi.h>`.

WiFi инициализируется с помощью статического метода `WiFi.begin(WIFI_SSID, WIFI_PASSWORD)` для подключения к закрытой сети, или метода `WiFi.begin(WIFI_SSID)` для работы с открытой сетью. Подключение происходит асинхронно, для проверки успешности подключения необходимо использовать сравнение вида `WiFi.status() == WL_CONNECTED`.

**ЗАДАЧА 3.1:** Подключить наше умное кресло к доступной сети WiFi.

Теперь наша плата подключается к WiFi и Интернету, что открывает перед нами бесконечный простор для творчества. Следующим шагом нам нужно научиться сохранять в Cloud текущее значение «здоровья», и перенести туда же управление параметрами разрядки и зарядки. 

В качестве облачного хранилища мы будем использовать [Firebase Realtime Database](https://firebase.google.com/docs/database/), как бесплатный и простой NoSQL Сloud Storage отлично подходящий для быстрого прототипирования пользовательских IoT решений.

Для взаимодействия между WioLink и Firebase базой данных мы будем использовать библиотеку https://github.com/googlesamples/firebase-arduino. Подключение осуществляется строкой `#include <FirebaseArduino.h>`. Инициализирование Firebase происходит с помощью вызова статического метода Firebase.begin(`FIREBASE_HOST`, `FIREBASE_AUTH`).

`FIREBASE_HOST` можно посмотреть на странице Firebase Admin SDK (шестеренка рядом с названием проекта > Project Settings > Service Accounts > Firebase Admin SDK > databaseURL). `FIREBASE_AUTH` можно найти в секретах базы данных (шестеренка рядом с названием проекта > Project Settings > Service Accounts > Database Secrets).

После успешной инициализации Firebase клиента мы можем работать с БД. Метод `Firebase.getInt(name)` позволяет читать целые значения, а метод `Firebase.setFloat(name, value)` позволяет записывать вещественные значения.

**ЗАДАЧА 3.2:** Создать новый Firebase проект. В базе данных создать три поля: `health` с произвольным вещественным значением, `time_to_discharge` равную 10-ти секундам и `time_to_recharge` равную 1-й секунде. После этого обновить логику программы так, чтобы значение параметров `time_to_discharge` и `time_to_recharge` динамически считывалось из Firebase и применялось к работе нашего приложения, а значение «здоровья» в режиме реального времени сохранялось в Firebase в поле `health`. В случае получения новых значений параметров (`time_to_discharge` или `time_to_recharge`) состояние кресла должно перезагружаться — «здоровье» сбрасываться в максимум, а дальнейшее поведение «на лету» подстраиваться под обновленные значения.

**Заметка:** При сборке проекта с этой библиотекой компилятор будет ругаться на отсутствие файлов `ESP8266HTTPClient.h` и `ESP8266WebServer`. Причина в том, что Platform IO добавляет на этапе прекомпиляции только те библиотеки на которые есть прямые ссылки из файлов из папки src. Поэтому достаточно добавить `#include <ESP8266HTTPClient.h>` и `#include <ESP8266WebServer.h>` до подключения библиотеки `FirebaseArduino`.

**Заметка:** Если последний вызванный Firebase метод завершился с ошибкой, метод `Firebase.failed()` - вернет `true`, в противном случае `false`. String переменную с описание ошибки можно получить вызовом метода `Firebase.error()`.

Получилось?! А сколько перерывов и разминок вы сделали за время работы? Если ноль, то встаньте и разомнитесь прямо сейчас! 

## Шаг 4. Создание веб-страницы

Поздравляем! Вы освоили базовые техники работы с IoT и Firebase! 

Теперь давайте добавим вишенку на торте и сделаем веб-страницу, отображающую «уровень здоровья». Для этого нам нужно сделать три шага — сверстать страницу, обновить настройки Firebase приложения и реализовать взаимодействие с Firebase средствами Javascript. 

С целью экономия времени вы можете ускорить первый шаг и [взять готовую верстку](https://drive.google.com/a/handsome.is/file/d/0B_f_gRIwcIw5WTdHYUFjMUFCNGM/view?usp=sharing) страницы. В ней нас интересует кастомный UI компонет для отображения «уровня здоровья», для установки его значения используйте:

```javascript
document.getElementById('health-bar').style.height = progress + '%'; // progress от 0 до 100
```

Для взаимодействия между веб-страницей и Firebase нам потребуется обновить настройки Firebase проекта. В разделе Database > Rules, нужно установить значение `true` для поля `.read`. Это позволит нам успешно считывать «уровня здоровья» и отображать его на веб-странице. 

Теперь нам нужно оживить нашу верстку и отображать корректное значение «уровня здоровья» на веб-страницу. Для этого нам нужно написать несколько строчек Javascript кода. Первым шагом нужно инициализировать Fireabase, для этого идем на главную страницу нашего Firebase проекта, там нажимаем кнопку "Add Firebase to your web app" и копируем предложенный листинг в начало нашего JS-скрипта.

Для отслеживания значения поля в Firebase базе данных нужно получить Reference объект и подипсаться на его изменения, следующим образом:

```javascript
firebase.database().ref('health').on('value', function(snapshot) {
	// этот callback будет вызыватьс каждый раз при изменении поля health
	// для получения значения нужно вызвать snapshot.health
});
```

**ЗАДАЧА 4.1:** Реализовать веб-страницу, отображающую «уровень здоровья» для нашего умного офисного кресла.

**Заметка**: Одним из преимуществ Firebase является то, что его можно легко подключить к всевозможным платформам и технологиям — Arduino, Android, iOS, Browser Javascript, Node.js, Unity и многим другим. Поэтому вы можете легко сделать красивое приложение для вашей любимой платформы.

## Заключение

Поздравляем с успешным прохождением codelab! Теперь вы знаете как легко и просто можно создать собственное решение для умного дома (или офиса) с помощью WioLink/Arduino и Firebase. Будем рады видеть ваш вклад в развитие нашего эксперимента! 

И приглашаем вас [подписаться на ВК-страницу Mobilatorium-а](https://vk.com/mobilatorium), чтобы быть в курсе наших следующих экспериментов с IoT/AR/VR.


