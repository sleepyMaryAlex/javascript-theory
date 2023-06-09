# ?Server Sent Events

Спецификация Server-Sent Events описывает встроенный класс `EventSource`, который позволяет поддерживать соединение с сервером и получать от него события.

Как и в случае с WebSocket, соединение постоянно.

Но есть несколько важных различий:

| WebSocket |	EventSource |
|-----------|-------------|
| Двунаправленность: и сервер, и клиент могут обмениваться сообщениями | Однонаправленность: данные посылает только сервер |
| Бинарные и текстовые данные	| Только текст |
| Протокол WebSocket | Обычный HTTP |

EventSource не настолько мощный способ коммуникации с сервером, как WebSocket.

Зачем нам его использовать?

Основная причина: он проще. Многим приложениям не требуется вся мощь WebSocket.

Если нам нужно получать поток данных с сервера: неважно, сообщения в чате или же цены для магазина – с этим легко справится EventSource. К тому же, он поддерживает автоматическое переподключение при потере соединения, которое, используя WebSocket, нам бы пришлось реализовывать самим. Кроме того, используется старый добрый HTTP, а не новый протокол.

### Получение сообщений

Чтобы начать получать данные, нам нужно просто создать `new EventSource(url)`.

Браузер установит соединение с `url` и будет поддерживать его открытым, ожидая события.

Сервер должен ответить со статусом `200` и заголовком `Content-Type: text/event-stream`, затем он должен поддерживать соединение открытым и отправлять сообщения в особом формате:

~~~
data: Сообщение 1

data: Сообщение 2

data: Сообщение 3
data: в две строки
~~~

Текст сообщения указывается после `data:`, пробел после двоеточия необязателен.

Сообщения разделяются двойным переносом строки `\n\n`.

Чтобы разделить сообщение на несколько строк, мы можем отправить несколько `data:` подряд (третье сообщение).

На практике сложные сообщения обычно отправляются в формате JSON, в котором перевод строки кодируется как `\n`, так что в разделении сообщения на несколько строк обычно нет нужды.

Например:

~~~
data: {"user":"Джон","message":"Первая строка\n Вторая строка"}
~~~

…Так что можно считать, что в каждом `data:` содержится ровно одно сообщение.

Для каждого сообщения генерируется событие `message`:

~~~
const eventSource = new EventSource("/events/subscribe");

eventSource.onmessage = function(event) {
  console.log("Новое сообщение", event.data);
};

// или eventSource.addEventListener('message', ...)
~~~

### Кросс-доменные запросы

`EventSource`, как и `fetch`, поддерживает кросс-доменные запросы. Мы можем использовать любой URL.

Сервер получит заголовок `Origin` и должен будет ответить с заголовком `Access-Control-Allow-Origin`.

Чтобы послать авторизационные данные, следует установить дополнительную опцию `withCredentials`:

~~~
const source = new EventSource("https://another-site.com/events", {
  withCredentials: true
});
~~~

### Переподключение

После создания `new EventSource` подключается к серверу и, если соединение обрывается, – переподключается.

Это очень удобно, так как нам не приходится беспокоиться об этом.

По умолчанию между попытками возобновить соединение будет небольшая пауза в несколько секунд.

Сервер может выставить рекомендуемую задержку, указав в ответе `retry`: (в миллисекундах):

~~~
retry: 15000
data: Привет, я выставил задержку переподключения в 15 секунд
~~~

Поле `retry`: может посылаться как вместе с данными, так и отдельным сообщением.

Браузеру следует ждать именно столько миллисекунд перед новой попыткой подключения. Или дольше, например, если браузер знает (от операционной системы) что соединения с сетью нет, то он может осуществить переподключение только когда оно появится.

Если сервер хочет остановить попытки переподключения, он должен ответить со статусом `204`.

Если браузер хочет прекратить соединение, он может вызвать `eventSource.close()`:

~~~
const eventSource = new EventSource(...);

eventSource.close();
~~~

Также переподключение не произойдёт, если в ответе указан неверный `Content-Type` или его статус отличается от `301`, `307`, `200` и `204`. Браузер создаст событие `error` и не будет восстанавливать соединение.

> После того как соединение окончательно закрыто, «переоткрыть» его уже нельзя. Если необходимо снова подключиться, просто создайте новый `EventSource`.

### Идентификатор сообщения

Когда соединение прерывается из-за проблем с сетью, ни сервер, ни клиент не могут быть уверены в том, какие сообщения были доставлены, а какие – нет.

Чтобы правильно возобновить подключение, каждое сообщение должно иметь поле `id`:

~~~
data: Сообщение 1
id: 1

data: Сообщение 2
id: 2

data: Сообщение 3
data: в две строки
id: 3
~~~

Получая сообщение с указанным `id:`, браузер:

* Установит его значение свойству `eventSource.lastEventId`.
* При переподключении отправит заголовок `Last-Event-ID` с этим `id`, чтобы сервер мог переслать последующие сообщения.

> Обратите внимание: `id` указывается сервером после данных `data` сообщения, чтобы обновление `lastEventId` произошло после того, как сообщение будет получено.

### Статус подключения: `readyState`

У объекта `EventSource` есть свойство `readyState`, имеющее одно из трёх значений:

~~~
EventSource.CONNECTING = 0; // подключение или переподключение
EventSource.OPEN = 1;       // подключено
EventSource.CLOSED = 2;     // подключение закрыто
~~~

При создании объекта и разрыве соединения оно автоматически устанавливается в значение `EventSource.CONNECTING` (равно `0`).

Мы можем обратиться к этому свойству, чтобы узнать текущее состояние `EventSource`.

### Типы событий

По умолчанию объект `EventSource` генерирует 3 события:

* `message` – получено сообщение, доступно как `event.data`.
* `open` – соединение открыто.
* `error` – не удалось установить соединение, например, сервер вернул статус `500`.

Сервер может указать другой тип события с помощью `event: ...` в начале сообщения.

Например:

~~~
event: join
data: Боб

data: Привет

event: leave
data: Боб
~~~

Чтобы начать слушать пользовательские события, нужно использовать `addEventListener`, а не `onmessage`:

~~~
eventSource.addEventListener('join', event => {
  console.log(`${event.data} зашёл`);
});
~~~

### Полный пример

В этом примере сервер посылает сообщения 1, 2, 3, затем пока-пока и разрывает соединение.

После этого браузер автоматически переподключается.

_index.html_
~~~
<body>
  <script>
    let eventSource;

    function start() {
      if (!window.EventSource) {
        // Internet Explorer или устаревшие браузеры
        console.log("Ваш браузер не поддерживает EventSource.");
        return;
      }

      eventSource = new EventSource('digits');

      eventSource.onopen = function (e) {
        log("Событие: open");
      };

      eventSource.onerror = function (e) {
        log("Событие: error");
        if (this.readyState == EventSource.CONNECTING) {
          log(`Переподключение (readyState=${this.readyState})...`);
        } else {
          log("Произошла ошибка.");
        }
      };

      eventSource.addEventListener('bye', function (e) {
        log("Событие: bye, данные: " + e.data);
      });

      eventSource.onmessage = function (e) {
        log("Событие: message, данные: " + e.data);
      };
    }

    function stop() {
      eventSource.close();
      log("Соединение закрыто");
    }

    function log(msg) {
      logElem.innerHTML += msg + "<br>";
      document.documentElement.scrollTop = 99999999;
    }
  </script>

  <button onclick="start()">Старт</button> Нажмите кнопку "Старт" для начала
  <div id="logElem"></div>

  <button onclick="stop()">Стоп</button> Чтобы закончить, нажмите "Стоп".
</body>
~~~

_server.js_
~~~
const http = require("http");
const static = require("node-static");
const fileServer = new static.Server(".");

function onDigits(req, res) {
  res.writeHead(200, {
    "Content-Type": "text/event-stream; charset=utf-8",
    "Cache-Control": "no-cache",
  });

  let i = 0;

  const timer = setInterval(write, 1000);
  write();

  function write() {
    i++;

    if (i == 4) {
      res.write("event: bye\ndata: пока-пока\n\n");
      clearInterval(timer);
      res.end();
      return;
    }

    res.write("data: " + i + "\n\n");
  }
}

function accept(req, res) {
  if (req.url == "/digits") {
    onDigits(req, res);
    return;
  }
  fileServer.serve(req, res);
}

if (!module.parent) {
  http.createServer(accept).listen(8080);
} else {
  exports.accept = accept;
}
~~~

Чтобы посмотреть последний пример локально, нужно:

1. Установить Node.js.
2. Инициализировать проект `npm init`, можно нажимать Enter до тех пор, пока процесс не закончится.
3. Установить зависимости `npm i debug node-static`.
4. Запустить сервер `node server`.
5. В браузере перейти по адресу `http://127.0.0.1:8080` и смотреть пример
