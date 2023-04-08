# ?Function declaration / function expression / arrow function

Вы можете определять функции JavaScript разными способами.

Первый способ — __function declaration__:
~~~
function greet(who) {
  return `Hello, ${who}!`;
}
~~~

Второй способ — __function expression__:
~~~
const greet = function(who) {
  return `Hello, ${who}`;
}
~~~

Третий способ — __arrow function__:

~~~
const greet = (who) => {
  return `Hello, ${who}!`;
}
~~~

### Function declaration & function expression vs. arrow function

#### this

__Function declaration & function expression__

Внутри этих функций JavaScript значение this (также известное как контекст выполнения) является динамическим.

Динамический контекст означает, что значение this зависит от того, как вызывается функция. В JavaScript есть 4 способа вызова обычной функции:

1. Во время простого вызова значение this равно глобальному объекту (или undefined, если функция работает в строгом режиме):

~~~
function myFunction() {
  console.log(this);
}

myFunction(); // global object (window)
~~~

2. Во время вызова метода значением this является объект, владеющий методом:

~~~
const myObject = {
  method() {
    console.log(this);
  }
};

myObject.method(); // myObject
~~~

3. Во время непрямого вызова с использованием ```myFunc.call(thisVal, arg1, ..., argN)``` или ```myFunc.apply(thisVal, [arg1, ..., argN])``` значение this равно первому аргументу:

~~~
function myFunction() {
  console.log(this);
}

const myContext = { value: 'A' };
myFunction.call(myContext);  // { value: 'A' }
myFunction.apply(myContext); // { value: 'A' }
~~~

4. Во время вызова конструктора с помощью ключевого слова new this равняется вновь созданному экземпляру:

~~~
function MyFunction() {
  console.log(this);
}

new MyFunction(); // MyFunction {}
~~~

__Arrow function__

Независимо от того, как и где выполняется, значение this внутри стрелочной функции всегда равно this значению из внешней функции.

~~~
const myObject = {
  myMethod(items) {
    console.log(this); // myObject
    const callback = () => {
      console.log(this); // myObject
    };
    items.forEach(callback);
  }
};

myObject.myMethod([1, 2, 3]);
~~~

В отличие от function declaration и function expression, непрямой вызов стрелочной функции с использованием myArrowFunc.call(thisVal) или myArrowFunc.apply(thisVal) не изменяет значение this: значение контекста всегда разрешается лексически.

#### Конструкторы

__Function declaration & function expression__

Эти функции могут легко создавать объекты.

~~~
function Car(color) {
  this.color = color;
}

const redCar = new Car('red');
console.log(redCar instanceof Car); // => true
~~~

__Arrow function__

Следствием отсутствия this является то, что стрелочную функцию нельзя использовать в качестве конструктора.

Если вы попытаетесь вызвать стрелочную функцию с префиксом ключевого слова new, JavaScript выдаст ошибку.

~~~
const Car = (color) => {
  this.color = color;
};

const redCar = new Car('red'); // TypeError: Car is not a constructor
~~~

#### Arguments

__Function declaration & function expression__

Внутри тела function declaration и function expression _arguments_ — это специальный массивоподобный объект, содержащий список аргументов, с которыми была вызвана функция.
~~~
function myFunction() {
  console.log(arguments);
}

myFunction('a', 'b'); // { '0': 'a', '1': 'b' }
~~~

__Arrow function__

Опять же (так же, как и с этим значением), объект arguments разрешается лексически: стрелочная функция получает доступ к аргументам из внешней функции.

~~~
function myRegularFunction() {
  const myArrowFunction = () => {
    console.log(arguments);
  }
  myArrowFunction('c', 'd');
}
myRegularFunction('a', 'b'); // { 0: 'a', 1: 'b' }
~~~

> Если вы хотите получить доступ к прямым аргументам стрелочной функции, вы можете использовать rest parameters.

#### Неявный возврат

__Function declaration & function expression__

Если return отсутствует или после оператора return нет выражения, function declaration и function expression неявно возвращает undefined:
~~~
function myEmptyFunction() {
  42;
}
function myEmptyFunction2() {
  42;
  return;
}

console.log(myEmptyFunction());  // => undefined
console.log(myEmptyFunction2()); // => undefined
~~~

__Arrow function__

Если стрелочная функция содержит одно выражение и вы опускаете фигурные скобки функции, выражение возвращается неявно.
~~~
const increment = (num) => num + 1;
console.log(increment(41)); // => 42
~~~

#### Методы

__Function declaration & function expression__

В следующем классе Hero метод logName() определяется с помощью function declaration:
~~~
class Hero {
  constructor(heroName) {
    this.heroName = heroName;
  }
  logName() {
    console.log(this.heroName);
  }
}

const batman = new Hero('Batman');
batman.logName(); // Batman
~~~

Обычно обычные функции в качестве методов — это то, что нужно.
Иногда вам нужно указать метод в качестве обратного вызова, например, для setTimeout() или прослушивателя событий. В таких случаях вы можете столкнуться с трудностями при доступе к этому значению.
~~~
class Hero {
  constructor(heroName) {
    this.heroName = heroName;
  }
  logName() {
    console.log(this.heroName); // undefined
  }
}

const batman = new Hero('Batman');
setTimeout(batman.logName, 1000);
~~~

Привяжем к нужному контексту вручную:
```setTimeout(batman.logName.bind(batman), 1000);```

Но есть лучший способ: стрелка работает как поле класса.

__Arrow function__

Метод, определенный с помощью стрелки, лексически связывает this с экземпляром класса.
~~~
class Hero {
  constructor(heroName) {
    this.heroName = heroName;
  }
  logName = () => {
    console.log(this.heroName);
  }
}

const batman = new Hero('Batman');
setTimeout(batman.logName, 1000); // Batman
~~~

### Function declaration vs. function expression

_Function declaration_ — это когда вы создаете функцию и даете ей имя. Вы объявляете имя функции, когда пишете ключевое слово function, за которым следует имя функции.

_Function expression_ — это когда вы создаете функцию и назначаете ее переменной.

Есть несколько ключевых различий между function declaration и function expression:

* Function declaration поднимаются, а function expression — нет. Это означает, что вы можете вызвать function declaration до того, как оно будет определено, но вы не можете сделать это с function expression.
* Function Declaration создаются интерпретатором до выполнения кода.
* Function expression могут использоваться в качестве аргумента другой функции, но function declaration — нет.
* Function expression могут быть анонимными, а function declaration — нет.
* Function expression можно использовать как IIFE.
~~~
const greeting = "Hello world"; 
(function () {
    console.log(greeting);
})();
~~~

В некоторых случаях лучше использовать function expression:
~~~
'use strict';
const age = 20;

if (age >= 18) {
  function sayHi() {
    console.log( 'Прошу вас!' );
  }
} else {
  function sayHi() {
    console.log( 'До 18 нельзя' );
  }
}

sayHi(); // ReferenceError: sayHi is not defined
~~~

Function Declaration при use strict видны только внутри блока, в котором объявлены.

А что, если использовать Function Expression?

~~~
'use strict';

const age = 20;

let sayHi;

if (age >= 18) {
  sayHi = function() {
    console.log( 'Прошу Вас!' );
  }
} else {
  sayHi = function() {
    console.log( 'До 18 нельзя' );
  }
}

sayHi(); // Прошу Вас!
~~~
