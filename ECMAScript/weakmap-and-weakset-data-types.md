# ?WeakMap & WeakSet data types

### WeakMap

Движок JavaScript хранит значения в памяти до тех пор, пока они достижимы (то есть, эти значения могут быть использованы).

~~~
let john = { name: "John" };

// объект доступен, переменная john -- это ссылка на него

// перепишем ссылку
john = null;

// объект будет удалён из памяти
~~~

Обычно свойства объекта, элементы массива или другой структуры данных считаются достижимыми и сохраняются в памяти до тех пор, пока эта структура данных содержится в памяти.

Например, если мы поместим объект в массив, то до тех пор, пока массив существует, объект также будет существовать в памяти, несмотря на то, что других ссылок на него нет.

~~~
let john = { name: "John" };

const array = [john];

john = null;

console.log(array[0]); // { name: 'John' }
console.log(john); // null
~~~

Аналогично, если мы используем объект как ключ в `Map`, то до тех пор, пока существует `Map`, также будет существовать и этот объект. Он занимает место в памяти и не может быть удалён сборщиком мусора.

~~~
let john = { name: "John" };

const map = new Map();
map.set(john, "...");

john = null;

console.log(map.keys()); // MapIterator {{ name: 'John' }}
~~~

`WeakMap` – принципиально другая структура в этом аспекте. Она не предотвращает удаление объектов сборщиком мусора, когда эти объекты выступают в качестве ключей.

Первое отличие `WeakMap` от `Map` в том, что ключи в `WeakMap` должны быть объектами, а не примитивными значениями:

~~~
const weakMap = new WeakMap();
const obj = {};

weakMap.set(obj, "ok"); // работает (объект в качестве ключа)
weakMap.set("test", "Whoops"); //  TypeError: Invalid value used as weak map key
~~~

Теперь, если мы используем объект в качестве ключа и если больше нет ссылок на этот объект, то он будет удалён из памяти (и из объекта `WeakMap`) автоматически.

~~~
let john = { name: "John" };

const weakMap = new WeakMap();
weakMap.set(john, "...");

john = null; // перезаписываем ссылку на объект

// объект john будет удалён из памяти!
~~~

Сравните это поведение с поведением обычного `Map`, пример которого был приведён ранее. Теперь `john` существует только как ключ в `WeakMap` и может быть удалён оттуда автоматически.

`WeakMap` не поддерживает перебор и методы `keys()`, `values()`, `entries()`, так что нет способа взять все ключи или значения из неё.

__В `WeakMap` присутствуют только следующие методы__:

* `weakMap.get(key)`
* `weakMap.set(key, value)`
* `weakMap.delete(key)`
* `weakMap.has(key)`

К чему такие ограничения? Из-за особенностей технической реализации. Если объект станет недостижим (как объект `john` в примере выше), то он будет автоматически удалён сборщиком мусора. Но нет информации, _в какой момент произойдёт эта очистка_.

Решение о том, когда делать сборку мусора, принимает движок JavaScript. Он может посчитать необходимым как удалить объект прямо сейчас, так и отложить эту операцию, чтобы удалить большее количество объектов за раз позже. Так что технически количество элементов в коллекции `WeakMap` неизвестно. Движок может произвести очистку сразу или потом, или сделать это частично. По этой причине методы для доступа ко всем сразу ключам/значениям недоступны.

__Но для чего же нам нужна такая структура данных?__

В основном, `WeakMap` используется в качестве дополнительного хранилища данных.

Если мы работаем с объектом, который «принадлежит» другому коду, может быть даже сторонней библиотеке, и хотим сохранить у себя какие-то данные для него, которые должны существовать лишь пока существует этот объект, то `WeakMap` – как раз то, что нужно.

Мы кладём эти данные в `WeakMap`, используя объект как ключ, и когда сборщик мусора удалит объекты из памяти, ассоциированные с ними данные тоже автоматически исчезнут.

~~~
const visitsCountMap = new WeakMap(); // map: пользователь => число визитов

// увеличиваем счётчик
function countUser(user) {
  const count = visitsCountMap.get(user) || 0;
  visitsCountMap.set(user, count + 1);
}

let john = { name: "John" };

countUser(john); // ведём подсчёт посещений

console.log(visitsCountMap); // WeakMap {{name: 'John'} => 1}

// пользователь покинул нас
john = null;
~~~

После того, как объект `john` стал недостижим другими способами, кроме как через `WeakMap`, он удаляется из памяти вместе с информацией по такому ключу из `WeakMap`.

### WeakSet

Коллекция `WeakSet` ведёт себя похоже:

* Она аналогична `Set`, но мы можем добавлять в `WeakSet` только объекты (не примитивные значения).
* Объект присутствует в множестве только до тех пор, пока доступен где-то ещё.
* Как и `Set`, она поддерживает `add()`, `has()` и `delete()`, но не `size`, `keys()` и не является перебираемой.

Будучи «слабой» версией оригинальной структуры данных, она тоже служит в качестве дополнительного хранилища. Но не для произвольных данных, а скорее для значений типа «да/нет». Присутствие во множестве WeakSet может что-то сказать нам об объекте.

Например, мы можем добавлять пользователей в `WeakSet` для учёта тех, кто посещал наш сайт:

~~~
const visitedSet = new WeakSet();

let john = { name: "John" };
let pete = { name: "Pete" };
let mary = { name: "Mary" };

visitedSet.add(john); // John заходил к нам
visitedSet.add(pete); // потом Pete
visitedSet.add(john); // John снова

// visitedSet сейчас содержит двух пользователей
console.log(visitedSet); // WeakSet {{name: 'John'}, {name: 'Pete'}}

// проверим, заходил ли John?
console.log(visitedSet.has(john)); // true

// проверим, заходила ли Mary?
console.log(visitedSet.has(mary)); // false

john = null;

// структура данных visitedSet будет очищена автоматически (объект john будет удалён из visitedSet)
~~~

Наиболее значительным ограничением `WeakMap` и `WeakSet` является то, что их нельзя перебрать или взять всё содержимое. Это может доставлять неудобства, но не мешает `WeakMap`/`WeakSet` выполнять их главную задачу – быть дополнительным хранилищем данных для объектов, управляемых из каких-то других мест в коде.