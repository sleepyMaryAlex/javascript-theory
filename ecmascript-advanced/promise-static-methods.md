# ?Promise static methods

В классе `Promise` есть 6 статических методов.

### `Promise.all`

Допустим, нам нужно запустить множество промисов параллельно и дождаться, пока все они выполнятся.

Например, параллельно загрузить несколько файлов и обработать результат, когда он готов.

Синтаксис:

~~~
const promise = Promise.all(iterable);
~~~

Метод `Promise.all` принимает массив промисов (может принимать любой перебираемый объект, но обычно используется массив) и возвращает новый промис.

Новый промис завершится, когда завершится весь переданный список промисов, и его результатом будет массив их результатов.

Например, `Promise.all`, представленный ниже, выполнится спустя 3 секунды, его результатом будет массив `[1, 2, 3]`:

~~~
Promise.all([
  new Promise((resolve) => setTimeout(() => resolve(1), 3000)), // 1
  new Promise((resolve) => setTimeout(() => resolve(2), 2000)), // 2
  new Promise((resolve) => setTimeout(() => resolve(3), 1000)), // 3
]).then(console.log); // когда все промисы выполнятся, результат будет 1,2,3
// каждый промис даёт элемент массива
~~~

Обратите внимание, что порядок элементов массива в точности соответствует порядку исходных промисов. Даже если первый промис будет выполняться дольше всех, его результат всё равно будет первым в массиве.

Часто применяемый трюк – пропустить массив данных через `map`-функцию, которая для каждого элемента создаст задачу-промис, и затем обернуть получившийся массив в `Promise.all`.

Например, если у нас есть массив ссылок, то мы можем загрузить их вот так:

~~~
const urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/remy",
  "https://api.github.com/users/jeresig",
];

// Преобразуем каждый URL в промис, возвращённый fetch
const requests = urls.map((url) => fetch(url));

// Promise.all будет ожидать выполнения всех промисов
Promise.all(requests).then((responses) =>
  responses.forEach((response) => console.log(response))
);
~~~

А вот пример побольше, с получением информации о пользователях GitHub по их логинам из массива (мы могли бы получать массив товаров по их идентификаторам, логика та же):

~~~
const names = ["iliakan", "remy", "jeresig"];

const requests = names.map((name) =>
  fetch(`https://api.github.com/users/${name}`)
);

Promise.all(requests)
  .then((responses) => {
    for (let response of responses) {
      console.log(`${response.url}: ${response.status}`);
    }
    return responses;
  })
  .then((responses) =>
    Promise.all(responses.map((response) => response.json()))
  )
  .then((users) => users.forEach((user) => console.log(user.name)));
~~~

Если любой из промисов завершится с ошибкой, то промис, возвращённый `Promise.all`, немедленно завершается с этой ошибкой.

~~~
Promise.all([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]).catch(console.log); // Error: Ошибка!
~~~

Здесь второй промис завершится с ошибкой через 2 секунды. Это приведёт к немедленной ошибке в `Promise.all`, так что выполнится `.catch`: ошибка этого промиса становится ошибкой всего `Promise.all`.

##### В случае ошибки, остальные результаты игнорируются

Если один промис завершается с ошибкой, то весь `Promise.all` завершается с ней, полностью забывая про остальные промисы в списке. Их результаты игнорируются.

Например, если сделано несколько вызовов `fetch`, как в примере выше, и один не прошёл, то остальные будут всё ещё выполняться, но `Promise.all` за ними уже не смотрит. Скорее всего, они так или иначе завершатся, но их результаты будут проигнорированы.

`Promise.all` ничего не делает для их отмены, так как в промисах вообще нет концепции «отмены».

##### `Promise.all(iterable)` разрешает передавать не-промисы в `iterable` (перебираемом объекте)

Обычно, `Promise.all(...)` принимает перебираемый объект промисов (чаще всего массив). Но если любой из этих объектов не является промисом, он передаётся в итоговый массив «как есть».

Например, здесь результат: `[1, 2, 3]`

~~~
Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000);
  }),
  2,
  3,
]).then(console.log); // 1, 2, 3
~~~

Таким образом, мы можем передавать уже готовые значения, которые не являются промисами, в `Promise.all`, иногда это бывает удобно.

### `Promise.allSettled`

Эта возможность была добавлена в язык недавно. В старых браузерах может понадобиться полифил.

Синтаксис:

~~~
const promise = Promise.allSettled(iterable);
~~~

Метод `Promise.allSettled` всегда ждёт завершения всех промисов. В массиве результатов будет

* `{status: "fulfilled", value: результат}` для успешных завершений,
* `{status: "rejected", reason: ошибка}` для ошибок.

Например, мы хотели бы загрузить информацию о множестве пользователей. Даже если в каком-то запросе ошибка, нас всё равно интересуют остальные.

~~~
const urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/remy",
  "https://no-such-url",
];

Promise.allSettled(urls.map((url) => fetch(url))).then((results) => {
  console.log(results); // (*)
  results.forEach((result, num) => {
    if (result.status == "fulfilled") {
      console.log(`${urls[num]}: ${result.value.status}`);
    }
    if (result.status == "rejected") {
      console.log(`${urls[num]}: ${result.reason}`);
    }
  });
});
~~~

Массив `results` в строке `(*)` будет таким:

~~~
[
  {status: 'fulfilled', value: Response},
  {status: 'fulfilled', value: Response},
  {status: 'rejected', reason: TypeError: Failed to fetch...}
]
~~~

То есть, для каждого промиса у нас есть его статус и значение/ошибка.

Если браузер не поддерживает `Promise.allSettled`, для него легко сделать полифил:

~~~
const urls = [
  "https://api.github.com/users/iliakan",
  "https://api.github.com/users/remy",
  "https://no-such-url",
];

if (!Promise.allSettled) {
  Promise.allSettled = function (promises) {
    return Promise.all(
      promises.map((promise) =>
        Promise.resolve(promise).then(
          (value) => ({
            status: "fulfilled",
            value: value,
          }),
          (error) => ({
            status: "rejected",
            reason: error,
          })
        )
      )
    );
  };
}

Promise.allSettled(urls.map((url) => fetch(url))).then((results) => {
  console.log(results);
});
~~~

В этом коде `promises.map` берёт аргументы, превращает их в промисы (на всякий случай) и добавляет каждому обработчик `.then`.

Этот обработчик превращает успешный результат `value` в `{state:'fulfilled', value: value}`, а ошибку `error` в `{state:'rejected', reason: error}`. Это как раз и есть формат результатов `Promise.allSettled`.

Затем мы можем использовать `Promise.allSettled`, чтобы получить результаты всех промисов, даже если при выполнении какого-то возникнет ошибка.

### `Promise.race`

Метод очень похож на `Promise.all`, но ждёт только первый выполненный промис, из которого берёт результат (или ошибку).

Синтаксис:

~~~
const promise = Promise.race(iterable);
~~~

Например, тут результат будет 1:

~~~
Promise.race([
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 2000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]).then(console.log); // 1
~~~

Быстрее всех выполнился первый промис, он и дал результат. После этого остальные промисы игнорируются.

### `Promise.any`

Метод очень похож на `Promise.race`, но ждёт только первый успешно выполненный промис, из которого берёт результат.

Если ни один из переданных промисов не завершится успешно, тогда возвращённый объект `Promise` будет отклонён с помощью `AggregateError` – специального объекта ошибок, который хранит все ошибки промисов в своём свойстве `errors`.

Синтаксис:

~~~
const promise = Promise.any(iterable);
~~~

Например, здесь, результатом будет 1:

~~~
Promise.any([
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 1000)
  ),
  new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000)),
]).then(console.log); // 1
~~~

Первый промис в этом примере был самым быстрым, но он был отклонён, поэтому результатом стал второй. После того, как первый успешно выполненный промис «выиграет гонку», все дальнейшие результаты будут проигнорированы.

Вот пример, в котором все промисы отклоняются:

~~~
Promise.any([
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ошибка!")), 1000)
  ),
  new Promise((resolve, reject) =>
    setTimeout(() => reject(new Error("Ещё одна ошибка!")), 2000)
  ),
]).catch((error) => {
  console.log(error.constructor.name); // AggregateError
  console.log(error.errors[0]); // Error: Ошибка!
  console.log(error.errors[1]); // Error: Ещё одна ошибка!
});
~~~

Как вы можете видеть, объекты ошибок для отклонённых промисов доступны в свойстве `errors` объекта `AggregateError`.

### `Promise.resolve`/`reject`

Методы `Promise.resolve` и `Promise.reject` редко используются в современном коде, так как синтаксис `async`/`await` делает их, в общем-то, не нужными.

Мы рассмотрим их здесь для полноты картины, а также для тех, кто по каким-то причинам не может использовать `async`/`await`.

#### `Promise.resolve`

`Promise.resolve(value)` создаёт успешно выполненный промис с результатом `value`.

То же самое, что:

~~~
const promise = new Promise(resolve => resolve(value));
~~~

Этот метод используют для совместимости: когда ожидается, что функция возвратит именно промис.

Например, функция `loadCached` ниже загружает URL и запоминает (кеширует) его содержимое. При будущих вызовах с тем же URL он тут же читает предыдущее содержимое из кеша, но использует `Promise.resolve`, чтобы сделать из него промис, для того, чтобы возвращаемое значение всегда было промисом:

~~~
const cache = new Map();

function loadCached(url) {
  if (cache.has(url)) {
    return Promise.resolve(cache.get(url)); // (*)
  }

  return fetch(url)
    .then((response) => response.text())
    .then((text) => {
      cache.set(url, text);
      return text;
    });
}

loadCached("https://api.github.com/users/iliakan");
console.log(cache);
~~~

Мы можем писать `loadCached(url).then(…)`, потому что функция `loadCached` всегда возвращает промис. Мы всегда можем использовать `.then` после `loadCached`. Это и есть цель использования `Promise.resolve` в строке `(*)`.

#### `Promise.reject`

`Promise.reject(error)` создаёт промис, завершённый с ошибкой `error`.

То же самое, что:

~~~
const promise = new Promise((resolve, reject) => reject(error));
~~~

На практике этот метод почти никогда не используется.
