# ?`Object.is`

JavaScript предоставляет три различных операции сравнения значений:

* `===` - строгое равенство (тройное равенство)
* `==` - нестрогое равенство (двойное равенство)
* `Object.is()`

Статический метод `Object.is()` определяет, являются ли два значения одним и тем же значением.

`Object.is()` похож на оператор ===. Единственная разница между `Object.is()` и `===` заключается в их обработке нулей со знаком и значений `NaN`. Оператор `===` рассматривает числовые значения `-0` и `+0` как равные, но обрабатывает `NaN` как не равные друг другу.

`console.log(Object.is(NaN, NaN)); // true`
`console.log(NaN === NaN); // false`

`console.log(Object.is(-0, 0)); // false`
`console.log(-0 === 0); // true`

### Полифил `Object.is`

~~~
Object.prototype.polyfillIs = function(x, y) {
  if (x === y) {
    return x !== 0 || 1 / x === 1 / y; // 0 === -0, but they are not identical
    }
  return x !== x && y !== y; // NaN !== NaN, but they are identical.
}

console.log(Object.polyfillIs(NaN, NaN)); // true
~~~

> Сейчас `Object.is` поддерживается всеми браузерами.
