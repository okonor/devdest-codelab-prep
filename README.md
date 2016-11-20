# IoT/Firebase Codelab — Smart Office Chair

Пройдя данный codelab вы научитесь основам программирования Arduino микроконтроллеров и поймете как использовать связку Arduino+Firebase для быстрого прототипирования IoT решений. Нас ждет увлекательная работа – соединение электронных компонентов, программирование микроконтроллера, подключение/настройка Firebase и разработка веб-страницы!

Для прохождения codelab вам понадобится ноутбук с MacOS (10.11+) или Windows (7+) и следующие хардварные компоненты:

* Обычное [офисное кресло](http://www.gadgetreview.com/wp-content/uploads/2015/10/best-office-chair-herman-aeron.jpg) или [офисный стул](http://www.lekas74.ru/images/rss/standart_stul.jpg).
* Плата [WioLink](https://www.seeedstudio.com/Wio-Link-p-2604.html) (в комплекте с кабелем USB-microUSB).
* [Grove Led Bar v2.0](https://www.seeedstudio.com/Grove-LED-Bar-v2.0-p-2474.html) (в комплекте с кабелем Grove).
* [Grove Ultrasonic Ranger](https://github.com/Seeed-Studio/Grove_Ultrasonic_Ranger) (в комплекте с кабелем Grove).
* Моток изоленты, лучше всего [синей](http://www.n-sb.ru/uploads/product/sinyaya-izolenta-6000-v-15mm-h-10m-zubr-master-1233-73_z01.jpg)!

## Небольшая история и постановка задачи

Офисное кресло является предметом, с которым человек взаимодействует гигантсткое количество времени. При этом в наши дни стремительно развития «Интернета вещей» наши офисные кресла по-прежнему остаются «глупыми». Алексей Коровянский и Евгений Комаров решили это исправить и в рамках [Mobilatorium-а](mobilatorium.org) сделали прототип IoT-решения для умного офисного кресла. Данный эксперимент лег в основу нашего codelab-а.

Ключевой особенностью умного офисного кресло является отображение «состояние здоровья» пользователя. Для этого используется LED Bar и простая аналогия с зарядом батарейки телефона. Если пользователь сидит на кресле, то «уровень здоровья» уменьшается, разряжаясь полностью за N минут. Если пользователь встал с кресла, то «здоровье» начинает увеличиваться, заряжаясь полностью за M минут.

Читая дальнейший текст README и выполняя предложенные **ЗАДАЧИ**, вы шаг за шагом реализуете:

0. Пару разминочных задачек по работе с Arduino.
0. Программно-аппаратное решение для умного кресла.
0. Умное кресло взаимодействующее с Firebase.
0. Умное кресло взаимодействующее с веб-страницей.

Если на данный момент, вы никогда всерьез не работали с Arduino или Firebase — не волнуйтесь, мы начнем с самых простых вещей и будем очень плавно двигаться вперед, более того вы в первую секунду почувствуете, что WioLink и Firebase являются очень простыми и мощными средствами. 

Код лаб рассчитан на 60-90 минут. Если вы справитесь быстрее, мы будем рады увидеть ваши предложения по дальнейшему развитию идеи умного офисного кресла. Пара идей которые у нас уже есть:

0. Если пользователь очень долго сидит в «красной» зоне (когда уровень здоровья равен нулю), мы можем автоматически отправить SOS-сообщение его другу/коллеге и за дружеской заботы помочь передохнуть и размяться. 
0. Считать среднюю недельную статистику для команды (если вся команда использует умные офисные кресла) — готовы поспорить, что статистика большинства команд будет за пределами «зеленой зоны» (обозначающий здоровый график перерывов и разминок). Выход из «красной» или «желтой» зоны в «зеленую» это отличный челендж для любой команды, которая заботится о здоровье. 

Ждем ваших креативных идей (issue с меткой enhancement), а еще лучше fork-ов и pull request-ов!

Итак, **давайте преступим к работе!**

## Шаг 0. Подготовка окружения и разминка

Первым делом нам нужно [установить драйвер](http://www.silabs.com/products/mcu/pages/usbtouartbridgevcpdrivers.aspx) для работы с WioLink. 

Дальше нам понадобится [PlatformIO](http://platformio.org/get-started), кроссплатформенная IDE на базе текстового редактора Atom, позволяющая программировать под большое количество микроконтроллеров. После установки и запуска PlatformIO необходимо создать новый проект (меню PlatformIO > Initialize or Update PlatformIO Project). В диалоге создания проекта необходимо указать тип платы микроконтроллера и путь по которому будет размещаться наш проект. WioLink основана на процессоре ESP8266, поэтому среди предложенного перечня плат нам подойдет Espressif ESP8266 ESP-12E. После нажатия кнопки Process PlatformIO скачает все необходимые файлы для работы с нашим микропроцессором.

Помимо служебных файлов во вновь созданном проекте будет содержаться папки `lib` и `src`. Папка `lib` является хранилищем библиотек, подключенных к этому проекту, а папка `src` предназначена для кода проекта. Точками входа в Arduino приложение выступают функции `void setup` (разовый запуск настроек) и `void loop` (бесконечный цикл логики программы). Компилятор ищет эти функции во всех файлах в директории `src`. 

Например, мы можем создать файл `main.cpp` в папке `src`, подключить библиотеку <Arduino.h> и объявить функции `setup` и `loop` следующим образом:

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

Для отображения информации с последовательного порта в PlatformIO необходимо включить монитор последовательного порта из меню PlatformIO > Serial Monitor, указав baudrate (в нашем случае 28800) и соответствующий порт (в нашем случае CP2102 USB to UART Bridge controller).

**ЗАДАЧА:** Реализовать программу, выводящую на консоль "Devfest Siberia 2016 rulezzzz" ~ раз в 1000 миллисекунд. Если вы справились с этой задачей значит вы корректно настроили окружение и готовы двигаться вперед к реализации нашего умного кресла!

**Заметка 1:** Периодически при работе с WioLink после прошивки плата «зависает». В этом случае помогает нажатие кнопки reset, расположенной на WioLink. Если ваша программа делает вид, что работает, но ничего не выводит на консоль значит плата зависла и вам нужно нажать кнопку reset.

**Заметка 2:** Вы узнали старый добрый синтаксис? Да, наша программа написана на C++. При этом в Arduino сообществе нет четко определенного code style и каждый пишет как умеет. Вы еще успеете посетовать на это подключая сторонние библиотеки в Шаге 1. А пока настройтесь писать несовершенный код, это же кодлаб, рефакторить будете потом, а сейчас будем творить!

## Шаг 1 - Сборка железа

На WioLink размещены [6 Grove совместимых разъемов](https://raw.githubusercontent.com/SeeedDocument/Wio_Link/master/image/Hardware%20overview.jpg). Из них 3 цифровых (Digital), 1 аналоговый (Analog), 1 UART и 1 I2C. Рядом с разъемами подписаны пины контроллера, подключенные к этим разъемам, а также пины к которым подключены 3.3 вольта и земля. К разъемам подключаются датчики и исполнительные механизмы.

Для отображения «уровня здоровья» мы будем использовать [Grove Led Bar](https://www.seeedstudio.com/Grove-LED-Bar-v2.0-p-2474.html). Этот элемент имеет четыре пина: два сигнальных пина (dcki и di), пин для подключения питания (3.3 - 5 В) и пин для подключения земли. Как и любой другой grove совместимый элемент, наш Led Bar легко и надежно подключается к плате с помощью grove кабеля. Если вы воткнете Led в разъем Digital 1, то dcki займет 13 пин контроллера, а di займет 12-ый. 

Для взаимодействия с Led Bar рекомендуем использовать подключаемую [библиотеку](https://github.com/Seeed-Studio/Grove_LED_Bar). Нужно скачать репозиторий библиотеки, распаковать архив и полученную папку перенести в папку lib нашего проекта. 

Подключение библиотеки осуществляется с помощью #include <Grove_LED_Bar.h> и позволяет создавать объекты Grove_LED_Bar с помощью конструктора Grove_LED_Bar(pinDcki, pinDi, direction), где pinDclk, pinDi - номера соответствующих сигнальных пинов, direction - направление загорания диодов (0 - прямое, 1 - обратное). Для инициализации полученного объекта необходимо вызвать метод объекта begin() в функции setup. Для управления LED Bar-ом рекомендуем использовать метод setLevel (float level), значение level может варьироваться от 0 до 10, включая дробные значения. В зависимости от выбранного level будет включаться соответственное количество диодов (от 0 до 10). В случае если указано дробное число, будет меняться яркость свечения последнего диода. 

**ЗАДАЧА:** Реализовать схему, которая на старте зажигает все десять индикаторов LED Bar-а и дальше постепенно гасит все индикаторы (10, 9, 8 ... 0). Если вам захочется перезапустить программу вы можете воспользоваться кнопкой reset.

Дальше нам нужно добавить бизенс-логику нашего умного кресла. Зарядка и разрядка «здоровья» должна осуществляться за счет определения сидим мы сейчас на стуле или нет. Для этого нам понадобится ультразвуковой дальномер [Grove Ultrasonic Ranger](https://www.seeedstudio.com/Grove-Ultrasonic-Ranger-p-960.html). Этот датчик имеет три функциональных пина: цифровой сигнальный пин, пин для подключения питания (3.3 - 5 В) и пин для подключения земли. Оставшийся пин Groove разъема не используется. 

Библиотека - https://github.com/Seeed-Studio/Grove_Ultrasonic_Ranger: подключаем командой #include <Ultrasonic.h>, создаем  объект класса Ultrasonic с помощью конструктора Ultrasonic(pinUltrasonicRangeFinder) и измеряем расстояние за счет метода MeasureInCentimeters().

**Задача:** Реализовать схему умного офисного кресла. Когда мы сидим на кресле «уровень здоровья» должен уменьшаться, разряжаясь полностью за 60 секунд. Если мы встали с кресла, то «здоровье» должно увеличиваться, заряжаясь полностью за 20 секунд. Схема должна быть собрана на реальном стуле и работать в боевых условиях. Для крепления датчиков и платы используйте синюю изоленту. Когда вы получите работающую версию, поаплодируйте себе, ведь теперь вы умеете легко и просто собирать умные контроллеры!

## Шаг 2 - Подключение к WiFi и Firebase

У нас уже есть умное офисное кресло, но пока что она не подключено к возможностям "Интернета вещей". Нашей конечной целью является отображение «здоровья» и управление параметрами N (время разрядки) и M (время разрядки) с помощью веб-страницы. Также подключение кресло к Интернету вещей открывает нам бесконечные просторы для творчества, например, мы можем считать среднее здоровье за день или за неделю, среднее для команды, или автоматически отправлять SOS-сообщение другу в мессенджер, если кто-то сидит с самого утра без перерыва и злоупотребляет сидячкей. Мы оставляем реализация данных возможностей за пределами текущей версии нашего кодлаб, и рассчитываем увидеть ваши пул-реквесты с креативными идеями по дальнейшему развитию проекта.

Плата WioLink основана на микропроцессоре esp8266 c встроенным WiFi модуль, для работы с ним нам потребуется добавить библиотеку ESP8266WiFi к проекту. Эта библиотека загрузилась PlatformIO вместе с инструментарием для разработки под esp8266. Она является глобальной, то есть нет необходимости ее добавлять в каждый проект. Достаточно одной команды #include <ESP8266WiFi.h>.

WiFi инициализируется с помощью статического метода WiFi.begin(WIFI_SSID, WIFI_PASSWORD), его статус можно проверить с помощью проверки WiFi.status() == WL_CONNECTED. В случае истинности выражения можно продолжать выполнение программы дальше.

**ЗАДАЧА:** Подключить наше умное кресло к доступной сети WiFi.

Мы будем использовать Firebase в качестве realtime базы данных для нашего проекта. Перейдя по ссылке https://firebase.google.com/ и авторизируясь под своим Google аккаунтом создадим новым проект. 

Для взаимодействия с Firebase базой данных мы будем использовать [библиотеку](https://github.com/googlesamples/firebase-arduino). Подключаем ее с помощью #include <FirebaseArduino.h>. Инициализирование Firebase происходит с помощью вызова статического метода Firebase.begin(FIREBASE_HOST, FIREBASE_AUTH). FIREBASE_HOST можно посмотреть на странице Firebase Admin SDK (шестеренка рядом с названием проекта > Настройки проекта > Сервисные аккаунты > Firebase Admin SDK > databaseURL). FIREBASE_AUTH можно найти в секретах базы данных (шестеренка рядом с названием проекта > Настройки проекта > Сервисные аккаунты > Секреты базы данных).

После успешной инициализации Firebase клиента мы можем читать значения из базы данных (например, метод Firebase.getInt("path/to/variable_name") позволяет читать целые значения) и писать значения в базу (например, метод Firebase.setFloat("path/to/variable_name", variable_value) позволяет записать вещественное значение).

**Задача:** Создать новый Firebase проект. В базе данных проекта создать две переменных: time_to_discharge равную 10-ти секундам и time_to_recharge равную 1-й секунде. После этого обновить метод loop таким образом, чтобы значение параметров time_to_discharge и time_to_recharge считывалось из Firebase и динамически применялось к работе нашего приложения, а значение «здоровья» синхронизировалсь со значением переменной health в Firebase. Постарайтесь сделать так, чтобы в случае получения новых значений параметров (time_to_discharge или time_to_recharge) состояние кресла мгновенно перезагружалось — "здоровье" сбрасывалось в максимум, а дальнейшее поведение "на лету" подстраивались под обновленные значения.

Заметка 1: При сборке проекта с этой библиотекой компилятор будет ругаться на отсутствие файлов  ESP8266HTTPClient.h и ESP8266WebServer. Причина в том, что Platform IO добавляет на этапе прекомпиляции только те библиотеки на которые есть прямые ссылки из файлов из папки src. Поэтому достаточно добавить #include <ESP8266HTTPClient.h> и #include <ESP8266WebServer.h> до подключения библиотеки FirebaseArduino.

Заметка 2: Если последний вызванный Firebase метод завершился с ошибкой, метод Firebase.failed() - вернет true, в противном случае false. String переменную с описание ошибки можно получить вызовом метода Firebase.error().

Мы верим, что у вас получилось и перед вами находится уникальное творение! Умное офисное кресло, которое показывает «уровень здоровья» (локально и глобально) и конфигурируется через Firebase.

## Шаг 3 - Создание веб-страницы

Теперь мы создадим веб-страницу, отображающую «уровень здоровья» и позволяющую менять знанение параметров time_to_discharge и time_to_recharge. Для этого нам нужно сверстать страницую, обновить настройки Firebase приложения и реализовать взаимодействие с Firebase средствами javascript.

С целью экономия времени верстку страницы можете взять по [ссылке](https://drive.google.com/a/handsome.is/file/d/0B_f_gRIwcIw5WTdHYUFjMUFCNGM/view?usp=sharing). В ней нас интересует три элемента:
* document.getElementById('health-bar') : меняя стиль *height* этого объекта мы сможет отображать текущий "уровень зоровья"

```
document.getElementById('health-bar').style.height = progress + '%'; // progress от 0 до 100
```

* document.getElementById('timeToDischarge') : меня *value* этого объекта мы сможем считывать введение в input поле значение длительности "разрядки"" или устанавливать его
```
var someVar = document.getElementById('timeToDischarge').value; // считываем значение
document.getElementById('timeToDischarge').value = someVar; // устанавливаем значение
```

* document.getElementById('timeToRecharge') : аналогично document.getElementById('timeToDischarge') управляет временем «зарядки»

Логику веб страницы будем писать на Javascript. Для доcтупа к функциям Firebase на нашей странице необходимо зайти на главную страницу вашего Firebase проекта и нажать "Добавьте Firebase в свое веб-приложение". После этого скопируйте предложенный текст в конце сверстанного документа.

Первое, что необходимо сделать, это добавить пользователя для чтения и записи из базы данных. Для этого на странице управления Firebase проектом зайте в закладу Authentication, нажмите "Добавить пользователя" и введите произвольный почтовый адрес и пароль (может потребоваться перейти на страницу "Способ входа" и включить авторизацию с помощь адреса электронной почты/пароля).

Далее вам необходимо после строчки *firebase.initializeApp(config);* добавить вызов метода, указав данные ранее созданного пользователя: 
```
firebase.auth().signInWithEmailAndPassword(email, password).catch(function(error) {
  // Handle Errors here.
  var errorCode = error.code;
  var errorMessage = error.message;
  // ...
});
```

Для взаимодействия с Firebase базой данных используются Javascript объекты типа Reference. Они создаются с помощью метода firebase.database().ref('/path/to') и указывают на конкретную ветвь в базе данных. К примеру
```
var refToSomewhere = firebase.database().ref('/path/to')
```
создаст объект типа Reference, указывающий путь FirebaseDatabaseRoot/path/to

Метод .on у созданного объекта позволяет подписаться на изменение данных по указанной ссылке. 
```
refToSomewhere.on('value', function(snapshot) {
	console.log(snapshot.val());
    });
```
При инициализации этого метода или при изменениях объекта по указанной ссылке произойдет вызов callbackа. При этом в качестве аргумента callback принимает объект типа snapshot. Получить измененное значение можно с помощью метода snapshot.val().


Сохранение производится с помощью метода update(). В качестве аргументов ему передается javascript объект для сохранения. К примеру вызов:
```
refToSomewhere.update(
	{
		time_to_discharge: 20
	}
)
```
создаст по указанной ссылке объект с именем time_to_discharge и значением 20.

**ЗАДАЧА:** Добавить к предложенной верстке логику, позволяющую: устаналивать уровень энергии в элементе health-bar, в зависимости от «уровень здоровья» из базы данных, устанавливать *value* элементов timeToDischarge и timeToСharge при изменении информации в базе данных, а также при нажатии кнопки OK сохранять из соотвественных полей значения timeToDischarge timeToСharge в базу данных.

Заметка 1: Одним из преимуществ Firebase является то, что его можно легко подключить к всевозможным платформам и технологиям — Arduino, Android, iOS, Browser Javascript, Node.js, Unity и многим другим. Поэтому вы можете пойти дальше создания веб-страницы и сделать красивое приложение для вашей любимой операционной системы.



