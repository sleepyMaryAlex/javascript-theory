# ?Object Oriented Programming

### `new` keyword

Обычный синтаксис `{...}` позволяет создать только один объект. Но зачастую нам нужно создать множество похожих, однотипных объектов, таких как пользователи, элементы меню и так далее. Это можно сделать при помощи функции-конструктора и оператора `new`.

Оператор `new` используется для создания объектов. Операндом этого оператора должна быть функция. Функция, которая создаётся специально для работы с оператором `new`, называется конструктором. Конструктор используется для инициализации нового созданного объекта. Работает это всё (оператор `new` с конструктором) следующим образом: встречая оператор `new` интерпретатор создаёт новый пустой объект, затем он вызывает конструктор, и передаёт ему новый созданный объект в качестве значения ключевого слова `this`. Внутри конструктора происходит инициализация свойств вновь созданного объекта. После того, как объект создан и инициализарован, оператор `new` возвращает созданный объект.

### Function constructor

Функции-конструкторы технически являются обычными функциями. Но есть два соглашения:
1. Имя функции-конструктора должно начинаться с большой буквы (как соглашение).
2. Функция-конструктор должна выполняться только с помощью оператора `"new"` (как обязательное условие).

~~~
function User(name) {
  // this = {}; (неявно)

  // добавляет свойства к this
  this.name = name;
  this.isAdmin = false;

  // return this; (неявно)
}

console.log(new User('Mary'));
~~~

Когда функция вызывается как `new User(...)`, происходит следующее:
1. Создаётся новый пустой объект, и он присваивается `this`.
2. Выполняется тело функции. Обычно оно модифицирует `this`, добавляя туда новые свойства.
3. Возвращается значение `this`.

~~~
function User(name) {
  this.name = name;
  this.sayHi = function() {
    console.log(this.name);
  };
}
~~~

Функцией-конструктором может быть только функция с ключевым словом `function`.

Мы можем использовать конструкторы для создания множества похожих объектов.
JavaScript предоставляет функции-конструкторы для множества встроенных объектов языка: таких как `Date`, `Set`, и других.

### Public, private, static members

_Публичные поля_ доступны извне, и мы можем к им обратиться в любом месте программы. Иногда необходимо, чтобы к данным или действиям извне класса нельзя было обратиться, и чтобы к ним можно было обращаться только внутри этого же класса. Или иными словами, сделать свойства и методы класса _приватными_ - доступными только для этого класса. Для этого название полей и методов должно начинаться с символа решетки `#`. Если потребуется как-то все-таки обратиться к свойствам и методам, то мы можем определить для этого методы `get` и `set`. Есть также _защищенные свойства_. Защищённые свойства обычно начинаются с префикса `_`. Это не синтаксис языка: есть хорошо известное соглашение между программистами, что такие свойства и методы не должны быть доступны извне. Для этого обычно есть методы `get` и `set`. Большинство программистов следуют этому соглашению. Статические поля относятся ко всему классу, а не к объекту. Перед названием _статического поля_ ставится ключевое слово `static`.

##### Public members

~~~
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  print() {
    console.log(`${this.name} ${this.age}`);
  }
}

const tom = new Person("Tom", 25);
tom.print(); // Tom 25
console.log(tom.name); // Tom
~~~

##### Private members

~~~
class Person {
  #name;
  #age;
  constructor(name, age) {
    this.#name = name;
    this.#age = age;
  }
  print() {
    console.log(`${this.#name} ${this.#age}`);
  }
}

const tom = new Person("Tom", 25);
tom.print(); // Tom 25
console.log(tom.name); // undefined
~~~


##### Static members

~~~
class Person {
  static retirementAge = 65;
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  print() {
    console.log(`${this.name} ${this.age}`);
  }
}

console.log(Person.retirementAge); // 65
const tom = new Person("Tom", 25);
tom.print(); // Tom 25
console.log(tom.retirementAge); // undefined
~~~

### OOP emulation patterns and conventions

Рассмотрим следующий очень простой пример, написанный с использованием псевдоклассов:

~~~
function Animal() {
  this.offspring = [];
}

Animal.prototype.makeBaby = function () {
  const baby = new Animal();
  this.offspring.push(baby);
  return baby;
};

function Cat() {}

Cat.prototype = new Animal();

const puff = new Cat();
puff.makeBaby();

const colonel = new Cat();
colonel.makeBaby();
console.log(colonel.offspring, puff.offspring); // (2) [Animal, Animal] (2) [Animal, Animal]
~~~

Пример выглядит достаточно невинно. Это шаблон наследования, который вы увидите во многих местах по всему Интернету. Однако здесь происходит кое-что забавное — если вы проверите `colonel.offspring` и `puff.offspring`, то заметите, что в каждом из них есть два одинаковых младенца! Проблема в том, что когда мы вызываем `makeBaby`, механизм JavaScript ищет свойство `offspring` в объекте `Cat` и не может его найти. Затем он находит свойство в `Cat prototype` и добавляет нового потомка в общий объект, от которого наследуются оба отдельных объекта. Один из способов решения проблемы заключается в том, чтобы гарантировать, что объект, для которого вызывается функция `makeBaby`, имеет собственное `offspring` свойство:

~~~
function Animal() {
  this.offspring = [];
}

Animal.prototype.makeBaby = function () {
  const baby = new Animal();
  if (!this.hasOwnProperty("offspring")) {
    this.offspring = [];
  }
  this.offspring.push(baby);
  return baby;
};

function Cat() {}
Cat.prototype = new Animal();

const puff = new Cat();
puff.makeBaby();

const colonel = new Cat();
colonel.makeBaby();
console.log(colonel.offspring, puff.offspring); // (1) [Animal] (1) [Animal]
~~~
