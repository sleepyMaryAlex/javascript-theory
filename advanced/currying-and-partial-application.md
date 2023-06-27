# ?Currying vs partial application

### Currying

_Каррирование_ – продвинутая техника для работы с функциями. Она используется не только в JavaScript, но и в других языках. _Каррирование_ –  это преобразование функции с несколькими аргументами в последовательность вложенных функций с одним аргументом ( `f(a, b, c)` => `f(a)(b)(c)`).

_Каррированные функции_ — это функции более высокого порядка, которые позволяют нам создавать специализированные версии исходных функций. Каррирование работает благодаря замыканиям, которые сохраняют объемлющие области действия после того, как они возвращаются. Каррированная функция может принимать несколько аргументов, но по одному за раз. Способность принимать по одному аргументу за раз, называется _унарностью_. Унарность, это обязательное свойство каррированных функций.

У нас есть функция `sum`:
~~~
function sum(a, b) {
  return a + b;
}
~~~

Вот как выглядит её каррированный вариант:

~~~
function sum(a) {
  return function(b) {
    return a + b;
  };
}

console.log(sum(1)(2)); // 3
~~~

Еще один пример:

~~~
function curry(fn) { // curry(fn) - функция высшего порядка, выполняющая каррирование
  return function(a) { // curry(fn) возвращает функцию, принимающую один аргумент
    return function(b) { // curry(fn) возвращает функцию, принимающую один аргумент
      return fn(a, b); // curry(fn) возвращает результат вызова исходной функции с двумя переданным в анонимные функции параметрами!
    }
  };
}

function sum(a, b) {
  return a + b;
}

// Создали новую функцию на основе функции sum, принимающей 2 значения
const curriedSum = curry(sum); // curriedSum - каррированная функция
console.log(curriedSum(1)(2)); // 3
~~~

В следующем примере стрелочные функции используются для создания как обычных, так и каррированных функций:

~~~
const curry = (fn) => (a) => (b) => fn(a, b);

const sum = (a, b) => a + b;

const curriedSum = curry(sum);
console.log(curriedSum(1)(2)); // 3
~~~

Вроде просто, но если аргументов будет больше 2, то придётся добавлять ещё одну обёртку. Поэтому лучше посчитать количество аргументов и автоматизировать создание обёрток:

~~~
function curry(func) {
  return function curried(...args) {
    if (args.length >= func.length) {
      return func(...args);
    }

    return function continueCurrying(...args2) {
      return curried(...args.concat(args2));
    };
  };
}

function multiply(a, b, c, d) {
  return a * b * c * d;
}

const curriedMultiply = curry(multiply);

console.log(curriedMultiply(2)(3)(4)(5)); // 120
~~~

### Partial application

_Частичное применение_ - создание некой функции на базе указанной, где некоторые аргументы заменены конкретными значениями.

~~~
const partial = (fn, ...argsToApply) => {
  return (...restArgsToApply) => {
    return fn(...argsToApply, ...restArgsToApply);
  };
};

const getApiURL = (apiHostname, resourceName, resourceId) => {
  return `https://${apiHostname}/api/${resourceName}/${resourceId}`;
};

console.log(partial(getApiURL, "localhost:3000")("users", "userId"));
// https://localhost:3000/api/users/userId
~~~

Интересно, что есть метод, который можно использовать точно так же partial, и он встроен в сам язык! Возможно, вы даже использовали его. Это называется `bind`. Он связывает функцию не только с ее контекстом `this` (более популярный вариант использования), но и с ее первыми аргументами. Давайте взглянем:

~~~
const getApiURL = (apiHostname, resourceName, resourceId) => {
  return `https://${apiHostname}/api/${resourceName}/${resourceId}`;
};

const getResourceURL = getApiURL.bind(null, "localhost:3000");
const getUserURL = getResourceURL.bind(null, "users", "userId");
console.log(getUserURL());
// https://localhost:3000/api/users/userId
~~~

### Currying vs partial application

«_Каррирование_» и «_частичное применение_» - это две совершенно разные функции.

_Каррирование_: позволяет вызывать функцию, разделяя ее на несколько вызовов, предоставляя один аргумент для каждого вызова. Имеет тенденцию превращать функцию `N` аргументов в `N` функций одного аргумента

_Частичное применение_: позволяет вызывать функцию, разделяя ее на несколько вызовов, предоставляя несколько аргументов для каждого вызова. Частичное применение лучше работает с вариативными функциями с неопределенным количеством аргументов.

~~~
const curriedSum = math => eng => geo => math + eng + geo;
const partialSum = math => (eng, geo) => math + eng + geo;
~~~
