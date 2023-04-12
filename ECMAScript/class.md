# ?Class

Синтаксис выглядит так:

~~~
class User {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    console.log(this.name);
  }
}

const user = new User("Иван");
user.sayHi();
~~~

Когда вызывается `new User("Иван")`:

* Создаётся новый объект.
* `constructor` запускается с заданным аргументом и сохраняет его в `this.name`.
* Затем можно вызывать на объекте методы, такие как `user.sayHi()`.

### Difference between class and constructor function

Разница между объявлением функции (`function declaration`) и объявлением класса (`class declaration`) в следующем:

* Объявление функции совершает подъём (`hoisting`), в то время как объявление класса — нет. Поэтому вначале необходимо объявить ваш класс и только затем работать с ним.
* В отличие от обычных функций, конструктор класса не может быть вызван без `new`.
* Классы всегда используют `use strict`. Весь код внутри класса автоматически находится в строгом режиме.

### Getter/setter

В классах можно объявлять геттеры/сеттеры:

~~~
class User {
  constructor(name) {
    // вызывает сеттер
    this.name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    if (value.length < 4) {
      console.log("Имя слишком короткое.");
      return;
    }
    this._name = value;
  }
}

let user = new User("Иван");
console.log(user.name); // Иван
user = new User(""); // Имя слишком короткое.
~~~

При объявлении класса геттеры/сеттеры создаются на User.prototype, вот так:

~~~
Object.defineProperties(User.prototype, {
  name: {
    get() {
      return this._name;
    },
    set(name) {
      // ...
    },
  },
});
~~~

### super()

В конструкторе ключевое слово `super()` используется как функция, вызывающая родительский конструктор. В конструкторе потомка мы обязаны вызвать `super()` до обращения к `this`. До вызова `super` не существует `this`, так как по спецификации в этом случае именно `super` инициализирует `this`. Ключевое слово `super` также может быть использовано для вызова функций родительского объекта.

~~~
class Animal {
  constructor(name) {
    this.name = name;
  }

  walk() {
    return `I walk: ${this.name}`;
  }
}

class Rabbit extends Animal {
  walk() {
    console.log(`${super.walk()} ...and jump!`);
  }
}

new Rabbit("Cheerio").walk(); // I walk: Cheerio ...and jump!
console.log(Rabbit.prototype.__proto__ == Animal.prototype); // true
~~~

Как видно из примера выше, методы родителя (`walk`) можно переопределить в наследнике. При этом для обращения к родительскому методу используют `super.walk()`.
