---
layout: post
title: " API Saures.ru "
date:   2019-04-29
description: "Интегрируем API Saures.ru в  ioBroker"
categories: ioBroker
main-class: 'iobroker'
color: '#EB7728'
tags:
- iobroker
- iob
- иоброкер
- иоб
- Saures
- script 
excerpt:
introduction: "⚙️ Интегрируем API Saures.ru в  ioBroker"
---
# Интеграции с системой SAURES.
### Что такое SAURES ?

**SAURES** - это система из многофункциональных контроллеров и облачного сервиса, автоматизирующая регулярные задачи потребителей коммунальных ресурсов: ежемесячная передача показаний счетчиков, контроль состояния датчиков, защита от протечек и оповещение об аварийных ситуациях. 

Что понадобится нам? Это документация [пользовательского API.pdf][1]

### Отправляем POST и GET запросы.

>Для отправки POST и GET запросов будем использовать программу  [Postman][2].

В теле запроса пропишем емайл и пасворд, в  теле ответа должны увидеть ** sid **.

![][3]

>Программа может сгенерировать код  запроса для разных ЯП ...**NodeJS** **PHP** **bash** и т.д.



### Скрипт в IoBroker.

Ну и сам скрипт, который получает значения через API и сохраняет в обьектах IoBrоker.

{% highlight javascript %}
let request = require('request');//подключаем библиотеку request запросов

createState('Saures.hot_water',0);//Создаем обьекты куда запишем значения из базы Saures
createState('Saures.cold_water',0);// с значениями = 0
createState('Saures.temp_water',0);

let apiUrlLogin= 'https://api.saures.ru/1.0/login';
let apiUrlMeter= 'https://api.saures.ru/1.0/object/meters',
sid;
let optionsPost = { //опции для Post запроса 
 url:apiUrlLogin,
 headers: {"Content-Type": " application/x-www-form-urlencoded; charset=utf-8"},
 formData: { email: 'demo@saures.ru', password: 'demo' }// логин с паролем
};

 
function getSid(error, response, body) {
  if (!error && response.statusCode == 200) {
    var info = JSON.parse(body);
    sid = info.data.sid
    console.log(" Sid is - "+sid );
    let optionsGet = {
    url: apiUrlMeter,
    qs: { sid: sid, id: '358' },   //опции для Get запроса 
}
    setTimeout(() => {request.get(optionsGet, getMeter)}, 1000);//Get запрос данных
  }
};
function getMeter(error, response, body) {
    if (!error && response.statusCode == 200  ) {
      var info = JSON.parse(body);
     console.log(info.data.sensors[2].meters[6].vals[0]); //раскоментировать для получения JSON данных
      console.log(" Статус - "+info.status );
      let HotW = info.data.sensors[2].meters[0].vals[0] ;//записываем значения в переменные
      let ColdW = info.data.sensors[2].meters[1].vals[0] ;
      let tempW = info.data.sensors[2].meters[2].vals[0] ;
      setState("javascript.0.Saures.hot_water", HotW, true);//записываем значения в созданные объекты
      setState("javascript.0.Saures.cold_water", ColdW, true);
      setState("javascript.0.Saures.temp_water", tempW, true);
    }
  };

// schedule("* */12 * * *",  () => {
//   request.post(optionsPost, getSid);
// }); // раскомментировать крон для отправки запросов каждые 12 часов

request.post(optionsPost, getSid); 

{% endhighlight %}

### Редактируем скрипт под свои задачи.

Для удобства распарсивания данных, которые приходят с сервера, воспользуемся 
[JSONeditorOnline][5]

в функции **getMeter** для получения данных раскомментируем строку

{% highlight javascript %}
//console.log(body); //раскомментировать для получения JSON данных
{% endhighlight %}

Полученный код необходимо вставить в левую панель , а в правой смотрим путь для нужного значения данных.
{% highlight javascript %}
 let HotW = info.data.sensors[0].meters[0].value
{% endhighlight %}
 
 ![][6]

Заменить **flat_id: '385'** на свой, который можно получить Get запросом на 
 https://lk.saures.ru/api/company/flats 

Поменялось **API** 
https://api.saures.ru/doc/

{% highlight javascript %}
qs: { sid: sid, flat_id: '385' },   //опции для Get запроса 
{% endhighlight %}

Ну конечно же свой логин и пароль.

{% highlight javascript %}
formData: { email: 'demo@saures.ru', password: 'demo' }// логин с паролем
{% endhighlight %}


### Итог.

В итоге получаем значения в обьектах **IoBroker**, которые будут обновляться один раз в 12 часов.

![][7]

[1]: https://www.saures.ru/upload/iblock/efd/SAURES-API-polzovatelskoe-v.3.12.pdf
[2]: https://web.postman.co
[3]: /assets/image/Postman_code.png
[5]: https://jsoneditoronline.org/
[6]: /assets/image/JSON_Editor.jpg
[7]: /assets/image/ObjSaures.png
