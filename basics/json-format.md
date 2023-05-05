# ?JSON format

Допустим, у нас есть сложный объект, и мы хотели бы преобразовать его в строку, чтобы отправить по сети или просто вывести для логирования.

Естественно, такая строка должна включать в себя все важные свойства.

Мы могли бы реализовать преобразование следующим образом:

~~~
const user = {
  name: "John",
  age: 30,

  toString() {
    return `{name: "${this.name}", age: ${this.age}}`;
  }
};

console.log(String(user)); // {name: "John", age: 30}. Без toString => [object Object]
~~~

…Но в процессе разработки добавляются новые свойства, старые свойства переименовываются и удаляются. Обновление такого toString каждый раз может стать проблемой. Мы могли бы попытаться перебрать свойства в нём, но что, если объект сложный, и в его свойствах имеются вложенные объекты? Мы должны были бы осуществить их преобразование тоже.

К счастью, нет необходимости писать код для обработки всего этого. У задачи есть простое решение.

### `JSON.stringify`

_JSON (JavaScript Object Notation)_ – это общий формат для представления значений и объектов. Его описание задокументировано в стандарте RFC 4627. Первоначально он был создан для JavaScript, но многие другие языки также имеют библиотеки, которые могут работать с ним. Таким образом, JSON легко использовать для обмена данными, когда клиент использует JavaScript, а сервер написан на Ruby/PHP/Java или любом другом языке.

JavaScript предоставляет методы:

* `JSON.stringify` для преобразования объектов в JSON.
* `JSON.parse` для преобразования JSON обратно в объект.

Например, здесь мы преобразуем через `JSON.stringify` данные студента:

~~~
const student = {
  name: 'John',
  age: 30,
  isAdmin: false,
  courses: ['html', 'css', 'js'],
  wife: null
};

const json = JSON.stringify(student);

console.log(typeof json); // string

console.log(json);

// выведет объект в формате JSON:
// {"name":"John","age":30,"isAdmin":false,"courses":["html","css","js"],"wife":null}
~~~

Метод `JSON.stringify(student)` берёт объект и преобразует его в строку.

Полученная строка `json` называется JSON-форматированным или сериализованным объектом. Мы можем отправить его по сети или поместить в обычное хранилище данных.

Обратите внимание, что объект в формате JSON имеет несколько важных отличий от объектного литерала:

* Строки используют двойные кавычки. Никаких одинарных кавычек или обратных кавычек в JSON. Так `'John'` становится `"John"`.
* Имена свойств объекта также заключаются в двойные кавычки. Это обязательно. Так `age: 30` становится `"age":30`.

`JSON.stringify` может быть применён и к примитивам.

JSON поддерживает следующие типы данных:

* Объекты `{ ... }`
* Массивы `[ ... ]`
* Примитивы:
  * строки
  * числа
  * логические значения `true`/`false`
  * `null`

~~~
console.log(JSON.stringify(1));
console.log(JSON.stringify("test"));
console.log(JSON.stringify(true));
console.log(JSON.stringify([1, 2, 3]));

// 1
// "test"
// true
// [1,2,3]
~~~

JSON является независимой от языка спецификацией для данных, поэтому `JSON.stringify` пропускает некоторые специфические свойства объектов JavaScript.

А именно:

* Свойства-функции (методы).
* Символьные ключи и значения.
* Свойства, содержащие `undefined`.

~~~
const user = {
  sayHi() {
    // будет пропущено
    console.log("Hello");
  },
  [Symbol("id")]: 123, // также будет пропущено
  something: undefined, // как и это - пропущено
};

console.log(JSON.stringify(user)); // {} (пустой объект)
~~~

Обычно это нормально. Если это не то, чего мы хотим, то скоро мы увидим, как можно настроить этот процесс.

Самое замечательное, что вложенные объекты поддерживаются и конвертируются автоматически.

~~~
const meetup = {
  title: "Conference",
  room: {
    number: 23,
    participants: ["john", "ann"],
  },
};

console.log(JSON.stringify(meetup));
// вся структура преобразована в строку:
// {"title":"Conference","room":{"number":23,"participants":["john","ann"]}}
~~~

Важное ограничение: не должно быть циклических ссылок.

~~~
const room = {
  number: 23
};

const meetup = {
  title: "Conference",
  participants: ["john", "ann"]
};

meetup.place = room; // meetup ссылается на room
room.occupiedBy = meetup; // room ссылается на meetup

JSON.stringify(meetup); // TypeError: Converting circular structure to JSON
~~~

#### Исключаем и преобразуем: `replacer`

Полный синтаксис `JSON.stringify`:

~~~
const json = JSON.stringify(value, [replacer, space])
~~~

* `value` - значение для кодирования.
* `replacer` - массив свойств для кодирования или функция соответствия `function(key, value)`.
* `space` - дополнительное пространство (отступы), используемое для форматирования.

В большинстве случаев `JSON.stringify` используется только с первым аргументом. Но если нам нужно настроить процесс замены, например, отфильтровать циклические ссылки, то можно использовать второй аргумент `JSON.stringify`.

Если мы передадим ему массив свойств, будут закодированы только эти свойства.

~~~
const room = {
  number: 23,
};

const meetup = {
  title: "Conference",
  participants: [{ name: "John" }, { name: "Alice" }],
  place: room, // meetup ссылается на room
};

room.occupiedBy = meetup; // room ссылается на meetup

console.log(JSON.stringify(meetup, ['title', 'participants', 'place', 'name', 'number']));
// {"title":"Conference","participants":[{"name":"John"},{"name":"Alice"}],"place":{"number":23}}
~~~

Здесь всё, кроме `occupiedBy`, сериализовано. Но список свойств довольно длинный.

К счастью, в качестве `replacer` мы можем использовать функцию, а не массив.

Функция будет вызываться для каждой пары `(key, value)`, и она должна возвращать заменённое значение, которое будет использоваться вместо исходного. Или `undefined`, чтобы пропустить значение.

В нашем случае мы можем вернуть `value` «как есть» для всего, кроме `occupiedBy`. Чтобы игнорировать `occupiedBy`, код ниже возвращает `undefined`:

~~~
const room = {
  number: 23,
};

const meetup = {
  title: "Conference",
  participants: [{ name: "John" }, { name: "Alice" }],
  place: room, // meetup ссылается на room
};

room.occupiedBy = meetup; // room ссылается на meetup

console.log(
  JSON.stringify(meetup, function replacer(key, value) {
    console.log(`${key}: ${value}`);
    return key == "occupiedBy" ? undefined : value;
  })
);

/* пары ключ:значение, которые приходят в replacer:
:             [object Object]
title:        Conference
participants: [object Object],[object Object]
0:            [object Object]
name:         John
1:            [object Object]
name:         Alice
place:        [object Object]
number:       23
occupiedBy: [object Object]
*/
// {"title":"Conference","participants":[{"name":"John"},{"name":"Alice"}],"place":{"number":23}}
~~~

Обратите внимание, что функция `replacer` получает каждую пару ключ/значение, включая вложенные объекты и элементы массива. И она применяется рекурсивно. Значение `this` внутри `replacer` – это объект, который содержит текущее свойство.

Первый вызов – особенный. Ему передаётся специальный «объект-обёртка»: `{"": meetup}`. Другими словами, первая `(key, value)` пара имеет пустой ключ, а значением является целевой объект в общем. Вот почему первая строка из примера выше будет `":[object Object]"`.

Идея состоит в том, чтобы дать как можно больше возможностей `replacer` – у него есть возможность проанализировать и заменить/пропустить даже весь объект целиком, если это необходимо.

#### Форматирование: `space`

Третий аргумент в `JSON.stringify(value, replacer, space)` – это количество пробелов, используемых для удобного форматирования.

Ранее все JSON-форматированные объекты не имели отступов и лишних пробелов. Это нормально, если мы хотим отправить объект по сети. Аргумент `space` используется исключительно для вывода в удобочитаемом виде.

Ниже `space = 2` указывает JavaScript отображать вложенные объекты в несколько строк с отступом в 2 пробела внутри объекта:

~~~
const user = {
  name: "John",
  age: 25,
  roles: {
    isAdmin: false,
    isEditor: true
  }
};

console.log(JSON.stringify(user, null, 2));
/* отступ в 2 пробела:
{
  "name": "John",
  "age": 25,
  "roles": {
    "isAdmin": false,
    "isEditor": true
  }
}
*/
~~~

Третьим аргументом также может быть строка. В этом случае строка будет использоваться для отступа вместо ряда пробелов.

Параметр `space` применяется исключительно для логирования и красивого вывода.

#### Метод `toJSON`

Как и `toString` для преобразования строк, объект может предоставлять метод `toJSON` для преобразования в JSON. `JSON.stringify` автоматически вызывает его, если он есть.

~~~
const room = {
  number: 23,
};

const meetup = {
  title: "Conference",
  date: new Date(Date.UTC(2017, 0, 1)),
  room,
};

console.log(JSON.stringify(meetup));
// {"title":"Conference","date":"2017-01-01T00:00:00.000Z","room":{"number":23}}
~~~

Как видим, `date` стал строкой. Это потому, что все объекты типа `Date` имеют встроенный метод `toJSON`, который возвращает такую строку.

Теперь давайте добавим собственную реализацию метода `toJSON` в наш объект `room`:

~~~
const room = {
  number: 23,
  toJSON() {
    return this.number;
  },
};

const meetup = {
  title: "Conference",
  room,
};

console.log(JSON.stringify(room)); // 23
console.log(JSON.stringify(meetup)); // {"title":"Conference","room":23}
~~~

Как видите, `toJSON` используется как при прямом вызове `JSON.stringify(room)`, так и когда `room` вложен в другой сериализуемый объект.

### `JSON.parse`

Чтобы декодировать JSON-строку, нам нужен другой метод с именем `JSON.parse`.

Синтаксис:

~~~
const value = JSON.parse(str, [reviver]);
~~~

* `str` - JSON для преобразования в объект.
* `reviver` - необязательная функция, которая будет вызываться для каждой пары (ключ, значение) и может преобразовывать значение.

Например:

~~~
// строковый массив
let numbers = "[0, 1, 2, 3]";

numbers = JSON.parse(numbers);

console.log(numbers[1]); // 1
~~~

Или для вложенных объектов:

~~~
let user =
  '{ "name": "John", "age": 35, "isAdmin": false, "friends": [0,1,2,3] }';

user = JSON.parse(user);

console.log(user.friends[1]); // 1
~~~

JSON может быть настолько сложным, насколько это необходимо, объекты и массивы могут включать другие объекты и массивы. Но они должны быть в том же JSON-формате.

Вот типичные ошибки в написанном от руки JSON (иногда приходится писать его для отладки):

~~~
const json = `{
  name: "John",                     // Ошибка: имя свойства без кавычек
  "surname": 'Smith',               // Ошибка: одинарные кавычки в значении (должны быть двойными)
  'isAdmin': false                  // Ошибка: одинарные кавычки в ключе (должны быть двойными)
  "birthday": new Date(2000, 2, 3), // Ошибка: не допускается конструктор "new", только значения.
  "friends": [0,1,2,3]              // Здесь всё в порядке
}`;
~~~

Кроме того, JSON не поддерживает комментарии. Добавление комментария в JSON делает его недействительным.

Существует ещё один формат JSON5, который поддерживает ключи без кавычек, комментарии и т.д. Но это самостоятельная библиотека, а не спецификация языка.

Обычный JSON настолько строг не потому, что его разработчики ленивы, а потому, что позволяет легко, надёжно и очень быстро реализовывать алгоритм кодирования и чтения.

#### Использование `reviver`

Представьте, что мы получили объект с сервера в виде строки данных.

И нам нужно десериализовать её, т.е. снова превратить в объект JavaScript.

Давайте сделаем это, вызвав `JSON.parse`:

~~~
const str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

const meetup = JSON.parse(str);

console.log(meetup.date.getDate()); // TypeError: meetup.date.getDate is not a function
~~~

Ой, ошибка!

Значением `meetup.date` является строка, а не `Date` объект. Как `JSON.parse` мог знать, что он должен был преобразовать эту строку в `Date`?

Давайте передадим `JSON.parse` функцию восстановления вторым аргументом, которая возвращает все значения «как есть», и `date` станет `Date`:

~~~
const str = '{"title":"Conference","date":"2017-11-30T12:00:00.000Z"}';

const meetup = JSON.parse(str, function (key, value) {
  if (key == "date") {
    return new Date(value);
  }
  return value;
});

console.log(meetup.date.getDate()); // 30 - теперь работает!
~~~

Кстати, это работает и для вложенных объектов:

~~~
const str = `{
  "meetups": [
    {"title":"Conference","date":"2017-11-30T12:00:00.000Z"},
    {"title":"Birthday","date":"2017-04-18T12:00:00.000Z"}
  ]
}`;

schedule = JSON.parse(str, function (key, value) {
  if (key == "date") {
    return new Date(value);
  }
  return value;
});

console.log(schedule.meetups[1].date.getDate()); // 18 - отлично!
~~~
