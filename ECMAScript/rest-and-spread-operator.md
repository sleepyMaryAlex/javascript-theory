# ?Rest vs spread operator

JavaScript использует три точки `...` как для `rest`, так и для `spread` операторов. Но эти два оператора не одно и то же.

__Rest vs spread operator. What’s the difference?__
Отличия `spread` или `rest` в том, как и где они используются. `Spread` преобразует массив (объект, строку или другой доступный для итерации элемент) в последовательность значений, а `rest` преобразует последовательность значений в массив.

_Spread:_
~~~
const arr = [1, 2, 3]; 
console.log(...arr); // 1 2 3 

const hi = 'hi!'; 
console.log(...hi); // h i !

function foo(a, b) {
  console.log(a, b); // 1 2
}

foo(...[1, 2]);
~~~

_Rest:_
~~~
function foo(...args) {
  console.log(args); // [ 1, 2, 3, 4 ]
}

foo(1, 2, 3, 4);
~~~

__Arguments vs rest parameters. What’s the difference?__

Существует три основных отличия `rest параметров` от объекта `arguments`:

* Остаточные параметры включают только те, которым не задано отдельное имя, в то время как объект arguments содержит все аргументы, передаваемые в функцию;
* Объект arguments не является массивом, в то время как остаточные параметры являются экземпляром Array;
* Объект arguments имеет дополнительную функциональность, специфичную только для него (например, свойство callee).

Свойства `arguments`:
* arguments.callee: cсылка на функцию, которая выполняется в текущий момент.
* arguments.callee.caller: cсылка на функцию, которая вызвала функцию, выполняющуюся в текущий момент.
* arguments.length: количество переданных в функцию аргументов.

__Spread operator for Array__

Merge Array:
`[...array1, ...array2];`

Clone Array:
`[...array];`

String → Array:
`[...'string'];`

Set → Array:
`[...new Set([1,2,3])];`

Node List → Array:
`[...nodeList];`

Arguments → Array:
`[...arguments];`

Math.min and Math.max from Array:
`Math.min(...array);`

Мы даже можем комбинировать оператор расширения с обычными значениями:
`Math.max(1, ...arr1, 2, ...arr2, 25);`

`Array.from` тоже преобразует итерируемый объект в массив. Но между `Array.from` и `spread` есть разница:
* Array.from работает как с псевдомассивами, так и с итерируемыми объектами
* Spread работает только с итерируемыми объектами

~~~
const arrayLikeObject = { 0: 'a', 1: 'b', length: 2 };
console.log(Array.from(arrayLikeObject)); // ['a', 'b']
console.log([...arrayLikeObject]); // arrayLikeObject is not iterable
~~~

__Spread operator for Array concatenation. Example__

~~~
const arr1 = [3, 5, 1];
const arr2 = [8, 9, 15];
const merged = [0, ...arr1, 2, ...arr2];
console.log(merged); // [ 0, 3, 5, 1, 2, 8, 9, 15 ]
~~~

__Spread operator for Object concatenation__
Можно объединить объекты при помощи spread:

~~~
const obj1 = { foo: 'bar', x: 42 };
const obj2 = { foo: 'baz', y: 13 };
console.log({ ...obj1, ...obj2 });
// { foo: "baz", x: 42, y: 13 }
~~~

Мы может добавить новые свойства объекту таким образом:

~~~
const user = { country: 'Poland' }
const name = 'Mary';
const age = 28;
console.log({ ...user, name, age }); // { country: 'Poland', name: 'Mary', age: 28 }
~~~

Если название переменной совпадает с названием свойства, можно сократить:

`console.log({ ...user, name, age });`

Вместо:

`console.log({ ...user, name: name, age: age });`
