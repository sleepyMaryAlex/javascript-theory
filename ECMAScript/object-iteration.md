# ?How to iterate object

Различные методы, которые можно использовать для перебора объектов в JavaScript:

1. Использование цикла `for...in`
2. Метод `Object.keys`
3. Метод `Object.values`
4. Метод `Object.entries`
5. Метод `Object.getOwnPropertyNames`

Допустим, у нас есть объект `obj`:

~~~
const triangle = { a: 1, b: 2, c: 3 };

function ColoredTriangle() {
  this.color = 'red';
}

ColoredTriangle.prototype = triangle;
const obj = new ColoredTriangle();

Object.defineProperty(obj, 'd', {
  value: 4,
  enumerable: false,
  configurable: true,
  writable: true,
});
~~~

* `for..in` - проходит через перечисляемые свойства объекта, в том числе в цепочке `prototype`. Цикл `for..in` проходит по свойствам в произвольном порядке:

~~~
for (const prop in obj) {
  console.log(prop); // color a b c
}
~~~

Чтобы не идти по цепочке прототипов, можно явно проверить, принадлежит ли свойство объекту, используя метод `hasOwnProperty()`:

~~~
for (const prop in obj) {
  if (obj.hasOwnProperty(prop)) {
    console.log(prop); // color
  }
}
~~~

* До ES6 единственным способом пройтись по объекту было использование цикла `for...in`. Метод `Object.keys()` был введен в ES6, чтобы упростить цикл по объектам. `Object.keys` - возвращает массив из собственных перечисляемых свойств переданного объекта, в том же порядке, в котором они бы обходились циклом `for..in`, но не идет по цепочке прототипов.

~~~
const keys = Object.keys(obj);
keys.forEach(key => {
  console.log(key); // color
});
~~~

Аналогично `Object.entries()`. `Object.entries()`, другой метод ES8, который можно использовать для обхода массива. `Object.entries()` выводит массив массивов, каждый из которых содержит два элемента. Первый элемент — это свойство, а второй — значение:

~~~
const entries = Object.entries(obj);
entries.forEach(entry => {
  console.log(entry[0]); // color
});
~~~

Метод `Object.values()` был введен в ES8. Он также идет по собственным перечисляемым свойствам переданного объекта и не идет по цепочке прототипов. Он возвращает значения всех свойств объекта в виде массива. Затем вы можете выполнить цикл по массиву значений, используя любой из методов цикла массива.

~~~
const values = Object.values(obj);
values.forEach(value => {
  console.log(value); // red
});
~~~

* `Object.getOwnPropertyNames` - возвращает массив строк, соответствующих перечисляемым и неперечисляемым свойствам, в том же порядке, в котором они бы обходились циклом `for..in`, не идет по цепочке прототипов.

~~~
const getOwnPropertyNames = Object.getOwnPropertyNames(obj);
getOwnPropertyNames.forEach(key => {
  console.log(key); // color d
});
~~~
