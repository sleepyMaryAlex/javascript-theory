# ?Block scope | function scope | global scope

_Область видимости (scope)_ - это область программы, в которой можно получить доступ к переменной.

__Block scope__

До ES6 (2015) у JavaScript были только global Scope и function Scope.

ES6 представил два важных новых ключевых слова JavaScript: _let_ и _const_. Эти два ключевых слова обеспечивают область видимости блока в JavaScript.

К переменным c _let_ и _const_, объявленным внутри блока (if, switch и т.д.), нельзя получить доступ снаружи блока.

Переменные, объявленные с помощью ключевого слова _var_, не могут иметь область видимости блока, а имеют область видимости функции.

Переменные, объявленные без ключевого слова, станут глобальными переменными.

~~~
function fn() {
  if (true) {
    var a = 7;
    let b = 6;
    let c = 5;
    d = 4;
  }
  console.log(a); // 7
  console.log(b); // b is not defined
  console.log(c); // c is not defined
  console.log(d); // 4
}

fn();

console.log(a); // a is not defined
console.log(b); // b is not defined
console.log(c); // c is not defined
console.log(d); // 4; работает только после вызова функции
~~~

Еще один пример с _var_:

~~~
if (true) {
  var n = 6;
}

console.log(n); // 6
~~~

Следующий код создаёт массив функций:

~~~
function makeArmy() {
  const shooters = [];
  for (var i = 0; i < 10; i++) {
    shooters.push(() => {
      console.log(i);
    });
  }
  return shooters;
}

const army = makeArmy();
army[0](); // 10
army[5](); // 10
~~~

Почему все функции shooters показывают одно и то же?

Можно представить цикл следующим образом:

~~~
function makeArmy() {
  const shooters = [];
  {
    var i = 0;
    shooters.push(() => {
      console.log(i);
    });
  }
  {
    var i = 1;
    shooters.push(() => {
      console.log(i);
    });
  }
  ...
  return shooters;
}
~~~

Функция makeArmy делает следующее:
1. Создаёт пустой массив shooters.
2. В цикле заполняет массив элементами через shooters.push. При этом каждый элемент массива – это функция, так что в итоге после цикла массив будет таким:

~~~
const shooters = [
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); },
  () => { console.log(i); }
];
~~~

Этот массив возвращается из функции.
3. Вызов функций shooters – это получение элемента массива, и тут же – её запуск.

Разберемся, почему все стрелки выводят одно и то же значение.

* В функциях shooters отсутствует переменная i. Когда такая функция вызывается, то i она берет из внешнего LexicalEnvironment.
* К моменту вызова ```army[0]()```, функция makeArmy уже закончила работу. Цикл завершился, последнее значение было i = 10.
* В результате все функции shooter получают из внешнего лексического окружения это, одно и то же, последнее, значение i = 10.

Мы можем исправить эту ситуацию сменив _var_ на _let_. Каждый раз, когда выполняется блок кода for (let i = 0...) {...}, для него создаётся новое лексическое окружение с соответствующей переменной i.

Если с переменными все понятно, то как насчет function declaration? Function declaration всегда локальны для текущей области, как переменная, объявленная с помощью ключевого слова var.

Интересный пример. Сравнение var и function declaration:

~~~
let i = 5;
function fn() {
  if (i < 6) {
    var n = 8;
  }
  function fn1() {
    console.log(n); // Если условие истинно, то 8, иначе undefined
  }

  fn1();
}

fn();
~~~

~~~
let i = 6;
function fn() {
  if (i < 6) {
    function fn2() {
      return 8;
    }
  }
  function fn1() {
    console.log(fn2()); // Если условие истинно, то 8, иначе TypeError: fn2 is not a function
  }

  fn1();
}

fn();
~~~

__Function scope__

Переменные, объявленные внутри функций (за исключением глобальных), имеют _области видимости функции_. Каждая функция имеет свою область видимости, т.е. переменные с одинаковыми названиями могут использоваться в разных функциях, поскольку они связаны с соответствующими функциями и не видны друг другу. Переменные, объявленные внутри функций, создаются при запуске функции и удаляются по завершении функции.

Переменные, объявленные внутри функций, доступны в дочерних функциях и блоках:

~~~
function fn1() {
  var num1 = 5;
  const num2 = 10;
  let num3 = 23;
  function fn3() {
    console.log(34);
  }
  function fn2() {
    console.log(num1); // 5
    console.log(num2); // 10
    console.log(num3); // 23
    fn3(); // 34
  }
  fn2();
}

fn1();
~~~

Если переменная объявлена без ключевых слов в функции, то она будет в глобальной области видимости. Но такая переменная должна быть инициализирована, иначе b is not defined.

~~~
function fn() {
  var a = b = 5;
  console.log(a); // 5
  console.log(b); // 5
}

fn();

console.log(a); // a is not defined
console.log(b); // 5, так как происходит следующее: var a = b; b = 5; работает только после вызова функции
~~~

Функции, определенные внутри функций, не могут быть вызваны из родительской области:

~~~
function fn() {
  if (true) {
    function sayHi() {
      console.log("Hi");
    }
  }
  function sayBye() {
    console.log("Bye");
  }
  sayHi(); // Hi
  sayBye(); // Bye
}

fn();
sayBye(); // Error
sayHi(); // Error
~~~

__Global scope__

Область за пределами всех функций или блоков считается _глобальной областью видимости_. Переменные, определенные в глобальной области, могут быть доступны и изменены в любых других областях.
Переменные, объявленные с помощью var, let и const, объявленные вне функции или блока, имеют глобальную область видимости.
Если вы присвоите значение переменной, которая не была объявлена, она автоматически станет глобальной переменной.

Также и функция, определенная в глобальной области, может быть вызвана в любых других областях и может получить доступ ко всем переменным и функциям, определенным в глобальной области. Функция, определенная внутри другой функции, также может получить доступ ко всем переменным и функциям, определенным в ее родительской функции, и к любым другим переменным и функциям, к которым имеет доступ родительская функция.

~~~
function fn2() {
  return 12;
}

function fn1() {
  function fn3() {
    console.log(fn2());
  }
  fn3(); // 12
}

fn1();
~~~