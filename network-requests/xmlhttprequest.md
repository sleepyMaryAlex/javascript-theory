# ?XMLHttpRequest

_XMLHttpRequest_ – это встроенный в браузер объект, который даёт возможность делать HTTP-запросы к серверу без перезагрузки страницы.

На сегодняшний день не обязательно использовать `XMLHttpRequest`, так как существует другой, более современный метод `fetch`.

В современной веб-разработке `XMLHttpRequest` используется по трём причинам:

* По историческим причинам: существует много кода, использующего `XMLHttpRequest`, который нужно поддерживать.
* Необходимость поддерживать старые браузеры и нежелание использовать полифилы (например, чтобы уменьшить количество кода).
* Потребность в функциональности, которую `fetch` пока что не может предоставить, к примеру, отслеживание прогресса отправки на сервер.

### Основы

`XMLHttpRequest` имеет два режима работы: синхронный и асинхронный.

Сначала рассмотрим асинхронный, так как в большинстве случаев используется именно он.

Чтобы сделать запрос, нам нужно выполнить четыре шага:

1. Создать `XMLHttpRequest`.

Конструктор не имеет аргументов.

2. Инициализировать его.

Метод `open` обычно вызывается сразу после `new XMLHttpRequest`. В него передаются основные параметры запроса:

* `method` – HTTP-метод. Обычно это `GET` или `POST`.
* `URL` – `URL`, куда отправляется запрос: строка, может быть и объект `URL`.
* `async` – если указать `false`, тогда запрос будет выполнен синхронно, это мы рассмотрим чуть позже.
* `user`, `password` – логин и пароль для базовой HTTP-авторизации (если требуется).

Заметим, что вызов `open`, вопреки своему названию, не открывает соединение. Он лишь конфигурирует запрос, но непосредственно отсылается запрос только лишь после вызова `send`.

3. Послать запрос.

Метод `send` устанавливает соединение и отсылает запрос к серверу. Необязательный параметр `body` содержит тело запроса.

Некоторые типы запросов, такие как `GET`, не имеют тела. А некоторые, как, например, `POST`, используют `body`, чтобы отправлять данные на сервер. Мы позже увидим примеры.

4. Слушать события на `xhr`, чтобы получить ответ.

Три наиболее используемых события:

* `load` – происходит, когда получен какой-либо ответ, включая ответы с HTTP-ошибкой, например `404`.
* `error` – когда запрос не может быть выполнен, например, нет соединения или невалидный URL.
* `progress` – происходит периодически во время загрузки ответа, сообщает о прогрессе.

~~~
const xhr = new XMLHttpRequest();

xhr.open("GET", "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits");

xhr.send();

xhr.onload = function () {
  if (xhr.status != 200) {
    console.log(`Ошибка ${xhr.status}: ${xhr.statusText}`);
  } else {
    console.log(`Готово, получили ${xhr.response.length} байт`);
  }
};

xhr.onprogress = function (event) {
  if (event.lengthComputable) {
    console.log(`Получено ${event.loaded} из ${event.total} байт`);
  } else {
    console.log(`Получено ${event.loaded} байт`); // если в ответе нет заголовка Content-Length
  }
};

xhr.onerror = function () {
  console.log("Запрос не удался");
};

// Получено 139701 байт
// Готово, получили 139691 байт
~~~

После ответа сервера мы можем получить результат запроса в следующих свойствах `xhr`:

* `status` - код состояния HTTP (число): `200`, `404`, `403` и так далее, может быть `0` в случае, если ошибка не связана с HTTP.
* `statusText` - сообщение о состоянии ответа HTTP (строка): обычно `OK` для `200`, `Not Found` для `404`, `Forbidden` для `403`, и так далее.
* `response` (в старом коде может встречаться как `responseText`) - тело ответа сервера.

Мы можем также указать таймаут – промежуток времени, который мы готовы ждать ответ:

~~~
const xhr = new XMLHttpRequest();

xhr.open("GET", "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits");

xhr.timeout = 10;

xhr.send();

xhr.ontimeout = function () {
  console.log('Interrupted');
};

...
~~~

Если запрос не успевает выполниться в установленное время, то он прерывается, и происходит событие `timeout`.

### Состояния запроса

У `XMLHttpRequest` есть состояния, которые меняются по мере выполнения запроса. Текущее состояние можно посмотреть в свойстве `xhr.readyState`.

~~~
UNSENT = 0; // исходное состояние
OPENED = 1; // вызван метод open
HEADERS_RECEIVED = 2; // получены заголовки ответа
LOADING = 3; // ответ в процессе передачи (данные частично получены)
DONE = 4; // запрос завершён
~~~

Изменения в состоянии объекта запроса генерируют событие `readystatechange`:

~~~
const xhr = new XMLHttpRequest();

xhr.onreadystatechange = function() {
  console.log(xhr.readyState); // 1 2 3 4
};

xhr.open("GET", "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits");

xhr.send();
~~~

Вы можете наткнуться на обработчики события readystatechange в очень старом коде, так уж сложилось исторически, когда-то не было событий `load` и других. Сегодня из-за существования событий `load`/`error`/`progress` можно сказать, что событие `readystatechange` устарело.

### Отмена запроса

Если мы передумали делать запрос, можно отменить его вызовом `xhr.abort()`:

~~~
xhr.abort(); // завершить запрос
~~~

При этом генерируется событие `abort`, а `xhr.status` устанавливается в `0`.

### Синхронные запросы

Если в методе `open` третий параметр `async` установлен на `false`, запрос выполняется синхронно.

Вот переписанный пример с параметром `async`, равным `false`:

~~~
const xhr = new XMLHttpRequest();

xhr.open(
  "GET",
  "https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits",
  false
);

try {
  xhr.send();
  if (xhr.status != 200) {
    console.log(`Ошибка ${xhr.status}: ${xhr.statusText}`);
  } else {
    console.log(xhr.response);
  }
} catch (err) {
  // для отлова ошибок используем конструкцию try...catch вместо onerror
  console.log("Запрос не удался");
}
~~~

Выглядит, может быть, и неплохо, но синхронные запросы используются редко, так как они блокируют выполнение JavaScript до тех пор, пока загрузка не завершена. В некоторых браузерах нельзя прокручивать страницу, пока идёт синхронный запрос. Ну а если же синхронный запрос по какой-то причине выполняется слишком долго, браузер предложит закрыть «зависшую» страницу.

Многие продвинутые возможности `XMLHttpRequest`, такие как выполнение запроса на другой домен или установка таймаута, недоступны для синхронных запросов. Также, как вы могли заметить, ни о какой индикации прогресса речь тут не идёт.

Из-за всего этого синхронные запросы используют очень редко.

### HTTP-заголовки

`XMLHttpRequest` умеет как указывать свои заголовки в запросе, так и читать присланные в ответ.

Для работы с HTTP-заголовками есть 3 метода:

1. `setRequestHeader(name, value)` - устанавливает заголовок запроса с именем `name` и значением `value`.

~~~
xhr.setRequestHeader('Content-Type', 'application/json');
~~~

2. `getResponseHeader(name)` - возвращает значение заголовка ответа name (кроме Set-Cookie и Set-Cookie2).

~~~
xhr.onload = function () {
  if (xhr.status != 200) {
    console.log(`Ошибка ${xhr.status}: ${xhr.statusText}`);
  } else {
    console.log(`Готово, получили ${xhr.response.length} байт`);
    console.log(xhr.getResponseHeader('Content-Type')); // application/json; charset=utf-8
  }
};
~~~

3. `getAllResponseHeaders()` - возвращает все заголовки ответа, кроме Set-Cookie и Set-Cookie2.

Заголовки возвращаются в виде единой строки.

### POST, FormData

Чтобы сделать `POST`-запрос, мы можем использовать встроенный объект `FormData`.

~~~
<form name="person">
  <input name="name" value="Петя">
  <input name="surname" value="Васечкин">
</form>

<script>
  // заполним FormData данными из формы
  const formData = new FormData(document.forms.person);

  // добавим ещё одно поле
  formData.append("middle", "Иванович");

  // отправим данные
  const xhr = new XMLHttpRequest();
  xhr.open("POST", url);
  xhr.send(formData);

  xhr.onload = () => console.log(xhr.response);
</script>
~~~

Обычно форма отсылается в кодировке `multipart/form-data`.

Если нам больше нравится формат JSON, то используем `JSON.stringify` и отправляем данные как строку.

Важно не забыть поставить соответствующий заголовок `Content-Type: application/json`, многие серверные фреймворки автоматически декодируют JSON при его наличии:

~~~
const xhr = new XMLHttpRequest();

const json = JSON.stringify({
  name: "Вася",
  surname: "Петров"
});

xhr.open("POST", '/submit')
xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');

xhr.send(json);
~~~

### Прогресс отправки

Событие `progress` срабатывает только на стадии загрузки ответа с сервера.

Но если мы отправляем что-то большое, то нас гораздо больше интересует прогресс отправки данных на сервер. Но `xhr.onprogress` тут не поможет.

Существует другой объект, без методов, только для отслеживания событий отправки: `xhr.upload`.

Он генерирует события, похожие на события xhr, но только во время отправки данных на сервер:

* `loadstart` – начало загрузки данных.
* `progress` – генерируется периодически во время отправки на сервер.
* `abort` – загрузка прервана.
* `error` – ошибка, не связанная с HTTP.
* `load` – загрузка успешно завершена.
* `timeout` – вышло время, отведённое на загрузку (при установленном свойстве `timeout`).
* `loadend` – загрузка завершена, вне зависимости от того, как – успешно или нет.

Загрузка файла на сервер с индикацией прогресса:

~~~
<body>
  <input type="file" onchange="upload(this.files[0])">

  <script>
    function upload(file) {
      const xhr = new XMLHttpRequest();

      // отслеживаем процесс отправки
      xhr.upload.onprogress = function(event) {
        console.log(`Отправлено ${event.loaded} из ${event.total}`);
      };

      // Ждём завершения: неважно, успешного или нет
      xhr.onloadend = function() {
        if (xhr.status == 200) {
          console.log("Успех");
        } else {
          console.log("Ошибка " + this.status);
        }
      };

      xhr.open("POST", "/article/xmlhttprequest/post/upload");
      xhr.send(file);
    }
  </script>
</body>
~~~

### Запросы на другой источник

`XMLHttpRequest` может осуществлять запросы на другие сайты, используя ту же политику CORS, что и `fetch`.

Точно так же, как и при работе с `fetch`, по умолчанию на другой источник не отсылаются куки и заголовки HTTP-авторизации. Чтобы это изменить, установите `xhr.withCredentials` в `true`:

~~~
const xhr = new XMLHttpRequest();
xhr.withCredentials = true;

xhr.open('POST', 'http://anywhere.com/request');
...
~~~

### Возобновляемая загрузка файлов

Чтобы возобновить отправку, нам нужно знать, какая часть файла была успешно передана до того, как соединение прервалось.

Можно установить обработчик `xhr.upload.onprogress`, чтобы отслеживать процесс загрузки, но, к сожалению, это бесполезно, так как этот обработчик вызывается, только когда данные отправляются, но были ли они получены сервером? Браузер этого не знает.

В общем, событие `progress` подходит только для того, чтобы показывать красивый индикатор загрузки, не более.

Для возобновления же загрузки нужно точно знать, сколько байт было получено сервером. И только сам сервер может это сказать, поэтому будем делать для этого отдельный запрос.

Примерный код выглядит так:

_index.html_
~~~
<body>
  <script src="uploader.js"></script>
  <form name="upload" method="POST" enctype="multipart/form-data" action="/upload">
    <input type="file" name="myfile" title="file">
    <input type="submit" name="submit" value="Загрузить (возобновляется автоматически)">
  </form>

  <button onclick="uploader.stop()">Остановить загрузку</button>

  <script>
    function onProgress(loaded, total) {
      console.log("progress " + loaded + ' / ' + total);
    }

    let uploader;

    document.forms.upload.onsubmit = async function (e) {
      e.preventDefault();

      const file = this.elements.myfile.files[0];
      if (!file) return;

      uploader = new Uploader({ file, onProgress });

      try {
        const uploaded = await uploader.upload();
        if (uploaded) {
          console.log('success');
        } else {
          console.log('stopped');
        }
      } catch (err) {
        console.error(err);
      }
    };
  </script>
</body>
~~~

_uploader.js_
~~~
class Uploader {
  constructor({ file, onProgress }) {
    this.file = file;
    this.onProgress = onProgress;
    this.fileId = file.name + "-" + file.size + "-" + +file.lastModifiedDate;
  }

  async getUploadedBytes() {
    const response = await fetch("status", {
      headers: {
        "X-File-Id": this.fileId,
      },
    });

    if (response.status != 200) {
      throw new Error("Can't get uploaded bytes: " + response.statusText);
    }
    const text = await response.text();
    return +text;
  }

  async upload() {
    this.startByte = await this.getUploadedBytes();

    const xhr = (this.xhr = new XMLHttpRequest());
    xhr.open("POST", "upload", true);

    xhr.setRequestHeader("X-File-Id", this.fileId);

    xhr.setRequestHeader("X-Start-Byte", this.startByte);

    xhr.upload.onprogress = (e) => {
      this.onProgress(this.startByte + e.loaded, this.startByte + e.total);
    };

    console.log("send the file, starting from", this.startByte);
    xhr.send(this.file.slice(this.startByte));

    return await new Promise((resolve, reject) => {
      xhr.onload = xhr.onerror = () => {
        console.log(
          "upload end status:" + xhr.status + " text:" + xhr.statusText
        );

        if (xhr.status == 200) {
          resolve(true);
        } else {
          reject(new Error("Upload failed: " + xhr.statusText));
        }
      };
      xhr.onabort = () => resolve(false);
    });
  }

  stop() {
    if (this.xhr) {
      this.xhr.abort();
    }
  }
}
~~~

Алгоритм:

1. Во-первых, создадим уникальный идентификатор для файла, который собираемся загружать:

~~~
const fileId = file.name + '-' + file.size + '-' + +file.lastModifiedDate;
~~~

Это нужно, чтобы при возобновлении загрузки серверу было понятно, какой файл мы продолжаем загружать.

Если имя или размер или дата модификация файла изменятся, то у него уже будет другой `fileId`.

2. Далее, посылаем запрос к серверу с просьбой указать количество уже полученных байтов:

~~~
const response = await fetch('status', {
  headers: {
    'X-File-Id': fileId
  }
});

// сервер получил столько-то байтов
const startByte = +await response.text();
~~~

Предполагается, что сервер учитывает загружаемые файлы с помощью заголовка `X-File-Id`. Это на стороне сервера должно быть реализовано.

Если файл серверу неизвестен, то он должен ответить `0`.

3.Затем мы можем использовать метод `slice` объекта `Blob`, чтобы отправить данные, начиная со `startByte` байта:

~~~
xhr.open("POST", "upload");

// Идентификатор файла, чтобы сервер знал, что мы загружаем
xhr.setRequestHeader('X-File-Id', fileId);

// Номер байта, начиная с которого мы будем отправлять данные.
// Таким образом, сервер поймёт, с какого момента мы возобновляем загрузку
xhr.setRequestHeader('X-Start-Byte', startByte);

xhr.upload.onprogress = (e) => {
  console.log(`Uploaded ${startByte + e.loaded} of ${startByte + e.total}`);
};

// файл file может быть взят из input.files[0] или другого источника
xhr.send(file.slice(startByte));
~~~

Здесь мы посылаем серверу и идентификатор файла в заголовке `X-File-Id`, чтобы он знал, что мы загружаем, и номер стартового байта в заголовке `X-Start-Byte`, чтобы он понял, что мы продолжаем отправку, а не начинаем её с нуля.

Сервер должен проверить информацию на своей стороне, и если обнаружится, что такой файл уже когда-то загружался, и его текущий размер равен значению из заголовка `X-Start-Byte`, то вновь принимаемые данные добавлять в этот файл.
