# ?How to iterate object keys

Допустим, у нас есть объект obj:

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

* for..in - проходит через перечисляемые свойства объекта, в том числе в цепочке prototype. Цикл for..in проходит по свойствам в произвольном порядке.

~~~
for (const prop in obj) {
  console.log(prop); // color a b c
}
~~~

* Object.keys - возвращает массив из собственных перечисляемых свойств переданного объекта, в том же порядке, в котором они бы обходились циклом for..in, но не идет по цепочке прототипов.

~~~
const keys = Object.keys(obj);
keys.forEach(key => {
  console.log(key); // color
});
~~~

Аналогично Object.entries:

~~~
const keys = Object.entries(obj);
keys.forEach(key => {
  console.log(key[0]); // color
});
~~~

* Object.getOwnPropertyNames - возвращает массив строк, соответствующих перечисляемым и неперечисляемым свойствам, в том же порядке, в котором они бы обходились циклом for..in, не идет по цепочке прототипов.

~~~
const getOwnPropertyNames = Object.getOwnPropertyNames(obj);
getOwnPropertyNames.forEach(key => {
  console.log(key); // color d
});
~~~
