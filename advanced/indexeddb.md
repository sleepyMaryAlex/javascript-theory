# ?IndexedDB

_IndexedDB_ – это встроенная база данных, более мощная, чем `localStorage`.

* Хранит практически любые значения по ключам, несколько типов ключей
* Поддерживает транзакции для надёжности.
* Поддерживает запросы в диапазоне ключей и индексы.
* Позволяет хранить больше данных, чем `localStorage`.

Для традиционных клиент-серверных приложений эта мощность обычно чрезмерна. IndexedDB предназначена для оффлайн приложений.

Интерфейс для IndexedDB основан на событиях.

Технически данные обычно хранятся в домашнем каталоге посетителя вместе с настройками браузера, расширениями и т.д.

У разных браузеров и пользователей на уровне ОС есть своё собственное независимое хранилище.

### Открыть базу данных

Для начала работы с IndexedDB нужно открыть базу данных.

~~~
const openRequest = indexedDB.open("store", 1);

openRequest.onsuccess = function () {
  console.log("success");
};

openRequest.onerror = function () {
  console.log("error");
};

openRequest.onupgradeneeded = function () {
  console.log("upgradeneeded");
};
~~~

* Первый параметр – название базы данных, строка.
* Второй параметр – версия базы данных, положительное целое число, по умолчанию 1 (объясняется ниже).

У нас может быть множество баз данных с различными именами, но все они существуют в контексте текущего источника (домен/протокол/порт). Разные сайты не могут получить доступ к базам данных друг друга.

После этого вызова назначен обработчик событий для объекта `openRequest`:

* `success`: база данных готова к работе, готов «объект базы данных» `openRequest.result`, его следует использовать для дальнейших вызовов.
* `error`: не удалось открыть базу данных.
* `upgradeneeded`: база открыта, но её схема устарела (см. ниже).

__IndexedDB имеет встроенный механизм «версионирования схемы», который отсутствует в серверных базах данных.__

В отличие от серверных баз данных, IndexedDB работает на стороне клиента, в браузере, и у нас нет прямого доступа к данным. Но когда мы публикуем новую версию нашего приложения, возможно, нам понадобится обновить базу данных.

Если локальная версия базы данных меньше, чем версия, определённая в `open`, то сработает специальное событие `upgradeneeded`, и мы сможем сравнить версии и обновить структуры данных по мере необходимости.

Это событие также сработает, если базы данных ещё не существует, так что в этом обработчике мы можем выполнить инициализацию.

Допустим, мы опубликовали первую версию нашего приложения.

Затем мы можем открыть базу данных с версией 1 и выполнить инициализацию в обработчике `upgradeneeded` вот так:

~~~
let openRequest = indexedDB.open("store", 1);

openRequest.onupgradeneeded = function() {
  // срабатывает, если на клиенте нет базы данных
  // ...выполнить инициализацию...
};

openRequest.onerror = function() {
  console.error("Error", openRequest.error);
};

openRequest.onsuccess = function() {
  let db = openRequest.result;
  // продолжить работу с базой данных, используя объект db
};
~~~

Затем, позже, мы публикуем 2-ю версию.

Мы можем открыть его с версией 2 и выполнить обновление следующим образом:

~~~
let openRequest = indexedDB.open("store", 2);

openRequest.onupgradeneeded = function(event) {
  // версия существующей базы данных меньше 2 (или база данных не существует)
  let db = openRequest.result;
  switch(event.oldVersion) { // существующая (старая) версия базы данных
    case 0:
      // версия 0 означает, что на клиенте нет базы данных
      // выполнить инициализацию
    case 1:
      // на клиенте версия базы данных 1
      // обновить
  }
};
~~~

Таким образом, в `openRequest.onupgradeneeded` мы обновляем базу данных. Скоро подробно увидим, как это делается. А после того, как этот обработчик завершится без ошибок, сработает `openRequest.onsuccess`.

После `openRequest.onsuccess` у нас есть объект базы данных в `openRequest.result`, который мы будем использовать для дальнейших операций.

Удалить базу данных:

~~~
const deleteRequest = indexedDB.deleteDatabase(name);
// deleteRequest.onsuccess/onerror отслеживает результат
~~~

А что, если открыть предыдущую версию?
Что если мы попробуем открыть базу с более низкой версией, чем текущая? Например, на клиенте база версии 3, а мы вызываем `open(...2)`.

Возникнет ошибка, сработает `openRequest.onerror`.

Такое может произойти, если посетитель загрузил устаревший код, например, из кеша прокси. Нам следует проверить `db.version` и предложить ему перезагрузить страницу. А также проверить наши кеширующие заголовки, убедиться, что посетитель никогда не получит устаревший код.

##### Проблема параллельного обновления

Раз уж мы говорим про версионирование, рассмотрим связанную с этим небольшую проблему.

Допустим:

1. Посетитель открыл наш сайт во вкладке браузера, с базой версии 1.
2. Затем мы выпустили обновление, так что наш код обновился.
3. И затем тот же посетитель открыл наш сайт в другой вкладке.

Так что есть две вкладки, на которых открыт наш сайт, но в одной открыто соединение с базой версии 1, а другая пытается обновить версию базы в обработчике `upgradeneeded`.

Проблема заключается в том, что база данных всего одна на две вкладки, так как это один и тот же сайт, один источник. И она не может быть одновременно версии 1 и 2. Чтобы обновить на версию 2, все соединения к версии 1 должны быть закрыты.

Чтобы это можно было организовать, при попытке обновления на объекте базы возникает событие `versionchange`. Нам нужно слушать его и закрыть соединение к базе (а также, возможно, предложить пользователю перезагрузить страницу, чтобы получить обновлённый код).

Если мы его не закроем, то второе, новое соединение будет заблокировано с событием `blocked` вместо `success`.

Код, который это делает:

~~~
const openRequest = indexedDB.open("store", 2);

openRequest.onupgradeneeded = ...;
openRequest.onerror = ...;

openRequest.onsuccess = function() {
  const db = openRequest.result;

  db.onversionchange = function() {
    db.close();
    alert("База данных устарела, пожалуйста, перезагрузите страницу.")
  };

  // ...база данных готова, используйте ее...
};

openRequest.onblocked = function() {
  // это событие не должно срабатывать, если мы правильно обрабатываем onversionchange

  // это означает, что есть ещё одно открытое соединение с той же базой данных
  // и он не был закрыт после того, как для него сработал db.onversionchange
};
~~~

…Другими словами, здесь мы делаем две вещи:

1. Обработчик `db.onversionchange` сообщает нам о попытке параллельного обновления, если текущая версия базы данных устарела.
2. Обработчик `OpenRequest.onblocked` сообщает нам об обратной ситуации: в другом месте есть соединение с устаревшей версией, и оно не закрывается, поэтому новое соединение установить невозможно.

Мы можем более изящно обращаться с вещами в `db.onversionchange`, например предлагать посетителю сохранить данные до закрытия соединения и так далее.

Или альтернативным подходом было бы не закрывать базу данных в `db.onversionchange`, а вместо этого использовать обработчик `onblocked` (на новой вкладке), чтобы предупредить посетителя, что более новая версия не может быть загружена, пока они не закроют другие вкладки.

Такой конфликт при обновлении происходит редко, но мы должны как-то его обрабатывать, хотя бы поставить обработчик `onblocked`, чтобы наш скрипт не «умирал» молча, удивляя посетителя.

### Хранилище объектов

Чтобы сохранить что-то в IndexedDB, нам нужно хранилище объектов.

_Хранилище объектов_ – это основная концепция IndexedDB. В других базах данных это «таблицы» или «коллекции». Здесь хранятся данные. В базе данных может быть множество хранилищ: одно для пользователей, другое для товаров и так далее.

Несмотря на то, что название – «хранилище объектов», примитивы тоже могут там храниться.

Мы можем хранить почти любое значение, в том числе сложные объекты.

IndexedDB использует стандартный алгоритм сериализации для клонирования и хранения объекта. Это как `JSON.stringify`, но более мощный, способный хранить гораздо больше типов данных.

Пример объекта, который нельзя сохранить: объект с циклическими ссылками. Такие объекты не сериализуемы. `JSON.stringify` также выдаст ошибку при сериализации.

__Каждому значению в хранилище должен соответствовать уникальный ключ.__

Ключ должен быть одним из следующих типов: `number`, `date`, `string` или `array`. Это уникальный идентификатор: по ключу мы можем искать/удалять/обновлять значения.

Можно указать ключ при добавлении значения в хранилище, аналогично localStorage. Но когда мы храним объекты, IndexedDB позволяет установить свойство объекта в качестве ключа, что гораздо удобнее. Или мы можем автоматически сгенерировать ключи.

Но для начала нужно создать хранилище.

Синтаксис для создания хранилища объектов:

~~~
db.createObjectStore(name[, keyOptions]);
~~~

Обратите внимание, что операция является синхронной, использование `await` не требуется.

* `name` – это название хранилища, например "books" для книг,
* `keyOptions` – это необязательный объект с одним или двумя свойствами:
  * `keyPath` – путь к свойству объекта, которое IndexedDB будет использовать в качестве ключа, например id.
  * `autoIncrement` – если `true`, то ключ будет формироваться автоматически для новых объектов, как постоянно увеличивающееся число.

Если при создании хранилища не указать `keyOptions`, то нам потребуется явно указать ключ позже, при сохранении объекта.

Например, это хранилище объектов использует свойство id как ключ:

~~~
db.createObjectStore('books', { keyPath: 'id' });
~~~

__Хранилище объектов можно создавать/изменять только при обновлении версии базы данных в обработчике `upgradeneeded`.__

Это техническое ограничение. Вне обработчика мы сможем добавлять/удалять/обновлять данные, но хранилища объектов могут быть созданы/удалены/изменены только во время обновления версии базы данных.

Для обновления версии базы есть два основных подхода:

1. Мы можем реализовать функции обновления по версиям: с 1 на 2, с 2 на 3 и т.д. Потом в `upgradeneeded` сравнить версии (например, была 2, сейчас 4) и запустить операции обновления для каждой промежуточной версии (2 на 3, затем 3 на 4).
2. Или мы можем взять список существующих хранилищ объектов, используя `db.objectStoreNames`. Этот объект является `DOMStringList`, в нём есть метод `contains(name)`, используя который можно проверить существование хранилища. Посмотреть, какие хранилища есть и создать те, которых нет.

Для простых баз данных второй подход может быть проще и предпочтительнее.

Вот демонстрация второго способа:

~~~
const openRequest = indexedDB.open("db", 2);

// создаём хранилище объектов для books, если ешё не существует
openRequest.onupgradeneeded = function() {
  const db = openRequest.result;
  if (!db.objectStoreNames.contains('books')) { // если хранилище "books" не существует
    db.createObjectStore('books', { keyPath: 'id' }); // создаём хранилище
  }
};
~~~

Чтобы удалить хранилище объектов:

~~~
db.deleteObjectStore('books');
~~~

### Транзакции

Термин «транзакция» является общеизвестным, транзакции используются во многих видах баз данных.

_Транзакция_ – это группа операций, которые должны быть или все выполнены, или все не выполнены (всё или ничего).

Например, когда пользователь что-то покупает, нам нужно:

1. Вычесть деньги с его счёта.
2. Отправить ему покупку.

Будет очень плохо, если мы успеем завершить первую операцию, а затем что-то пойдёт не так, например отключат электричество, и мы не сможем завершить вторую операцию. Обе операции должны быть успешно завершены (покупка сделана, отлично!) или необходимо отменить обе операции (в этом случае пользователь сохранит свои деньги и может попытаться купить ещё раз).

Транзакции гарантируют это.

Все операции с данными в IndexedDB могут быть сделаны только внутри транзакций.

Для начала транзакции:

~~~
db.transaction(store[, type]);
~~~

* `store` – это название хранилища, к которому транзакция получит доступ, например, "books". Может быть массивом названий, если нам нужно предоставить доступ к нескольким хранилищам.
* `type` – тип транзакции, один из:
  * `readonly` – только чтение, по умолчанию.
  * `readwrite` – только чтение и запись данных, создание/удаление самих хранилищ объектов недоступно.

Есть ещё один тип транзакций: `versionchange`. Такие транзакции могут делать любые операции, но мы не можем создать их вручную. IndexedDB автоматически создаёт транзакцию типа `versionchange`, когда открывает базу данных, для обработчика `upgradeneeded`. Вот почему это единственное место, где мы можем обновлять структуру базы данных, создавать/удалять хранилища объектов.

__Почему существует несколько типов транзакций?__

Производительность является причиной, почему транзакции необходимо помечать как `readonly` или `readwrite`.

Несколько `readonly` транзакций могут одновременно работать с одним и тем же хранилищем объектов, а `readwrite` транзакций – не могут. Транзакции типа `readwrite` «блокируют» хранилище для записи. Следующая такая транзакция должна дождаться выполнения предыдущей, перед тем как получит доступ к тому же самому хранилищу.

После того, как транзакция будет создана, мы можем добавить элемент в хранилище, вот так:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  const index = store.createIndex("NameIndex", ["name.last", "name.first"]);
  console.log(index);
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const index = store.index("NameIndex");

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.add({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });

  const getJohn = store.get(12345);
  const getBob = index.get(["Smith", "Bob"]);

  getJohn.onsuccess = function () {
    console.log(getJohn.result.name.first); // => "John"
  };

  getBob.onsuccess = function () {
    console.log(getBob.result.name.first); // => "Bob"
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

Мы сделали четыре шага:

1. Создать транзакцию и указать все хранилища, к которым необходим доступ.
2. Получить хранилище объектов, используя `transaction.objectStore(name)`.
3. Выполнить запрос на добавление элемента в хранилище объектов `store.put`/`store.add`.
4. …Обработать результат запроса, затем мы можем выполнить другие запросы и так далее.

Хранилища объектов поддерживают два метода для добавления значений:

* `put(value, [key])` Добавляет значение `value` в хранилище. Ключ `key` необходимо указать, если при создании хранилища объектов не было указано свойство `keyPath` или `autoIncrement`. Если уже есть значение с таким же ключом, то оно будет заменено.

* `add(value, [key])` То же, что `put`, но если уже существует значение с таким ключом, то запрос не выполнится, будет сгенерирована ошибка с названием `ConstraintError`.

### Автоматическая фиксация транзакций

В примере выше мы запустили транзакцию и выполнили запрос `add`. Но, как говорилось ранее, транзакция может включать в себя несколько запросов, которые все вместе должны либо успешно завершиться, либо нет. Как нам закончить транзакцию, обозначить, что больше запросов в ней не будет?

Короткий ответ: этого не требуется.

В следующей 3.0 версии спецификации, вероятно, будет возможность вручную завершить транзакцию, но сейчас, в версии 2.0, такой возможности нет.

__Когда все запросы завершены и очередь микрозадач пуста, тогда транзакция завершится автоматически.__

Как правило, это означает, что транзакция автоматически завершается, когда выполнились все её запросы и завершился текущий код.

Таким образом, в приведённом выше примере не требуется никакого специального вызова, чтобы завершить транзакцию.

Такое автозавершение транзакций имеет важный побочный эффект. Мы не можем вставить асинхронную операцию, такую как `fetch` или `setTimeout` в середину транзакции. IndexedDB никак не заставит транзакцию «висеть» и ждать их выполнения.

Всё потому, что `fetch` является асинхронной операцией, макрозадачей. Транзакции завершаются раньше, чем браузер приступает к выполнению макрозадач.

Авторы спецификации IndexedDB из соображений производительности считают, что транзакции должны завершаться быстро.

В частности, `readwrite` транзакции «блокируют» хранилища от записи. Таким образом, если одна часть приложения инициирует `readwrite` транзакцию в хранилище объектов `store`, то другая часть приложения, которая хочет сделать то же самое, должна ждать: новая транзакция «зависает» до завершения первой. Это может привести к странным задержкам, если транзакции слишком долго выполняются.

Что же делать?

Лучше выполнять операции вместе, в рамках одной транзакции: отделить транзакции IndexedDB от других асинхронных операций.

Чтобы поймать момент успешного выполнения, мы можем повесить обработчик на событие `transaction.oncomplete`:

~~~
const transaction = db.transaction("MyObjectStore", "readwrite");

// ...выполнить операции...

transaction.oncomplete = function() {
  console.log("Транзакция выполнена");
};
~~~

Только `complete` гарантирует, что транзакция сохранена целиком. По отдельности запросы могут выполниться, но при финальной записи что-то может пойти не так (ошибка ввода-вывода, проблема с диском, например).

Чтобы вручную отменить транзакцию, выполните:

~~~
transaction.abort();
Это отменит все изменения, сделанные запросами в транзакции, и сгенерирует событие transaction.onabort.
~~~

### Обработка ошибок

Запросы на запись могут выполниться неудачно.

Мы должны быть готовы к этому, не только из-за возможных ошибок на нашей стороне, но и по причинам, которые не связаны с транзакцией. Например, размер хранилища может быть превышен. И мы должны быть готовы обработать такую ситуацию.

При ошибке в запросе соответствующая транзакция отменяется полностью, включая изменения, сделанные другими её запросами.

Если мы хотим продолжить транзакцию (например, попробовать другой запрос без отмены изменений), это также возможно. Для этого в обработчике `request.onerror` следует вызвать `event.preventDefault()`.

В примере ниже новая книга добавляется с тем же ключом (id), что и существующая. Метод `store.add` генерирует в этом случае ошибку `ConstraintError`. Мы обрабатываем её без отмены транзакции:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", ["name.last", "name.first"]);
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");

  const user = { id: 12345, name: { first: "John", last: "Doe" }, age: 42 };

  store.put(user);
  const request = store.add(user);

  request.onerror = function (event) {
    // ConstraintError возникает при попытке добавить объект с ключом, который уже существует
    if (request.error.name == "ConstraintError") {
      console.log("Книга с таким id уже существует"); // обрабатываем ошибку
      event.preventDefault(); // предотвращаем отмену транзакции и не идем на onabort
      // ...можно попробовать использовать другой ключ...
    } else {
      // неизвестная ошибка
      // транзакция будет отменена
      console.log("error");
    }
  };

  tx.onabort = function () {
    console.log("Ошибка", tx.error);
  };

  tx.oncomplete = function () {
    console.log("complete");
  };
};
// Книга с таким id уже существует
// complete
~~~

##### Делегирование событий

Нужны ли обработчики `onerror`/`onsuccess` для каждого запроса? Не всегда. Мы можем использовать делегирование событий.

События IndexedDB всплывают: `запрос → транзакция → база данных`.

Все события являются DOM-событиями с фазами перехвата и всплытия, но обычно используется только всплытие.

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", ["name.last", "name.first"]);
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");

  tx.onerror = function () {
    console.log("Error in transaction");
  };

  const user = { id: 12345, name: { first: "John", last: "Doe" }, age: 42 };

  store.add(user);
  const request = store.add(user);

  request.onerror = function () {
    if (request.error.name == "ConstraintError") {
      console.log("Книга с таким id уже существует");
    } else {
      console.log("Error in request");
    }
  };
};
// Error in transaction
// Error in request
// Error in transaction
~~~

В большинстве случаев мы можем перехватить все ошибки, используя обработчик `db.onerror`, для оповещения пользователя или других целей.

…А если мы полностью обработали ошибку? В этом случае мы не хотим сообщать об этом.

Мы можем остановить всплытие, используя `event.stopPropagation()`.

### Поиск по ключам

Есть два основных вида поиска в хранилище объектов:

1. По значению ключа или диапазону ключей. Например, в хранилище «books» это будет значение или диапазон значений `book.id`.
2. С помощью другого поля объекта, например `book.price`. Для этого потребовалась дополнительная структура данных, получившая название «index».

##### По ключу

Сначала давайте разберёмся с первым типом поиска: по ключу.

Методы поиска поддерживают либо точные ключи, либо так называемые «запросы с диапазоном» – IDBKeyRange объекты, которые задают «диапазон ключей».

Диапазоны создаются с помощью следующих вызовов:

* `IDBKeyRange.lowerBound(lower, [open])` означает: `>lower` (или `≥lower`, если `open` это `true`)
* `IDBKeyRange.upperBound(upper, [open])` означает: `<upper` (или `≤upper`, если `open` это `true`)
* `IDBKeyRange.bound(lower, upper, [lowerOpen], [upperOpen])` означает: между `lower` и `upper`, включительно, если соответствующий `open` равен `true`.
* `IDBKeyRange.only(key)` – диапазон, который состоит только из одного ключа `key`, редко используется.

Очень скоро мы увидим практические примеры их использования.

Для выполнения фактического поиска существуют следующие методы. Они принимают аргумент `query`, который может быть либо точным ключом, либо диапазоном ключей:

* `store.get(query)` – поиск первого значения по ключу или по диапазону.
* `store.getAll([query], [count])` – поиск всех значений, можно ограничить, передав `count`.
* `store.getKey(query)` – поиск первого ключа, который удовлетворяет запросу, обычно передаётся диапазон.
* `store.getAllKeys([query], [count])` – поиск всех ключей, которые удовлетворяют запросу, обычно передаётся диапазон, возможно ограничить поиск, передав `count`.
* `store.count([query])` – получить общее количество ключей, которые удовлетворяют запросу, обычно передаётся диапазон.

Например, в хранилище у нас есть множество пользователей. Помните, поле `id` является ключом, поэтому все эти методы могут искать по ключу `id`.

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  db.createObjectStore("MyObjectStore", { keyPath: "id" });
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.put({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });
  store.put({ id: 7890, name: { first: "Mary", last: "Jackson" }, age: 34 });
  store.put({ id: 890, name: { first: "Anna", last: "Smith" }, age: 32 });

  const getJohn = store.get(12345);
  const getAll = store.getAll(IDBKeyRange.bound(1, 10000));
  const getKey = store.getKey(IDBKeyRange.bound(1, 10000));

  getJohn.onsuccess = function () {
    console.log(getJohn.result.name.first); // => "John"
  };

  getAll.onsuccess = function () {
    console.log(getAll.result); // => [{id: 890, name: {…}, age: 32}, {id: 7890, name: {…}, age: 34}]
  };

  getKey.onsuccess = function () {
    console.log(getKey.result); // 890
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

> Хранилище объектов всегда отсортировано. Хранилище объектов внутренне сортирует значения по ключам. Поэтому запросы, которые возвращают много значений, всегда возвращают их в порядке сортировки по ключу.

##### Поиск по индексированному полю

Для поиска по другим полям объекта нам нужно создать дополнительную структуру данных, называемую «индекс» (index).

Индекс является «расширением» к хранилищу, которое отслеживает данное поле объекта. Для каждого значения этого поля хранится список ключей для объектов, которые имеют это значение. Ниже будет более подробная картина.

Синтаксис:

~~~
objectStore.createIndex(name, keyPath, [options]);
~~~

* `name` – название индекса,
* `keyPath` – путь к полю объекта, которое индекс должен отслеживать (мы собираемся сделать поиск по этому полю),
* `option` – необязательный объект со свойствами:
  * `unique` – если `true`, тогда в хранилище может быть только один объект с заданным значением в `keyPath`. Если мы попытаемся добавить дубликат, то индекс сгенерирует ошибку.
  * `multiEntry` – имеет смысл только, если `keyPath` является массивом. Если `false` (по умолчанию), то для каждого ключа, являющегося массивом, имеется одна запись. Но если мы укажем `true` в `multiEntry`, тогда индекс будет хранить список объектов хранилища для каждого значения в этом массиве. Таким образом, элементы массива становятся ключами индекса.

Индексы должны создаваться в `upgradeneeded`, как и хранилище объектов:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", "names", { unique: false, multiEntry: true });
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const index = store.index("NameIndex");

  store.put({ id: 12345, names: ["John", "Doe"], age: 42 });
  store.put({ id: 67890, names: ["Bob", "Smith"], age: 35 });
  store.put({ id: 7890, names: ["Mary", "Jackson"], age: 34 });
  store.put({ id: 890, names: ["Anna", "Smith"], age: 32 });

  const getBob = index.get("Bob");

  getBob.onsuccess = function () {
    console.log(getBob.result.names); // ['Bob', 'Smith']
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

Индекс будет отслеживать поле `names`.
Поле `names` не уникальное, у нас может быть несколько книг с одинаковой ценой, поэтому мы не устанавливаем опцию `unique`.
Поле `names` является массивом, поэтому применяем флаг `multiEntry`.

Пример с `multiEntry: false`:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", ["name.last", "name.first"], {
    unigue: false,
    multiEntry: false,
  });
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const index = store.index("NameIndex");

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.put({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });
  store.put({ id: 7890, name: { first: "Mary", last: "Jackson" }, age: 34 });
  store.put({ id: 890, name: { first: "Anna", last: "Smith" }, age: 32 });

  const getBob = index.get(["Smith", "Bob"]);

  getBob.onsuccess = function () {
    console.log(getBob.result.name.first); // => "Bob"
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

### Удаление из хранилища

Метод `delete` удаляет значения по запросу, формат вызова такой же как в `getAll`:

`delete(query)` – производит удаление соответствующих запросу значений.

Например:

Удалить пользователя с `id=12345`:

~~~
store.delete(12345);
~~~

Если нам нужно удалить пользователей, основываясь на определенном поле, сначала нам надо найти ключ в индексе, а затем выполнить `delete`:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", ["name.last", "name.first"]);
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const index = store.index("NameIndex");

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.put({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });
  store.put({ id: 7890, name: { first: "Mary", last: "Jackson" }, age: 34 });
  store.put({ id: 890, name: { first: "Anna", last: "Smith" }, age: 32 });

  const getBob = index.getKey(["Smith", "Bob"]);

  getBob.onsuccess = function () {
    console.log(getBob.result); // 67890
    store.delete(getBob.result);
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

Чтобы удалить всё:

~~~
store.clear(); // очищаем хранилище
~~~

### Курсоры

Такие методы как `getAll`/`getAllKeys` возвращают массив ключей/значений.

Но хранилище объектов может быть огромным, больше, чем доступно памяти. Тогда метод `getAll` вернёт ошибку при попытке получить все записи в массиве.

Что делать?

Курсоры предоставляют возможности для работы в таких ситуациях.

__Объект `cursor` идёт по хранилищу объектов с заданным запросом `query` и возвращает пары ключ/значение по очереди, а не все сразу. Это позволяет экономить память.__

Так как хранилище объектов внутренне отсортировано по ключу, курсор проходит по хранилищу в порядке хранения ключей (по возрастанию по умолчанию).

Синтаксис:

~~~
// как getAll, но с использованием курсора:
const request = store.openCursor([query], [direction]);

// чтобы получить ключи, не значения (как getAllKeys): store.openKeyCursor
~~~

* `query` - ключ или диапазон ключей, как для `getAll`.
* `direction` - необязательный аргумент, доступные значения:
  * `next` – по умолчанию, курсор будет проходить от самого маленького ключа к большему.
  * `prev` – обратный порядок: от самого большого ключа к меньшему.
  * `nextunique`, `prevunique` – то же самое, но курсор пропускает записи с тем же ключом, что уже был (только для курсоров по индексам, например, для нескольких книг с `price=5`, будет возвращена только первая).

Основным отличием курсора является то, что `request.onsuccess` генерируется многократно: один раз для каждого результата.

Вот пример того, как использовать курсор:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  db.createObjectStore("MyObjectStore", { keyPath: "id" });
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const request = store.openCursor();

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.put({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });
  store.put({ id: 7890, name: { first: "Mary", last: "Jackson" }, age: 34 });
  store.put({ id: 890, name: { first: "Anna", last: "Smith" }, age: 32 });

  request.onsuccess = function () {
    const cursor = request.result;
    if (cursor) {
      const key = cursor.key; 
      const value = cursor.value;
      console.log(key, value); // 890 {id: 890, name: {…}, age: 32} ...
      cursor.continue();
    } else {
      console.log("Пользователей больше нет");
    }
  };

  tx.oncomplete = function () {
    db.close();
  };
};
~~~

Основные методы курсора:

* `advance(count)` – продвинуть курсор на `count` позиций, пропустив значения.
* `continue([key])` – продвинуть курсор к следующему значению в диапазоне соответствия (или до позиции сразу после ключа `key`, если указан).

Независимо от того, есть ли ещё значения, соответствующие курсору или нет – вызывается `onsuccess`, затем в `result` мы можем получить курсор, указывающий на следующую запись или равный `undefined`.

В приведённом выше примере курсор был создан для хранилища объектов.

Но мы также можем создать курсор для индексов. Как мы помним, индексы позволяют искать по полю объекта. Курсоры для индексов работают так же, как для хранилищ объектов – они позволяют экономить память, возвращая одно значение в единицу времени.

Для курсоров по индексам `cursor.key` является ключом индекса, и нам следует использовать свойство `cursor.primaryKey` как ключ объекта:

~~~
const open = indexedDB.open("MyDatabase", 1);

open.onupgradeneeded = function () {
  const db = open.result;
  const store = db.createObjectStore("MyObjectStore", { keyPath: "id" });
  store.createIndex("NameIndex", "name.first");
};

open.onsuccess = function () {
  const db = open.result;
  const tx = db.transaction("MyObjectStore", "readwrite");
  const store = tx.objectStore("MyObjectStore");
  const index = store.index("NameIndex");
  const request = index.openCursor(IDBKeyRange.upperBound("Jack"));

  store.put({ id: 12345, name: { first: "John", last: "Doe" }, age: 42 });
  store.put({ id: 67890, name: { first: "Bob", last: "Smith" }, age: 35 });
  store.put({ id: 7890, name: { first: "Mary", last: "Jackson" }, age: 34 });
  store.put({ id: 890, name: { first: "Anna", last: "Smith" }, age: 32 });

  request.onsuccess = function () {
    const cursor = request.result;
    if (cursor) {
      const key = cursor.primaryKey; // ключ в хранилище объектов (поле id)
      const value = cursor.value; // значение в хранилище объектов (объект "user")
      const keyIndex = cursor.key; // ключ индекса (name.first)
      console.log(key, value, keyIndex);
      cursor.continue();
    } else {
      console.log("Пользователей больше нет");
    }
  };

  tx.oncomplete = function () {
    db.close();
  };
};

// 890 {id: 890, name: {…}, age: 32} 'Anna'
// 67890 {id: 67890, name: {…}, age: 35} 'Bob'
// Пользователей больше нет
~~~

### Обёртка для промисов

Добавлять к каждому запросу `onsuccess`/`onerror` немного громоздко. Мы можем сделать нашу жизнь проще, используя делегирование событий, например, установить обработчики на все транзакции, но использовать `async`/`await` намного удобнее.

Давайте далее в главе использовать небольшую обёртку над промисами [https://github.com/jakearchibald/idb](https://github.com/jakearchibald/idb). Она создаёт глобальный `idb` объект с промисифицированными IndexedDB методами.

Тогда вместо `onsuccess`/`onerror` мы можем писать примерно так:

~~~
<script src="https://cdn.jsdelivr.net/npm/idb@7/build/umd-with-async-ittr.js"></script>
<script>
  async function demo() {
    const db = await idb.openDB("Articles", 1, {
      upgrade(db) {
        const store = db.createObjectStore("articles", {
          keyPath: "id",
          autoIncrement: true,
        });
        store.createIndex("date", "date");
      },
    });

    await db.add("articles", {
      title: "Article 1",
      date: new Date("2019-01-01"),
      body: "…",
    });

    {
      const tx = db.transaction("articles", "readwrite");
      await Promise.all([
        tx.store.add({
          title: "Article 2",
          date: new Date("2019-01-01"),
          body: "…",
        }),
        tx.store.add({
          title: "Article 3",
          date: new Date("2019-01-02"),
          body: "…",
        }),
        tx.done,
      ]);
    }

    console.log(await db.getAllFromIndex("articles", "date"));

    {
      const tx = db.transaction("articles", "readwrite");
      const index = tx.store.index("date");

      for await (const cursor of index.iterate(new Date("2019-01-01"))) {
        const article = { ...cursor.value };
        article.body += " And, happy new year!";
        cursor.update(article);
      }

      await tx.done;
    }
  }
  demo();
</script>

// [{…}, {…}, {…}]
~~~

##### Обработка ошибок

Если мы не перехватим ошибку, то она «вывалится» наружу, вверх по стеку вызовов, до ближайшего внешнего try..catch.

Необработанная ошибка становится событием «unhandled promise rejection» в объекте window.

Мы можем обработать такие ошибки вот так:

~~~
window.addEventListener('unhandledrejection', event => {
  const request = event.target; // объект запроса IndexedDB
  const error = event.reason; //  Необработанный объект ошибки, как request.error
  ...сообщить об ошибке...
});
~~~

##### Подводный камень: «Inactive transaction»

Как мы уже знаем, транзакции автоматически завершаются, как только браузер завершает работу с текущим кодом и макрозадачу. Поэтому, если мы поместим макрозадачу наподобие `fetch` в середину транзакции, транзакция не будет ожидать её завершения. Произойдёт автозавершение транзакции. Поэтому при следующем запросе возникнет ошибка.

Для промисифицирующей обёртки и `async`/`await` поведение такое же.

Решение такое же, как при работе с обычным IndexedDB: либо создать новую транзакцию, либо разделить задачу на части.

Подготовить данные и получить всё, что необходимо. Затем сохранить в базу данных.
