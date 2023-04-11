# ?this

В JavaScript `this` — это переменная, которая хранит в себе контекст (объект) выполняемого кода.

_Контекст_ — это значение ключевого слова `this`, это объект "владеющий" исполняемым кодом.

Механизм `this` подобен динамической области видимости, так как динамическая область и `this` определяются во время выполнения. Динамическая область заботится о том, откуда была вызвана функция.

`this` имеет различные значения в зависимости от того, где используется:

* Сама по себе `this` относится к глобальному объекту (`window`).
* В методе `this` относится к родительскому объекту.
* В функции `this` относится к глобальному объекту.
* В функции в 'strict mode' `this` = undefined.
* В стрелочной функции `this` относится к контексту, где функция была создана.
* В событии `this` ссылается на элемент запустивший событие.

Зачем `this`? `this` позволяет использовать одну и ту же функцию/метод для разных объектов.
Вне функции `this` ссылается на глобальный контекст независимо от режима. Но если мы в модуле, `this` равен undefined.

`console.log(this); // Window`

~~~
"use strict";
console.log(this); // Window
~~~

В функциях не в строгом режиме, если значение `this` не установлено в контексте выполнения, оно остаётся `Window`, в строгом - `undefined`.

~~~
function foo() {
  console.log(this); // Window
}

foo();
~~~

~~~
"use strict";
function foo() {
  console.log(this); // undefined
}

foo();
~~~

Еще один пример:

~~~
var bar = 'bar1';

function foo() {
  var bar = 'bar2';
  console.log(this.bar); // bar1 (в строгом режиме будет Cannot read properties of undefined (reading 'bar'))
}

foo();
~~~

Здесь используется `var`, так как глобальные переменные, задекларированные при помощи ключевого слова `var`, принадлежат глобальному объекту `window`, а при помощи `let` и `const` не принадлежат.

Для того, чтобы при вызове функции установить `this` в определённое значение, можно использовать `call()`, `apply()` или `bind()`:

~~~
const obj = { a: 'Custom' };
var a = 'Global'; // именно var

function whatsThis() {
  return this.a;
}

console.log(whatsThis());          // 'Global'
console.log(whatsThis.call(obj));  // 'Custom'
console.log(whatsThis.apply(obj)); // 'Custom'
const foo = whatsThis.bind(obj);
console.log(foo()); // 'Custom'
~~~

### this в методах

В теле каждого метода мы можем использовать `this`, чтобы обратится к родительскому объекту, в котором находится метод. Упрощенно можно сказать — `this` — это родительский объект, который вызвал метод (функцию). Что находится с левой стороны от точки, то и будет в `this` вызываемого метода.

~~~
const object = {
  method() {
    console.log(this);
  },
};

object.method(); // object
~~~

### this в функциях-конструкторах

Функции-конструкторы технически являются обычными функциями. Но есть два соглашения:

1. Имя функции-конструктора должно начинаться с большой буквы.
2. Функция-конструктор должна выполняться только с помощью оператора "`new`".

~~~
function User(name) {
  // this = {} - новый пустой объект, который вызвал эту функцию благодаря new
  this.name = name;
  // return this;
}

console.log(new User("Tom")); // User { name: 'Tom' }
~~~

Когда функция вызывается как `new User(...)`, происходит следующее:

1. Создаётся новый пустой объект, и он присваивается `this`.
2. Выполняется тело функции. Обычно оно модифицирует `this`, добавляя туда новые свойства.
3. Возвращается значение `this`.

Когда вызывается конструктор, `this` указывает на вновь созданный объект, а не на объект который вызвал функцию.
Основной целью конструкторов является реализовать код для многократного создания однотипных объектов.

Используя специальное свойство `new target` внутри функции, мы можем проверить, вызвана ли функция при помощи оператора `new` или без него. В случае обычного вызова функции `new target` будет `undefined`. Если же она была вызвана при помощи `new`, `new target` будет равен самой функции.

~~~
if (!new.target) { // в случае, если вы вызвали меня без оператора new
  return new User(name); // ...я добавлю new за вас
}
~~~

Обычно конструкторы не имеют оператора `return`. Их задача – записать все необходимое в `this`, и это автоматически становится результатом.

Но если `return` всё же есть, то применяются простые правила:

* При вызове return с объектом, вместо `this` вернётся этот объект.

~~~
function User(name) {
  this.name = name;
  return { age: 28 };
}

console.log(new User('Tom')) // { age: 28 }
~~~

* При вызове `return` с примитивным значением, оно проигнорируется.

~~~
function User(name) {
  this.name = name;
  return 'hello';
}

console.log(new User('Tom'))  // User { name: 'Tom' }
~~~

Другими словами, `return` с объектом возвращает этот объект, во всех остальных случаях возвращается `this`.

Можно не ставить круглые скобки после `new`, если вызов конструктора идёт без аргументов.

Что будет если мы поместим обычную функцию внутрь функции-конструктора и вызовем её там?

~~~
function getThis() {
  console.log(this); // Window | undefined в strict mode
}

function User(saying) {
  this.saying = saying;
  getThis();
}

new User("Нi!");
~~~

`getThis()` указывает на `Window`, несмотря на то что `getThis()` вызывается внутри функции конструктора.

Конечно, мы можем добавить к `this` не только свойства, но и методы:

~~~
function User(name) {
  this.name = name;
  this.sayHi = function() {
    console.log('My name is ' + this.name);
  }
}

new User('Tom').sayHi();  // My name is Tom
~~~

### this в стрелочных функциях

~~~
const globalThis = this;
const globalArrowFunc = () => {
  console.log(this);
}

console.log(globalThis); // Window независимо от режима
globalArrowFunc(); // Window независимо от режима
~~~

Стрелочная функция указывает на `window` независимо от режима, потому что она была создана внутри глобального контекта, как и переменная `globalThis`.

Для стрелочных функций `this` определяется в момент их создания и больше никогда не изменяется:

~~~
const user = {
  userThis: this, 
  arrowFunc: () => {
    console.log(this);
  }
};

console.log(user.userThis); // Window независимо от режима. Поскольку этот объект находится внутри глобального контекста (window), то и this в этом случае указывает на window. 
user.arrowFunc(); // Window независимо от режима
~~~

Поскольку стрелочная функция запомнила свой контекст в момент создания, то и при вызове она будет ссылаться именно на него. Таким образом мы получаем `window`. Стрелочная функция в момент создания ищет ближайший к ней контекст и запоминает его, а он у нас точно такой же как и в случае простого присвоения внутри `user.userThis`.

### this во вложенных функциях

~~~
const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
  funcArrow() {
    return () => {
      console.log(this);
    }
  },
  arrowFunc: () => {
    return function() {
      console.log(this);
    }
  },
  arrowArrow: () => {
    return () => {
      console.log(this);
    }
  },
};
user.funcFunc()(); // Window | undefined в strict mode
user.funcArrow()(); // user
user.arrowFunc()(); // Window | undefined в strict mode
user.arrowArrow()(); // Window независимо от режима
~~~

1. Когда вызывается `user.funcFunc()` — нам возвращается функция `function`. Обратим внимание, что мы сразу же вызываем ее и в этот момент у нее нет никакого объекта перед точкой, а значит `this` ссылается на `window`.
2. Когда мы вызвали метод `funcArrow`, то создали стрелочную функцию и в момент создания она запомнила окружающий ее контекст. Теперь она всегда будет его возвращать и никогда не потеряет.
3. Насчет `arrowFunc`. Здесь все то же самое, что и первом примере. Нам не важно, каким образом была создана `function`. Важно лишь что в момент ее вызова у нее нет объекта перед точкой.
4. Внутри метода `arrowArrow` `this` указывал на `window`, так как стрелочные функции не имеют своего контекста.

Еще один пример:

~~~
const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
  funcArrow() {
    return () => {
      console.log(this);
    }
  },
  arrowFunc: () => {
    return function() {
      console.log(this);
    }
  },
  arrowArrow: () => {
    return () => {
      console.log(this);
    }
  },
};
const user2 = {
  name: 'Jim',
  funcFunc: user.funcFunc(),
  funcArrow: user.funcArrow(),
  arrowFunc: user.arrowFunc(),
  arrowArrow: user.arrowArrow()
}
user2.funcFunc(); // user2
user2.funcArrow(); // user
user2.arrowFunc(); // user2
user2.arrowArrow(); // window
~~~

1. В момент вызова `funcFunc()`, перед точкой объект `user2`, поэтому на него и будет ссылаться `this`.
2. Когда мы вызвали `user.funcArrow()`, мы вернули стрелочную функцию, но в момент создания она запомнила ближайший к ней контекст (`user`) и теперь она всегда и везде будет возвращать именно его.
3. Насчет `arrowFunc()` точно то же самое, что и насчет `funcFunc()`. Мы вернули функцию и вызвали ее. Объект перед точкой `user2`, поэтому `this` указывает именно на него.
4. Когда мы вызвали `user.arrowArrow()`, то вернули стрелочную функцию, которая в момент создания запомнила ближайший контекст (`window`). Теперь она всегда будет указывать на него.

### this в классах

~~~
class Animal {
  speak() {
    console.log(this);
  }
  static eat() {
    console.log(this);
  }
}

const obj = new Animal();
obj.speak(); // объект Animal
const speak = obj.speak;
speak(); // undefined

Animal.eat() // класс Animal
const eat = Animal.eat;
eat(); // undefined
~~~

Код внутри тела класса всегда выполняется в строгом режиме.

Сравнение class и object:

~~~
const user = {
  userThis: this,
  name: "Bob",
  arrowArrow: () => {
    console.log(this.userThis); // Window
    return () => {
      console.log(this); // Window
    };
  },
}

user.arrowArrow()();
~~~

~~~
class User {
  userThis = this;
  name = "Bob";
  arrowArrow = () => {
    console.log(this.userThis); // user
    return () => {
      console.log(this); // user
    };
  };
}

const user = new User();
user.arrowArrow()();
~~~

***

### Difference between function and method

Функция существует сама по себе. Вызывается откуда угодно и для чего угодно. _Метод_ - функция привязанная к объекту. Метод одного объекта можно привязать к другому объекту. Метод может получить доступ к свойствам объекта. Вызывается только для объекта.

### this possible issues

Проблемой может быть что при работе с вызовом функции можно подумать, что `this` во внутренней функции такой же, как и во внешней. Контекст внутренней функции зависит только от вызова, а не от контекста внешней функции. Чтобы получить ожидаемый `this`, можно модифицировать контекст внутренней функции при помощи непрямого вызова (используя `call()`, `apply()` или `bind()`).

~~~
const user = {
  name: 'Bob',
  funcFunc() {
    return function() {
      console.log(this);
    }
  },
};

user.funcFunc().apply(user); // { name: 'Bob', funcFunc: ƒ }
~~~

Потеря контекста:

~~~
function Animal(type, legs) {
  this.type = type;
  this.legs = legs;
  this.logInfo = function() {
    console.log('The ' + this.type + ' has ' + this.legs + ' legs');
  };
}

const myCat = new Animal('Cat', 4);
setTimeout(myCat.logInfo, 1000); // The undefined has undefined legs
setTimeout(() => myCat.logInfo(), 1000); // The Cat has 4 legs. Работает, так как сначала вызывается, потом присваивается новый контекст.
setTimeout(myCat.logInfo.bind(myCat), 1000); // The Cat has 4 legs
~~~

Можно подумать, что `setTimeout` вызовет `myCat.logInfo()`, которая запишет информацию об объекте `myCat`. Но метод отделяется от объекта, когда передаётся в качестве параметра: `setTimeout(myCat.logInfo)`, и через секунду происходит вызов функции. Когда `logInfo` вызывается как функция, `this` становится глобальным объектом или `undefined` (но не объектом `myCat`), поэтому информация об объекте выводится некорректно.

Стрелочная функция не создаёт свой контекст исполнения, а заимствует this из внешней функции, в которой она определена.

~~~
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  log() {
    console.log(this === myPoint); // true
    setTimeout(() => {
      console.log(this === myPoint); // true
      console.log(this.x + ':' + this.y); // '95:165'
    }, 1000);
  }
}
const myPoint = new Point(95, 165);
myPoint.log();
~~~

Еще пример:

~~~
const numbers = {
  num1: 5,
  num2: 10,
  sum: function() {
    console.log(this === numbers); // => true
    function calculate() {
      // this is window or undefined in strict mode
      console.log(this === numbers); // => false
      return this.num1 + this.num2;
    }
    return calculate(); // NaN. Но если написать return calculate.call(this) или сделать функцию calculate стрелочной; // 15
  }
};

console.log(numbers.sum());
~~~

Функция `calculate` определена внутри `sum`, поэтому вы можете ожидать, что `this` — это объект numbers и в `calculate()`. Тем не менее, `calculate()` — это вызов функции, а не метода, и поэтому его `this` — это глобальный объект `window` или `undefined` в `strict mode`. Даже если контекстом внешней функции sum является объект numbers, у него здесь нет власти.
