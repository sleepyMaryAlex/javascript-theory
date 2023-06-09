# ?HTTP methods, headers, request, response

### HTTP methods

HTTP определяет множество методов запроса, которые указывают, какое желаемое действие выполнится для данного ресурса. Несмотря на то, что их названия могут быть существительными, эти методы запроса иногда называются HTTP глаголами.

Каждый реализует свою семантику, но каждая группа команд разделяет общие свойства: так, методы могут быть безопасными, идемпотентными или кешируемыми.

* `GET` - запрашивает представление ресурса. Запросы с использованием этого метода могут только извлекать данные.
* `HEAD` - запрашивает ресурс так же, как и метод `GET`, но без тела ответа.
* `POST` - используется для отправки сущностей к определённому ресурсу. Часто вызывает изменение состояния или какие-то побочные эффекты на сервере.
* `PUT` - заменяет все текущие представления ресурса данными запроса.
* `DELETE` - удаляет указанный ресурс.
* `CONNECT` - устанавливает "туннель" к серверу, определённому по ресурсу.
* `OPTIONS` - используется для описания параметров соединения с ресурсом.
* `TRACE` - выполняет вызов возвращаемого тестового сообщения с ресурса.
* `PATCH` - используется для частичного изменения ресурса.

### HTTP Headers

_Заголовки_ – это специальные параметры, которые несут определенную служебную информацию о соединении по HTTP. Некоторые заголовки имеют лишь информационный характер для пользователя или для компьютера, другие передают определенные команды, исходя из которых, сервер или клиент будет выполнять какие-то действия.

В зависимости от того, где эти заголовки могут находиться, они разделяются на:

* __General Headers__ (Основные заголовки) — должны быть и в запросах и в ответах клиента и сервера.
* __Request Headers__ (Заголовки запроса) — используются только в запросах клиента.
* __Response Headers__ (Заголовки ответа) — используются только в ответах сервера.
* __Entity Headers__ (Заголовки сущности) — сопровождают каждую сущность сообщения.

Рассмотрим некоторые из наиболее распространенных HTTP headers.

#### Заголовки HTTP в запросах HTTP

* `Host`. HTTP-запрос отправляется на определенные IP-адреса. Но так как большинство серверов способны размещать несколько сайтов под одним IP, они должны знать, какое доменное имя ищет браузер.

`Host` указывает хост, на который отправляется запрос.

~~~
Host: net.tutsplus.com
~~~

Это в основном имя `host`, включая домен и поддомен.

* `User-Agent`. Этот заголовок может содержать несколько частей информации, таких как:

1. Имя и версия браузера.
2. Название и версия операционной системы.
3. Язык по умолчанию.

~~~
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5 (.NET CLR 3.5.30729)
~~~

* `Accept-Language`. Этот заголовок отображает настройки языка по умолчанию.

Он может содержать несколько языков, разделённых запятыми. Первый - это предпочтительный язык, и каждый из перечисленных языков может иметь значение «q», которое представляет собой оценку предпочтения пользователя для языка (min. 0 max. 1).

~~~
Accept-Language: en-us,en;q=0.5
~~~

* `Accept-Encoding`. Большинство современных браузеров поддерживают `gzip` и отправляют это в `header`. Затем веб-сервер может отправить выходной HTML-код в сжатом формате. Это позволяет уменьшить размер до 80% для экономии пропускной способности и времени.

~~~
Accept-Encoding: gzip,deflate
~~~

* `If-Modified-Since`. Если веб-документ уже сохранен в кеше в браузере и вы посещаете его снова, ваш браузер может проверить, был ли документ обновлён, отправив следующее:

~~~
If-Modified-Since: Sat, 28 Nov 2009 06:38:19 GMT
~~~

* `Cookie`. Как следует из названия, это отправляет файлы `cookie`, хранящиеся в вашем браузере для этого домена.

~~~
Cookie: PHPSESSID=r2t5uvjq435r4q7ib3vtdjq120; foo=bar
~~~

* `Referer`. Как следует из названия, этот `HTTP header` содержит ссылочный `url`.

Например, если я зашел на домашнюю страницу Nettuts + и нажал ссылку на статью, этот `header` будет отправлен в мой браузер:

~~~
Referer: https://net.tutsplus.com/
~~~

Возможно, вы заметили, что слово «referrer» написано с ошибкой, как «referer». К сожалению, он превратился в официальную спецификацию HTTP подобным образом и застрял.

* `Authorization`. Когда веб-страница запрашивает авторизацию, браузер открывает окно входа в систему. Когда вы вводите имя пользователя и пароль в этом окне, браузер отправляет другой HTTP-запрос, но на этот раз он содержит этот `header`:

~~~
Authorization: Basic bXl1c2VyOm15cGFzcw==
~~~

Данные внутри `header` имеют кодировку `base64`. Например, `base64_decode ('bXl1c2VyOm15cGFzcw ==')` возвратит `'myuser: mypass'`.

#### Заголовки HTTP в ответах HTTP

* `Cache-Control`. Поле заголовка `Cache-Control` используется для указания директив, которые должны выполняться всеми механизмами кэширования по цепочке запросов/ответов.

~~~
Cache-Control: max-age=3600, public
~~~

`"public"` означает, что ответ может быть кэширован кем угодно. `"max-age"` указывает, сколько секунд действителен кеш. Разрешение кэширования вашего сайта может снизить нагрузку на сервер и пропускную способность, а также увеличить время загрузки в браузере.

Кэширование также может быть предотвращено с помощью директивы `"no-cache"`.

~~~
Cache-Control: no-cache
~~~

* `Content-Type`. Этот header указывает `"mime-type"` документа. Затем браузер определяет, как интерпретировать содержимое на основании этого. Например, страница html может возвращать это:

~~~
Content-Type: text/html; charset=UTF-8
~~~

`"text"` - это тип, а `"html"` - подтип документа. Заголовок также может содержать больше информации, такой как `charset`.

* `Content-Disposition`. Этот `header` указывает браузеру открыть окно загрузки файла, вместо того, чтобы пытаться проанализировать содержимое. Пример:

~~~
Content-Disposition: attachment; filename="download.zip"
~~~

Обратите внимание, что соответствующий заголовок Content-Type также должен быть отправлен вместе с этим:

~~~
Content-Type: application/zip
Content-Disposition: attachment; filename="download.zip"
~~~

* `Content-Length`. Когда контент будет передаваться браузеру, сервер может указать его размер (в байтах), используя этот `header`.

~~~
Content-Length: 89123
~~~

* `Etag`. Это еще один `header`, который используется для кеширования. Это выглядит так:

~~~
Etag: "pub1259380237;gz"
~~~

Веб-сервер может отправлять этот `header` с каждым документом, который он обслуживает. Значение может быть основано на последней изменённой дате, размере файла или даже контрольной сумме файла.

Браузер затем сохраняет это значение, так как он кэширует документ. В следующий раз, когда браузер запрашивает тот же файл, он отправляет это в HTTP-запросе:

~~~
If-None-Match: "pub1259380237;gz"
~~~

Если значение `Etag` документа совпадает с этим, сервер будет отправлять код `304` вместо `200`, и никакого содержимого. Браузер будет загружать содержимое из своего кеша.

* `Last-Modified`. Как следует из названия, этот `header` указывает дату последнего изменения документа в формате GMT:

~~~
Last-Modified: Mon, 12 Jun 2023 07:26:57 GMT
~~~

* `Location`. Этот заголовок используется для перенаправления. Если код ответа `301` или `302`, сервер также должен отправить этот `header`.

~~~
Location: https://net.tutsplus.com/
~~~

* `Set-Cookie`. Когда веб-сайт хочет установить или обновить файл `cookie` в вашем браузере, он будет использовать этот `header`.

~~~
Set-Cookie: skin=noskin; path=/; domain=.amazon.com; expires=Sun, 29-Nov-2009 21:42:28 GMT
Set-Cookie: session-id=120-7333518-8165026; path=/; domain=.amazon.com; expires=Sat Feb 27 08:00:00 2010 GMT
~~~

Каждый файл `cookie` отправляется как отдельный `header`. Обратите внимание, что файлы `cookie`, установленные с помощью JavaScript, не проходят через `HTTP headers`.

* `WWW-Authenticate`. Сайт может отправить этот `header` для аутентификации пользователя через HTTP. Когда браузер увидит этот `header`, он откроет диалоговое окно входа в систему.

~~~
WWW-Authenticate: Basic realm="Restricted Area"
~~~

* `Content-Encoding`. Этот `header` обычно устанавливается, когда возвращаемое содержимое сжимается.

~~~
Content-Encoding: gzip
~~~

### HTTP request

HTTP-запрос содержит следующие элементы:

1. Строка запроса
2. Заголовки HTTP-запросов
3. Текст сообщения запроса

#### Строка запроса

Строка запроса — это первая строка сообщения запроса, содержащая как минимум три элемента:

1. Метод HTTP. Методы — это команды из одного слова, которые сообщают серверу, что делать с ресурсами.
2. Компонент пути URL для запроса. Путь идентифицирует ресурс на сервере.
3. Номер версии HTTP. Отображение спецификации HTTP.

~~~
GET /echo HTTP/1.1
~~~

#### Заголовки HTTP-запросов

Заголовки HTTP указываются для сообщения, чтобы предоставить получателю информацию о сообщении, отправителе и способе, которым отправитель хочет общаться с получателем.

Каждый заголовок HTTP состоит из пары «ключ: значение». Заголовки HTTP для запроса клиента содержат информацию, которую сервер может использовать, чтобы решить, как ответить на запрос.

~~~
Content-Type: application/json
~~~

#### Текст сообщения запроса

Тело сообщения запроса содержит тело объекта, который может быть в исходном состоянии или закодирован. Тела сообщений подходят для одних методов запроса и не подходят для других.

Например, запрос с методом HTTP POST, который отправляет входные данные на сервер, имеет тело сообщения, содержащее данные. Запрос с методом HTTP GET, который просит сервер отправить ресурс, не имеет тела сообщения.

### HTTP response

HTTP-ответ отправляется сервером клиенту. Цель ответа — предоставить клиенту запрошенный им ресурс или проинформировать клиента о том, что запрошенное действие было выполнено; или же сообщить клиенту, что при обработке его запроса произошла ошибка.

Ответ HTTP содержит:

1. Строка состояния.
2. Заголовки ответа HTTP
3. Тело ответного сообщения

Как и в сообщении запроса, за каждым заголовком HTTP следует CRLF. После последнего из HTTP-заголовков используется дополнительный CRLF (чтобы дать пустую строку), а затем начинается тело сообщения.

> CR и LF это управляющие символы или байт-код которые можно использовать для обозначения разрыва строки в текстовых файлах.

#### Строка состояния

Строка состояния — это первая строка в сообщении запроса, и она содержит три элемента:

1. Номер версии HTTP. Отображает спецификацию HTTP.
2. Код состояния, указывающий на результат запроса.
3. Фраза причины или текст состояния резюмирует значение кода состояния в удобочитаемом тексте.

~~~
HTTP/1.1 200 OK
~~~

#### HTTP-заголовки

Заголовки HTTP для ответа сервера содержат информацию, которую клиент может использовать, чтобы узнать больше об ответе и сервере, который его отправил.

Эта информация может помочь клиенту отобразить ответ пользователю и сохранить (или кэшировать) ответ для использования в будущем. Если запрос терпит неудачу, заголовки могут сообщить клиенту, что ему нужно сделать, чтобы добиться успеха.

~~~
Content-Type: text/html; charset=utf-8
~~~

Пустая строка (CRLF) помещается в сообщение запроса после ряда заголовков HTTP, чтобы отделить заголовки от тела сообщения.

#### Тело сообщения

Тела сообщений используются для большинства ответов. Исключения составляют случаи, когда сервер отвечает на запрос клиента с помощью метода HTTP HEAD (который запрашивает заголовки, но не тело ответа) и когда сервер использует определенные коды состояния.

### HTTP Request и Response пример

Пример HTTP-запроса

~~~
GET /echo/get/json HTTP/1.1
Host: reqbin.com
Accept: application/json
~~~

Пример HTTP-ответа

~~~
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 19

{"success":"true"}
~~~
