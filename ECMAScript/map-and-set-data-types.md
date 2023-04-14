# ?Map & Set data types

### Map

`Map` – это коллекция ключ/значение, как и `Object`. Но основное отличие в том, что `Map` позволяет использовать ключи любого типа.

__Методы и свойства:__

* `new Map()` – создаёт коллекцию.
* `map.set(key, value)` – записывает по ключу `key` значение `value`.
* `map.get(key)` – возвращает значение по ключу или `undefined`, если ключ `key` отсутствует.
* `map.has(key)` – возвращает `true`, если ключ `key` присутствует в коллекции, иначе `false`.
* `map.delete(key)` – удаляет элемент (пару «ключ/значение») по ключу `key`.
* `map.clear()` – очищает коллекцию от всех элементов.
* `map.size` – возвращает текущее количество элементов.

Например:

~~~
const map = new Map();

map.set("1", "str");    // строка в качестве ключа
map.set(1, "num");      // цифра как ключ
map.set(true, "bool");  // булево значение как ключ

// помните, обычный объект Object приводит ключи к строкам?
// Map сохраняет тип ключей, так что в этом случае сохранится 2 разных значения:
console.log(map.get(1)); // "num"
console.log(map.get("1")); // "str"

console.log(map.size); // 3
~~~

Как мы видим, в отличие от объектов, ключи не были приведены к строкам. Можно использовать любые типы данных для ключей.

> Хотя `map[key]` также работает, например, мы можем установить `map[key] = 2`, в этом случае `map` рассматривался бы как обычный JavaScript объект, таким образом это ведёт ко всем соответствующим ограничениям (только строки/символьные ключи и так далее). Поэтому нам следует использовать методы `map`: `set`, `get` и так далее.

Использование объектов в качестве ключей – одна из наиболее заметных и важных функций `Map`. Это то что невозможно для `Object`. Строка в качестве ключа в `Object` – это нормально, но мы не можем использовать другой `Object` в качестве ключа в `Object`.

> Чтобы сравнивать ключи, объект `Map` использует алгоритм `SameValueZero`. Это почти такое же сравнение, что и `===`, с той лишь разницей, что `NaN` считается равным `NaN`. Так что `NaN` также может использоваться в качестве ключа.

__Перебор `Map`__

~~~
const recipeMap = new Map([
  ["cucumber", 500],
  ["tomato", 350],
  ["onion", 50],
]);

console.log(recipeMap.keys()); // MapIterator { cucumber, tomato, onion }
console.log(recipeMap.values()); // MapIterator { 500, 350, 50 }
console.log(recipeMap.entries()); // MapIterator { cucumber => 500, tomato => 350, onion => 50 }

for (const entry of recipeMap) {
  console.log(entry);
}
// ['cucumber', 500]
// ['tomato', 350]
// ['onion', 50]


// Map имеет встроенный метод forEach
recipeMap.forEach((value, key, map) => console.log(key, value));
// cucumber 500
// tomato 350
// onion 50
~~~

__`Map` из `Object` и наоборот__

`Object.entries` - это метод, который помогает привести объект к списку пар ключ-значение.

~~~
const obj = {
  name: "John",
  age: 30,
};

const map = new Map(Object.entries(obj));

console.log(map.get("name")); // John
~~~

Здесь `Object.entries` возвращает массив пар ключ-значение: `[ ["name","John"], ["age", 30] ]`. Это именно то, что нужно для создания `Map`.

`Object.fromEntries` - это метод, который помогает привести список пар ключ-значение к объекту.

Мы можем использовать `Object.fromEntries`, чтобы получить обычный объект из `Map`.

~~~
const map = new Map();
map.set('banana', 1);
map.set('orange', 2);
map.set('meat', 4);

const obj = Object.fromEntries(map);

console.log(obj); // { banana: 1, orange: 2, meat: 4 }
~~~

### Set

Объект `Set` – это особый вид коллекции: «множество» значений (без ключей), где каждое значение может появляться только один раз.

__Его основные методы это:__

* `new Set(iterable)` – создаёт `Set`, и если в качестве аргумента был предоставлен итерируемый объект (обычно это массив), то копирует его значения в новый `Set`.
* `set.add(value)` – добавляет значение (если оно уже есть, то ничего не делает), возвращает тот же объект `set`.
* `set.delete(value)` – удаляет значение, возвращает `true`, если `value` было в множестве на момент вызова, иначе `false`.
* `set.has(value)` – возвращает `true`, если значение присутствует в множестве, иначе `false`.
* `set.clear()` – удаляет все имеющиеся значения.
* `set.size` – возвращает количество элементов в множестве.

Основная «изюминка» – это то, что при повторных вызовах `set.add()` с одним и тем же значением ничего не происходит, за счёт этого как раз и получается, что каждое значение появляется один раз.

~~~
const set = new Set();

const john = { name: "John" };
const pete = { name: "Pete" };
const mary = { name: "Mary" };

// считаем гостей, некоторые приходят несколько раз
set.add(john);
set.add(pete);
set.add(mary);
set.add(john);
set.add(mary);

// set хранит только 3 уникальных значения
console.log(set.size); // 3

for (const user of set) {
  console.log(user.name); // John Pete Mary
}
~~~

__Перебор `Set`__

Мы можем перебрать содержимое объекта `set` как с помощью метода `for..of`, так и используя `forEach`:

~~~
const set = new Set(["orange", "apple", "banana"]);

for (const value of set) {
  console.log(value); // orange apple banana
}

// то же самое с forEach:
set.forEach((value, valueAgain, set) => {
  console.log(value, valueAgain);
});
// orange orange
// apple apple
// banana banana
~~~

Заметим забавную вещь. Функция в `forEach` у `Set` имеет 3 аргумента: значение `value`, потом снова то же самое значение `valueAgain`, и только потом целевой объект. Это действительно так, значение появляется в списке аргументов дважды.

Это сделано для совместимости с объектом `Map`, в котором колбэк `forEach` имеет 3 аргумента. Выглядит немного странно, но в некоторых случаях может помочь легко заменить `Map` на `Set` и наоборот:

~~~
const nameSet = new Set(["John", "Pete", "Mary"]);

console.log(nameSet); // Set(3) {size: 3, John, Pete, Mary}
const map = new Map(nameSet.entries());
console.log(map); // Map(3) {size: 3, John => John, Pete => Pete, Mary => Mary}
~~~

`Set` имеет те же встроенные методы, что и `Map`:

* `set.keys() `– возвращает перебираемый объект для значений,
* `set.values()` – то же самое, что и `set.keys()`, присутствует для обратной совместимости с `Map`,
* `set.entries()` – возвращает перебираемый объект для пар вида `[значение, значение]`, присутствует для обратной совместимости с `Map`.

~~~
const nameSet = new Set(["John", "Pete", "Mary"]);

console.log(nameSet.keys()); // SetIterator {John, Pete, Mary}
console.log(nameSet.values()); // SetIterator {John, Pete, Mary}
console.log(nameSet.entries()); // SetIterator {John => John, Pete => Pete, Mary => Mary}
~~~

Перебор `Map` и `Set` всегда осуществляется в порядке добавления элементов, так что нельзя сказать, что это – неупорядоченные коллекции, но поменять порядок элементов или получить элемент напрямую по его номеру нельзя.
