# ?Arrays and tuples

### Массивы

Массивы определяются с помощью выражения `[]` и также являются строго типизированными. То есть если изначально массив содержит строки, то в будущем он сможет работать только со строками.

~~~
const list: number[] = [10, 20, 30];
const colors: string[] = ["red", "green", "blue"];
~~~

Альтернативный способ определения массивов представляет применение типа `Array<>`, где в фигурных скобках указывается тип элементов массива:

~~~
const names: Array<string> = ["Tom", "Bob", "Alice"];
~~~

Фактически такие формы массивов, как `number[]` или `string[]` являются сокращением соответственно типов `Array<number>` или `Array<string>`.

Можно определить массив с несколькими типами:

~~~
const tom: (string | number)[] = ["Tom", 36];
~~~

Или:

~~~
const tom: Array<string | number> = ["Tom", 36];
~~~

### `ReadonlyArray`

Массивы позволяют изменять значения своих элементов:

~~~
const people = ["Tom", "Bob", "Sam"];
people[1] = "Kate";
console.log(people[1]); // Kate
~~~

Но TypeScript также позволяет определять массивы, элементы которых нельзя изменять. Для этого применяется тип `ReadonlyArray<>`, для которого в угловых скобках указывается тип элементов массива.

В отличие от типа `Array` для типа `ReadonlyArray` мы не можем принимать конструктор, как в следующем случае:

~~~
const people: ReadonlyArray<string> = ["Tom", "Bob", "Sam"];
~~~

Для определения подобных массивов также можно использовать сокращение типа - `readonly Тип_данных[]`:

~~~
const people: readonly string[] = ["Tom", "Bob", "Sam"];
~~~

Массив `ReadonlyArray` поддерживаtт большинство тех же операций, что и обычные массивы, за тем исключением операций, которые изменяют массив и его элементы. Так, мы не можем менять отдельные значения:

~~~
const people: ReadonlyArray<string> = ["Tom", "Bob", "Sam"];
people[1] = "Kate"; // Index signature in type 'readonly string[]' only permits reading.
~~~

Также мы не можем добавлять новые или удалять уже имеющиеся элементы.

Более того при компиляции компилятор сообщит нам, что для типа `ReadonlyArray` не определены методы `push()` и `pop()`. Это относится ко всем операциям, которые изменяют массив.

Все остальные операции, которые предусматривают чтение массива, мы по прежнему можем использовать.

### Tuples. Кортежи

Кортежи (Tuples) также, как и массивы, представляют набор элементов, для которых уже заранее известен тип. В отличие от массивов кортежи могут хранить значения разных типов. Для определения кортежа применяется синтаксис массива:

~~~
const user: [string, number] = ["Tom", 36];
~~~

В данном случае кортеж `user` представляет тип `[string, number]`, то есть такой кортеж, который состоит из двух элементов, при чем первый элемент представляет тип `string`, а второй элемент - тип `number`.

Для присвоения значения применяется массив, причем передаваемые кортежу данные должны соответствовать элементам по типу.

Кортежи как параметры функции и как результат функции:

~~~
function printUser(user: [string, number]): [number, string] {
  return [user[1], user[0]];
}

const tom: [string, number] = ["Tom", 36];
console.log(printUser(tom)); // [36, 'Tom']
~~~

#### Необязательные элементы кортежей

Кортежи могут иметь необязательные элементы, для которых можно не предоставлять значение. Чтобы указать, что элемент является необязательным, после типа элемента ставится вопросительный знак `?`:

~~~
const tom: [string, number, boolean?] = ["Tom", 36];
~~~

В данном случае последний элемент, который представляет тип `boolean`, необязательный. Причем необязательные элементы должны идти в самом конце - после обязательных элементов, иначе ошибка.

#### Заполнение кортежа

С помощью оператора `...` внутри определения типа кортежа можно определить набор элементов, количество которых неопределено. Например:

~~~
const math: [string, ...number[]] = ["Math", 5, 4, 5, 4, 4];
~~~

В данном случае кортеж представляет тип `[string, ...number[]]`. То есть первый элемент кортежа должен представлять тип `string`. А остальные элементы кортежа должны представлять тип `number`, причем таких элементов может быть неопределенное количество.

При этом неопределенное количество элементов можно определять как в конце, так и в середине и в начале кортежа:

~~~
const math: [string, ...number[]] = ["Math", 5, 4, 5, 4, 4];
const physics: [...number[], string] = [5, 5, 5, "Physics"];
const chemistry: [string, ...number[], boolean] = ["Chemistry", 3, 3, 4, 5, false];
~~~

В случае с кортежем `[...number[], string]` он должен оканчиваться на элемент типа `string`, перед которым может быть неопределенное количество элементов типа `number`.

А в кортеже типа `[string, ...number[], boolean]` первый элемент должен представлять тип `string`, а последний - тип `boolean`. Между ними может быть неопределенное количество элементов типа `number`.

#### Помеченные элементы кортежа

При маркировке элемента кортежа все остальные элементы в кортеже также должны быть маркированы.

В таких случаях помеченные кортежи могут быть удобны:

~~~
type Name = [first: string, last: string] | [first: string, middle: string, last: string];

function createPerson(...name: Name) {
  // ...
}

createPerson("Tom", "Smith");
~~~

#### Кортеж для чтения

Стандартные кортежи позволяют изменять значения их элементов:

~~~
const tom: [string, number] = ["Tom", 36];
tom[1] = 37;
console.log(tom[1]); // 37
~~~

Однако TypeScript также позволяет создавать кортежи только для чтения, элементы которого нельзя изменить. Для этого перед типом кортежа указывается ключевое слово `readonly`:

~~~
const tom: readonly [string, number] = ["Tom", 36];
tom[1] = 37; // Cannot assign to '1' because it is a read-only property.
~~~

Кортеж для чтения в качестве параметра функции и в качестве результата функции:

~~~
function printUser(user: readonly [string, number]): readonly [string, number] {
  return [user[0].toUpperCase(), user[1]];
}

console.log(printUser(["Tom", 45])); // ['TOM', 45]
~~~
