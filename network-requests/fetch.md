# ?Fetch

Для сетевых запросов из JavaScript есть широко известный термин «AJAX» (аббревиатура от _Asynchronous JavaScript And XML_).

Есть несколько способов делать сетевые запросы и получать информацию с сервера. `fetch` как один из вариантов.

`Fetch API` создан для работы с запросами и ответами HTTP и позволяет легко и логично получать ресурсы по сети асинхронно. 

* `Promise` возвращаемый вызовом `fetch()` не перейдёт в состояние "отклонено" из-за ответа HTTP, который считается ошибкой, даже если ответ HTTP `404` или `500`. Вместо этого, он будет выполнен нормально (с значением `false` в статусе `ok`) и будет отклонён только при сбое сети или если что-то помешало запросу выполниться.
* По умолчанию, `fetch` не будет отправлять или получать `cookie` файлы с сервера.

Метод `fetch()` — современный и очень мощный, поэтому начнём с него. Он не поддерживается старыми (можно использовать полифил), но поддерживается всеми современными браузерами.

Типичный запрос с помощью `fetch` состоит из двух операторов `await`:

~~~
const response = await fetch(url, options); // завершается с заголовками ответа
const result = await response.json(); // читать тело ответа в формате JSON
~~~

Или, без `await`:

~~~
fetch(url, options)
  .then(response => response.json())
  .then(result => /* обрабатываем результат */)
~~~

* `url` – URL для отправки запроса.
* `options` – дополнительные параметры: метод, заголовки и так далее. Без `options` это простой GET-запрос, скачивающий содержимое по адресу `url`.

Процесс получения ответа обычно происходит в два этапа.

1. `Promise` выполняется с объектом встроенного класса `Response` в качестве результата, как только сервер пришлёт заголовки ответа.

На этом этапе мы можем проверить статус HTTP-запроса и определить, выполнился ли он успешно, а также посмотреть заголовки, но пока без тела ответа.

Промис завершается с ошибкой, если `fetch` не смог выполнить HTTP-запрос, например при ошибке сети или если нет такого сайта. HTTP-статусы `404` и `500` не являются ошибкой.

Мы можем увидеть HTTP-статус в свойствах ответа:

* `response.status` – код статуса HTTP-запроса, например `200`.
* `response.ok` – логическое значение: будет `true`, если код HTTP-статуса в диапазоне `200`-`299`.

2. Для получения тела ответа нам нужно использовать дополнительный вызов метода.

`Response` предоставляет несколько методов, основанных на промисах, для доступа к телу ответа в различных форматах:

* `response.text()` – читает ответ и возвращает как обычный текст.
* `response.json()`– декодирует ответ в формате JSON.
* `response.formData()` – возвращает ответ как объект `FormData` (объект, представляющий данные HTML формы).
* `response.blob()` – возвращает объект как `Blob` (бинарные данные с типом). `Blob` может быть использован как URL для `<a>`, `<img>`  или других тегов, для показа содержимого.
* `response.arrayBuffer()` – возвращает ответ как `ArrayBuffer` (низкоуровневое представление бинарных данных). Мы встречаемся с бинарными данными чаще всего тогда, когда требуется выполнить какие-то действия над файлами (создать, загрузить или скачать) или обработать изображения.
* Помимо этого, `response.body` – это объект `ReadableStream`, с помощью которого можно считывать тело запроса по частям.

Чаще всего используется JSON.

Например, получим JSON-объект с последними коммитами из репозитория на GitHub:

~~~
async function getData() {
  const response = await fetch(
    "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
  );

  if (response.ok) {
    // если HTTP-статус в диапазоне 200-299
    // получаем тело ответа
    const json = await response.json(); // читаем ответ в формате JSON
    console.log(json);
  } else {
    console.log("Ошибка HTTP: " + response.status);
  }
}

getData();
~~~

В качестве примера работы с бинарными данными, давайте запросим и выведем на экран картинку:

~~~
async function getData() {
  const response = await fetch("https://img1.akspic.ru/previews/5/4/5/1/7/171545/171545-graficheskij_dizajner-rabochij_process-dizajn-grafika-dizajner-x750.jpg");

  const blob = await response.blob(); // скачиваем как Blob-объект

  // создаём <img>
  const img = document.createElement("img");
  img.style = "position:fixed; top:10px; left:10px; width:100px";
  document.body.append(img);

  // выводим на экран
  img.src = URL.createObjectURL(blob);

  setTimeout(() => {
    // прячем через три секунды
    img.remove();
    URL.revokeObjectURL(img.src);
  }, 3000);
}

getData();
~~~

> Мы можем выбрать только один метод чтения ответа. Если мы уже получили ответ с `response.text()`, тогда `response.json()` не сработает, так как данные уже были обработаны.

### Заголовки ответа

Заголовки ответа хранятся в похожем на `Map` объекте `response.headers`.

Это не совсем `Map`, но мы можем использовать такие же методы, как с `Map`, чтобы получить заголовок по его имени или перебрать заголовки в цикле:

~~~
async function getData() {
  const response = await fetch(
    "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
  );

  console.log(response.headers.get("Content-Type")); // application/json; charset=utf-8
  for (let [key, value] of response.headers) {
    console.log(`${key} = ${value}`); // cache-control = public, max-age=60, s-maxage=60 ...
  }
}

getData();
~~~

### Заголовки запроса

Для установки заголовка запроса в `fetch` мы можем использовать опцию `headers`. Она содержит объект с исходящими заголовками, например:

~~~
async function getData() {
  const response = await fetch(protectedUrl, {
    headers: {
      Authentication: "secret",
    },
  });
  console.log(response.status);
}

getData();
~~~

### POST-запросы

Для отправки POST-запроса или запроса с другим методом, нам необходимо использовать `fetch` параметры:

* `method` – HTTP метод, например POST,
* `body` – тело запроса, одно из списка:
  * строка (например, в формате JSON),
  * объект `FormData` для отправки данных как `form/multipart`,
  * `Blob/BufferSource` для отправки бинарных данных,
  * `URLSearchParams` для отправки данных в кодировке `x-www-form-urlencoded`, используется редко.

Чаще всего используется JSON.

Например, этот код отправляет объект user как JSON:

~~~
const user = {
  name: "John",
  surname: "Smith",
};

async function createUser() {
  await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json;charset=utf-8",
    },
    body: JSON.stringify(user),
  });
}

createUser(user);
~~~

Заметим, что так как тело запроса `body` – строка, то заголовок `Content-Type` по умолчанию будет `text/plain;charset=UTF-8`.

Но, так как мы посылаем JSON, то используем параметр headers для отправки вместо этого `application/json`, правильный `Content-Type` для JSON.

### Отправка изображения

Мы можем отправить бинарные данные при помощи `fetch`, используя объекты `Blob` или `BufferSource`.

В этом примере есть элемент `<canvas>`, на котором мы можем рисовать движением мыши. При нажатии на кнопку «Отправить» изображение отправляется на сервер:

~~~
<body style="margin:0">
  <canvas id="canvasElem" width="100" height="80" style="border:1px solid"></canvas>

  <input type="button" value="Отправить" onclick="submit()">

  <script type="module">
    canvasElem.onmousemove = function(e) {
      const ctx = canvasElem.getContext('2d');
      ctx.lineTo(e.clientX, e.clientY);
      ctx.stroke();
    };

    async function submit() {
      const blob = await new Promise(resolve => canvasElem.toBlob(resolve, 'image/png'));
      const response = await fetch('/article/fetch/post/image', {
        method: 'POST',
        body: blob
      });

      // сервер ответит подтверждением и размером изображения
      const result = await response.json();
      console.log(result.message);
    }

  </script>
</body>
~~~

Заметим, что здесь нам не нужно вручную устанавливать заголовок `Content-Type`, потому что объект `Blob` имеет встроенный тип (`image/png`, заданный в `toBlob`). При отправке объектов `Blob` он автоматически становится значением `Content-Type`.

### FormData

Здесь речь пойдёт об отправке HTML-форм.

Конструктор:

~~~
const formData = new FormData([form]);
~~~

Если передать в конструктор элемент HTML-формы form, то создаваемый объект автоматически прочитает из неё поля.

Его особенность заключается в том, что методы для работы с сетью, например `fetch`, позволяют указать объект `FormData` в свойстве тела запроса `body`.

Он будет соответствующим образом закодирован и отправлен с заголовком `Content-Type: multipart/form-data`.

~~~
<body>
  <form id="formElem">
    <input type="text" name="name" value="John">
    <input type="text" name="surname" value="Smith">
    <input type="submit">
  </form>

  <script>
    formElem.onsubmit = async (e) => {
      e.preventDefault();

      const response = await fetch(url, {
        method: 'POST',
        body: new FormData(formElem)
      });

      const result = await response.json();

      console.log(result.message);
    };
  </script>
</body>
~~~

С помощью указанных ниже методов мы можем изменять поля в объекте `FormData`:

* `formData.append(name, value)` – добавляет к объекту поле с именем `name` и значением `value`. Технически форма может иметь много полей с одним и тем же именем name, поэтому несколько вызовов append добавят несколько полей с одинаковыми именами. Ещё существует метод `set`, его синтаксис такой же, как у `append`. Разница в том, что `.set` удаляет все уже имеющиеся поля с именем name и только затем добавляет новое. То есть этот метод гарантирует, что будет существовать только одно поле с именем name, в остальном он аналогичен `.append`.
* `formData.append(name, blob, fileName)` – добавляет поле, как будто в форме имеется элемент `<input type="file">`, третий аргумент `fileName` устанавливает имя файла (не имя поля формы), как будто это имя из файловой системы пользователя.
* `formData.delete(name)` – удаляет поле с заданным именем `name`.
* `formData.get(name)` – получает значение поля с именем `name`.
* `formData.has(name)` – если существует поле с именем `name`, то возвращает `true`, иначе `false`.

~~~
<body>
  <form id="formElem">
    <input type="text" name="name" value="John" title="name" />
    <input type="text" name="surname" value="Smith" title="surname" />
    <input type="submit">
  </form>

  <script>
    formElem.onsubmit = async (e) => {
      e.preventDefault();
      const formData = new FormData(formElem);
      formData.append('name', 'John');

      console.log(formData.has('surname')); // true
      console.log(formData.get('surname')); // Smith
      for (let [name, value] of formData) {
        console.log(`${name} = ${value}`); // name = John surname = Smith name = John
      }
    };
  </script>
</body>
~~~

### Ход загрузки

Метод `fetch` позволяет отслеживать процесс получения данных.

Заметим, на данный момент в `fetch` нет способа отслеживать процесс отправки. Для этого используйте `XMLHttpRequest`, позже мы его рассмотрим.

Чтобы отслеживать ход загрузки данных с сервера, можно использовать свойство `response.body`. Это `ReadableStream` («поток для чтения») – особый объект, который предоставляет тело ответа по частям, по мере поступления. Потоки для чтения описаны в спецификации Streams API.

В отличие от `response.text()`, `response.json()` и других методов, `response.body` даёт полный контроль над процессом чтения, и мы можем подсчитать, сколько данных получено на каждый момент.

Вот примерный код, который читает ответ из `response.body`:

~~~
async function getData() {
  const response = await fetch(
    "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits"
  );

  const reader = response.body.getReader();
  
  while (true) {
    const { done, value } = await reader.read();
    console.log(done);
    if (done) {
      break;
    }

    console.log(`Получено ${value.length} байт`);
  }
}

getData();
~~~

Результат вызова `await reader.read()` – это объект с двумя свойствами:

* `done` – `true`, когда чтение закончено, иначе `false`.
* `value` – типизированный массив данных ответа Uint8Array.

### Прерывание запроса

Как мы знаем, метод `fetch` возвращает промис. А в JavaScript в целом нет понятия «отмены» промиса. Как же прервать запрос `fetch`?

Для таких целей существует специальный встроенный объект: `AbortController`, который можно использовать для отмены не только `fetch`, но и других асинхронных задач.

Использовать его достаточно просто:

1. Создаём контроллер.

Контроллер `controller` – чрезвычайно простой объект.

Он имеет единственный метод `abort()` и единственное свойство `signal`.

При вызове `abort()`:

* генерируется событие с именем `abort` на объекте `controller.signal`.
* свойство `controller.signal.aborted` становится равным `true`.

Все, кто хочет узнать о вызове `abort()`, ставят обработчики на `controller.signal`, чтобы отслеживать его.

2. Передайте свойство signal опцией в метод `fetch`.

Метод `fetch` умеет работать с `AbortController`, он слушает событие `abort` на `signal`.

3. Чтобы прервать выполнение `fetch`, вызовите `controller.abort()`.

Вот и всё: `fetch` получает событие из `signal` и прерывает запрос.

Когда `fetch` отменяется, его промис завершается с ошибкой `AbortError`, поэтому мы должны обработать её, например, в `try..catch`:

~~~
async function getData() {
  const controller = new AbortController();
  setTimeout(() => controller.abort(), 10);

  try {
    const response = await fetch(
      "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits",
      {
        signal: controller.signal,
      }
    );
    console.log(response.status);
  } catch (err) {
    if (err.name == "AbortError") {
      // обработать ошибку от вызова abort()
      console.log("Interrupted!");
    } else {
      throw err;
    }
  }
}

getData();

// Interrupted!
~~~

### Запросы на другие сайты

Если мы сделаем запрос `fetch` на другой веб-сайт, он, вероятно, завершится неудачей.

Запросы на другой источник – отправленные на другой домен (или даже поддомен), или протокол, или порт – требуют специальных заголовков от удалённой стороны.

Эта политика называется «CORS»: _Cross-Origin Resource Sharing_ («совместное использование ресурсов между разными источниками»).

С точки зрения браузера запросы к другому источнику бывают двух видов: «простые» и все остальные.

Простые запросы должны удовлетворять следующим условиям:

* Метод: `GET`, `POST` или `HEAD`.
* Заголовки – мы можем установить только:
  * `Accept`
  * `Accept-Language`
  * `Content-Language`
  * `Content-Type` со значением `application/x-www-form-urlencoded`, `multipart/form-data` или `text/plain`.

Основное их отличие заключается в том, что простые запросы с давних времён выполнялись с использованием тегов `<form>` или `<script>`, в то время как непростые долгое время были невозможны для браузеров.

Практическая разница состоит в том, что простые запросы отправляются сразу с заголовком `Origin`, а для других браузер делает предварительный запрос, спрашивая разрешения.

Для простых запросов:

* Браузер посылает заголовок `Origin` с источником.

Например, если мы запрашиваем `https://anywhere.com/request` со страницы `https://javascript.info/page`, заголовки будут такими:

~~~
GET /request
Host: anywhere.com
Origin: https://javascript.info
... 
~~~

* Для запросов без авторизационных данных (не отправляются по умолчанию) сервер должен установить:
  * `Access-Control-Allow-Origin` в `*` или то же значение, что и `Origin`
* Для запросов с авторизационными данными сервер должен установить:
  * `Access-Control-Allow-Origin` в то же значение, что и `Origin`
  * `Access-Control-Allow-Credentials` в `true` // Если сервер согласен принять запрос с куками и заголовками HTTP-аутентификации.

Дополнительно, чтобы разрешить JavaScript доступ к любым заголовкам ответа, кроме `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified` или `Pragma`, сервер должен перечислить разрешённые в заголовке `Access-Control-Expose-Headers`.

Для непростых запросов перед основным запросом отправляется предзапрос. Предзапрос осуществляется «за кулисами», невидимо для JavaScript:

~~~
const response = await fetch('https://site.com/service.json', {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    'API-Key': 'secret'
  }
});
~~~

1. Браузер посылает запрос OPTIONS на тот же адрес с заголовками:

~~~
OPTIONS /service.json
Host: site.com
Origin: https://javascript.info
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: Content-Type,API-Key
~~~

* Метод: OPTIONS.
* Путь – точно такой же, как в основном запросе: `/service.json`.
* Особые заголовки:
  * `Origin` – источник.
  * `Access-Control-Request-Method` – запрашиваемый метод.
  * `Access-Control-Request-Headers` – разделённый запятыми список «непростых» заголовков запроса.

2. Сервер должен ответить со статусом `200` и заголовками:

~~~
200 OK
Access-Control-Allow-Methods: PUT,PATCH,DELETE
Access-Control-Allow-Headers: API-Key,Content-Type,If-Modified-Since,Cache-Control
Access-Control-Max-Age: 86400
~~~

* `Access-Control-Allow-Methods` со списком разрешённых методов,
* `Access-Control-Allow-Headers` со списком разрешённых заголовков,
* `Access-Control-Max-Age` с количеством секунд для кеширования разрешений

3. Затем отправляется основной запрос, применяется предыдущая «простая» схема.

4. Далее основной ответ. Сервер не должен забывать о добавлении `Access-Control-Allow-Origin` к ответу на основной запрос. Успешный предзапрос не освобождает от этого:

~~~
Access-Control-Allow-Origin: https://javascript.info
~~~

### Fetch API

Нижеследующий список – это все возможные опции для `fetch` с соответствующими значениями по умолчанию (в комментариях указаны альтернативные значения):

~~~
const promise = fetch(url, {
  method: "GET", // POST, PUT, DELETE, etc.
  headers: {
    // значение этого заголовка обычно ставится автоматически,
    // в зависимости от тела запроса
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined, // string, FormData, Blob, BufferSource или URLSearchParams
  referrer: "about:client", // или "" для того, чтобы не послать заголовок Referer,
  // или URL с текущего источника
  referrerPolicy: "strict-origin-when-cross-origin", // no-referrer-when-downgrade, no-referrer, origin, same-origin...
  mode: "cors", // same-origin, no-cors
  credentials: "same-origin", // omit, include
  cache: "default", // no-store, reload, no-cache, force-cache или only-if-cached
  redirect: "follow", // manual, error
  integrity: "", // контрольная сумма, например "sha256-abcdef1234567890"
  keepalive: false, // true
  signal: undefined, // AbortController, чтобы прервать запрос
});
~~~

Мы уже разобрали параметры `method`, `headers` и `body`.

Опция `signal` также разъяснена выше.

* `referrer`, `referrerPolicy` - определяют, как `fetch` устанавливает HTTP-заголовок `Referer`.

Обычно этот заголовок ставится автоматически и содержит URL-адрес страницы, с которой пришёл запрос. В большинстве случаев он совсем неважен, в некоторых случаях, с целью большей безопасности, имеет смысл убрать или укоротить его.

Опция `referrer` позволяет установить любой `Referer` в пределах текущего источника или же убрать его.

В отличие от настройки `referrer`, которая позволяет задать точное значение `Referer`, настройка `referrerPolicy` сообщает браузеру общие правила, что делать для каждого типа запроса.

* `mode` – это защита от нечаянной отправки запроса на другой источник.

* `credentials` - указывает, должен ли fetch отправлять куки и авторизационные заголовки HTTP вместе с запросом.

* `cache` - позволяет игнорировать HTTP-кеш или же настроить его использование.

* `redirect`. Обычно fetch прозрачно следует HTTP-редиректам, таким как 301, 302 и так далее. Это можно поменять при помощи опции `redirect`.

* `integrity` - позволяет проверить, соответствует ли ответ известной заранее контрольной сумме.

* `keepalive` - указывает на то, что запрос может «пережить» страницу, которая его отправила.

### Длинные опросы

_Длинные опросы_ – это самый простой способ поддерживать постоянное соединение с сервером, не используя при этом никаких специфических протоколов (типа WebSocket или Server Sent Events).

Как это происходит:

1. Запрос отправляется на сервер.
2. Сервер не закрывает соединение, пока у него не возникнет сообщение для отсылки.
3. Когда появляется сообщение – сервер отвечает на запрос, посылая его.
4. Браузер немедленно делает новый запрос.

Соединение прерывается только доставкой сообщений. 

Если соединение будет потеряно, скажем, из-за сетевой ошибки, браузер немедленно посылает новый запрос.

Примерный код клиентской функции `subscribe`, которая реализует длинные опросы:

~~~
function SubscribePane(elem, url) {
  function showMessage(message) {
    const messageElem = document.createElement("div");
    messageElem.append(message);
    elem.append(messageElem);
  }

  async function subscribe() {
    const response = await fetch(url);

    if (response.status == 502) {
      // Таймаут подключения
      // случается, когда соединение ждало слишком долго.
      // давайте восстановим связь
      await subscribe();
    } else if (response.status != 200) {
      // Показать ошибку
      showMessage(response.statusText);
      // Подключиться снова через секунду.
      await new Promise((resolve) => setTimeout(resolve, 1000));
      await subscribe();
    } else {
      // Получить сообщение
      const message = await response.text();
      showMessage(message);
      await subscribe();
    }
  }
  subscribe();
}
~~~

Функция `subscribe()` делает запрос, затем ожидает ответ, обрабатывает его и снова вызывает сама себя.

Длинные опросы прекрасно работают, когда сообщения приходят редко.

Если сообщения приходят очень часто, то схема приёма-отправки сообщений, приведённая выше, становится похожей на «пилу».

Каждое сообщение – это отдельный запрос, с заголовками, авторизацией и так далее.

Поэтому в этом случае предпочтительнее использовать другой метод, такой как Websocket или Server Sent Events.
