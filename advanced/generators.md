# ?Generators

_Генераторы_ – новый вид функций в современном JavaScript. Они отличаются от обычных тем, что могут приостанавливать своё выполнение, возвращать промежуточный результат и далее возобновлять его позже, в произвольный момент времени.

С помощью генераторов мы можем порционно выдавать какой-то результат. Для определения генераторов (а это особый вид функции) применяется символ звездочки. Для возвращения значения из генератора применяется оператор `yield`. Для получения значения из генератора применяется метод `next()`.

~~~
function* strGenerator() {
  yield 'h';
  yield 'i';
}

console.log(strGenerator().next()); // { value: 'h', done: false }
~~~
