# ?Function methods

__call()__ вызывает функцию с указанным значением `this` и индивидуально предоставленными аргументами.

__apply()__ вызывает функцию с указанным значением `this` и аргументами, предоставленными в виде массива.

__bind()__ создаёт новую функцию, которая при вызове устанавливает в качестве контекста выполнения `this` предоставленное значение. В метод также передаются индивидуально предоставленными аргументами.

__toString()__ возвращает результат в виде строки.

### Call, apply and bind. Example

~~~
const user = { name: "Mary" };

function foo(age) {
  console.log(this.name, age);
}

foo.call(user, 24);
foo.apply(user, [24]);
const fn = foo.bind(user, 24);
fn();
// Все вернут Mary 24
~~~

### Binding one function twice

`bind` возвращает новую функцию, которая сохраняет контекст исходной функции нетронутым.

~~~
Function.prototype.bindTwice = function (context, originalFunction, ...args) {
  return () => {
    originalFunction.call(context, ...args);
  };
};

const user1 = { name: "Mary" };
const user2 = { name: "Alex" };

function foo(age) {
  console.log(this.name, age);
}

const fn1 = foo.bind(user1, 24);
fn1(); // Mary 24

// Здесь все равно будет старое значение, нельзя переназначить:
const bound = fn1.bind(user2, 21);
bound(); // Mary 24

const fn2 = fn1.bindTwice(user2, foo, 25);
fn2(); // Alex 25
~~~

Независимо от того, как мы вызываем `fn1`, `originalFunction` будет вызываться с определенным контекстом. Мы можем повторно связать `fn1`, которая вернет еще одну новую функцию, но это не повлияет на исходную функцию и контекст "внутренней оболочки".

### toString. Example

~~~
function sum(a, b) {
  return a + b;
}

console.log(sum.toString());

// Expected output: 
"function sum(a, b) {
  return a + b;
}"
~~~
