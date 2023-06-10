# ?Behavioral Patterns

Согласно Википедии, _behavioral patterns_ (поведенческие шаблоны) — шаблоны проектирования, определяющие алгоритмы и способы реализации взаимодействия различных объектов и классов.

Проще говоря, поведенческие паттерны связаны с распределением обязанностей между объектами и описывают структуру и шаблоны для передачи сообщений / связи между компонентами.

К ним относятся:

* Chain of Responsibility
* Command
* Interpreter
* Iterator
* Mediator
* Memento
* Observer
* State
* Strategy
* Template
* Visitor

По мнению разработчиков MediaSoft Template Method, Iterator и Observer — самые используемые поведенческие паттерны в разработке.

### Chain of Responsibility (Цепочка обязанностей)

_Chain of Responsibility_ позволяет у одного и того же объекта последовательно вызывать набор операций и последовательно их модифицировать.

~~~
class MySum {
  constructor(initiaValue = 90) {
    this.sum = initiaValue;
  }
  
  add(value) {
    this.sum += value;
    return this;
  }
}

const sum1 = new MySum();
console.log(sum1.add(1).add(9).add(10).sum); // 110
~~~

Chain of Responsibility заключается в строчке `return this` и этот паттерн является цепочкой.

### Command (Команда)

Смысл _Command_ отделить объект-источник запроса от объекта, принимающего и выполняющего эти запросы.

Другими словами, Command убирает прямую зависимость между объектами, вызывающими операции, и объектами, которые их непосредственно выполняют.

~~~
class MyMath {
  constructor(initiaValue = 0) {
    this.num = initiaValue;
  }

  square() {
    return this.num ** 2;
  }

  cube() {
    return this.num ** 3;
  }
}

class Command {
  constructor(subject) {
    this.subject = subject;
    this.commandsExecuted = [];
  }

  execute(command) {
    this.commandsExecuted.push(command);
    return this.subject[command]();
  }
}

const myMath = new MyMath(2);
const command = new Command(myMath);
console.log(command.execute("square")); // 4
console.log(command.execute("cube")); // 8
console.log(command.commandsExecuted); // ['square', 'cube']
~~~

### Interpreter (Интерпретатор)

_Interpreter_ определяет представление грамматики для заданного языка и интерпретатор предложений этого языка. Как правило, данный шаблон проектирования применяется для часто повторяющихся операций.

~~~
class Numeral {
  constructor(value) {
    this.value = parseInt(value);
  }

  interpret() {
    return this.value;
  }
}

class Add {
  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() + this.right.interpret();
  }
}

class Subtract {
  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() - this.right.interpret();
  }
}

const sentence = "5 + 4 - 3 + 7 - 2";

const tokens = sentence.split(" ");

const AST = []; // Abstract Syntax Tree(AST)
AST.push(new Add(new Numeral(tokens[0]), new Numeral(tokens[2]))); // 5 + 4
AST.push(new Subtract(AST[0], new Numeral(tokens[4]))); // - 3
AST.push(new Add(AST[1], new Numeral(tokens[6]))); // + 7
AST.push(new Subtract(AST[2], new Numeral(tokens[8]))); // - 2

const ASTRoot = AST.pop();

// Interpret recursively through the full AST starting from the root.
console.log(ASTRoot.interpret()); // 11
~~~

### Iterator (Итератор)

_Iterator_ нужен для последовательного перебора составных элементов коллекции, не раскрывая их внутреннего представления.

Iterator стоит использовать для перебора сложных составных объектов коллекции, если необходимо скрыть детали реализации от клиента.

~~~
class Iterator {
  constructor(data) {
    this.index = 0;
    this.data = data;
  }

  next() {
    if (this.hasNext()) {
      return {
        value: this.data[this.index++],
        done: false,
      };
    }

    return {
      value: undefined,
      done: true,
    };
  }

  hasNext() {
    return this.index < this.data.length;
  }
}

const cars = ["BMW", "Audi", "Honda"];

const carsIterator = new Iterator(cars);

console.log(carsIterator.next()); // {value: 'BMW', done: false}
console.log(carsIterator.next()); // {value: 'Audi', done: false}
console.log(carsIterator.hasNext()); // true
console.log(carsIterator.next()); // {value: 'Honda', done: false}
console.log(carsIterator.next()); // {value: 'Honda', done: false}
console.log(carsIterator.hasNext()); // false
~~~

### Mediator (Посредник)

_Mediator_ позволяет выстраивать тесную коммуникацию между различными объектами. При этом он предоставляет некоторую централизованную абстракцию, которая позволяет взаимодействовать группе объектов через друг друга.

~~~
class User {
  constructor(name) {
    this.name = name;
    this.room = null;
  }

  send(message, to) {
    this.room.send(message, this, to);
  }

  receive(message, from) {
    console.log(`${from.name} => ${this.name}: ${message}`);
  }
}

class ChatRoom {
  constructor() {
    this.users = {};
  }

  register(user) {
    this.users[user.name] = user;
    user.room = this;
  }
  
  send(message, from, to) {
    if (to) {
      to.receive(message, from);
    } else {
      Object.keys(this.users).forEach((key) => {
        if (this.users[key] !== from) {
          this.users[key].receive(message, from);
        }
      });
    }
  }
}

const alex = new User("Alex");
const mary = new User("Mary");
const dima = new User("Dima");

const room = new ChatRoom();
room.register(alex);
room.register(mary);
room.register(dima);

alex.send("Hello", mary); // Alex => Mary: Hello
mary.send("Hi", alex); // Mary => Alex: Hi
dima.send("Hi there");
// Dima => Alex: Hi there
// Dima => Mary: Hi there
~~~

### Memento (Снимок)

_Memento_ позволяет сохранять и восстанавливать прошлые состояния объектов, не раскрывая подробностей их реализации.

~~~
class Originator {
  returnState(memento) {
    this.state = memento.getState();
    console.log(`State = ${this.state}`);
  }

  newValue(state) {
    return new Memento(state);
  }
}

class Memento {
  constructor(state) {
    this.state = state;
  }

  getState() {
    return this.state;
  }
}

class Caretaker {
  constructor() {
    this.mementos = [];
    this.indexMementos = false;
  }

  saveState(memento) {
    this.mementos.push(memento);
  }

  getBackMemento() {
    if (this.indexMementos === false) {
      this.indexMementos = this.mementos.length - 2;
    } else if (this.indexMementos > 0) {
      this.indexMementos = this.indexMementos - 1;
    }
    return this.mementos[this.indexMementos];
  }

  getForwardMemento() {
    if (
      this.indexMementos >= 0 &&
      this.indexMementos <= this.mementos.length - 2
    ) {
      this.indexMementos = this.indexMementos + 1;
    }
    return this.mementos[this.indexMementos];
  }
}

const caretaker = new Caretaker();
const originator = new Originator();

caretaker.saveState(originator.newValue("a"));
caretaker.saveState(originator.newValue("b"));
caretaker.saveState(originator.newValue("c"));

originator.returnState(caretaker.getBackMemento()); // State = b
originator.returnState(caretaker.getBackMemento()); // State = a
originator.returnState(caretaker.getForwardMemento()); // State = b
originator.returnState(caretaker.getForwardMemento()); // State = c

console.log(originator.state); // c
~~~

### Observer (Наблюдатель)

_Observer_ позволяет одному объекту подписываться на другие объекты и отслеживать их изменения.

Пример из жизни: после того как вы оформили подписку на газету или журнал, вам больше не нужно ездить в супермаркет и проверять, вышел ли очередной номер. Издательство будет присылать новые номера по почте прямо к вам домой сразу после их выхода.

Часто используется при проектировании библиотек, которые управляют состоянием приложения.

~~~
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  unsubscribe(observer) {
    this.observers = this.observers.filter((obs) => obs !== observer);
  }

  fire(changes) {
    this.observers.forEach((observer) => observer.update(changes));
  }
}

class Observer {
  constructor(state) {
    this.state = state;
    this.initialState = state;
  }

  update(changes) {
    switch (changes.type) {
      case "INCREMENT":
        this.state = ++this.state;
        break;
      case "DECREMENT":
        this.state = --this.state;
        break;
      case "ADD":
        this.state += changes.payload;
        break;
      default:
        this.state = this.initialState;
    }
  }
}

const stream = new Subject();
const obs1 = new Observer(1);
const obs2 = new Observer(42);

stream.subscribe(obs1);
stream.subscribe(obs2);
console.log(obs1.state); // 1
console.log(obs2.state); // 42

stream.fire({ type: "INCREMENT" });
console.log(obs1.state); // 2
console.log(obs2.state); // 43

stream.fire({ type: "ADD", payload: 10 });
console.log(obs1.state); // 12
console.log(obs2.state); // 53
~~~

### State (Состояние)

Мы можем создавать какие-то различные классы, которые будут являться элементами `state`, и мы можем делегировать изменения состояния этих классов на какой-то общий класс, который будет являться `state` и который будет менять внутреннее состояние этих отдельных элементов. Вся логика будет построена именно на верхнем уровне.

~~~
class Light {
  constructor(light) {
    this.light = light;
  }
}

class RedLight extends Light {
  constructor() {
    super("red");
  }

  sign() {
    return "STOP";
  }
}

class YellowLight extends Light {
  constructor() {
    super("yellow");
  }

  sign() {
    return "GET READY";
  }
}

class GreenLight extends Light {
  constructor() {
    super("green");
  }

  sign() {
    return "GO";
  }
}

class TrafficLight {
  constructor() {
    this.states = [new RedLight(), new YellowLight(), new GreenLight()];
    this.current = this.states[0];
  }

  change() {
    const total = this.states.length;
    const index = this.states.findIndex((light) => light === this.current);
    if (index + 1 < total) {
      this.current = this.states[index + 1];
    } else {
      this.current = this.states[0];
    }
  }

  sign() {
    return this.current.sign();
  }
}

const traffic = new TrafficLight();
console.log(traffic.sign()); // STOP
traffic.change();
console.log(traffic.sign()); // GET READY
traffic.change();
console.log(traffic.sign()); // GO
~~~

### Strategy (Стратегия)

_Strategy_ определяет семейство схожих алгоритмов и помещает каждый из них в собственный класс, после чего алгоритмы можно взаимозаменять прямо во время исполнения программы.

~~~
class Vehicle {
  travelTime() {
    return this.timeTaken;
  }
}

class Bus extends Vehicle {
  constructor() {
    super();
    this.timeTaken = 10;
  }
}

class Taxi extends Vehicle {
  constructor() {
    super();
    this.timeTaken = 5;
  }
}

class Car extends Vehicle {
  constructor() {
    super();
    this.timeTaken = 3;
  }
}

class Commute {
  travel(transport) {
    return transport.travelTime();
  }
}

const commute = new Commute();
console.log(commute.travel(new Bus())); // 10
console.log(commute.travel(new Taxi())); // 5
console.log(commute.travel(new Car())); // 3
~~~

### Template Method (Шаблонный метод)

_Template Method_ или шаблонный метод описывает скелет алгоритма, перекладывая ответственность за некоторые его шаги на подклассы. Паттерн позволяет подклассам переопределять шаги алгоритма, не меняя его общей структуры.

Строители используют подход, похожий на шаблонный метод, при строительстве типовых домов. У них есть основной архитектурный проект, в котором расписаны шаги строительства: заливка фундамента, постройка стен, перекрытие крыши, установка окон и так далее.

Но, несмотря на стандартизацию каждого этапа, строители могут вносить небольшие изменения на любом из этапов, чтобы сделать дом чуточку непохожим на другие.

Используется для избежания дублирования кода. Например, при описании шаблонов.

~~~
// создаем абстрактный класс Car, которому присваиваем общее свойство
// cost и метод getCost
class Car {
  constructor(cost) {
    this.cost = cost;
  }

  getCost() {
    return `Стоимость автомобиля: ${this.cost}`;
  }
}

// создаем подклассы от класса Car, которые расширяют его поведение
class SportCar extends Car {
  constructor(cost, speed) {
    super(cost);
    this.speed = speed;
  }

  getSpeed() {
    return `Скорость: ${this.speed}`;
  }
}

class Truck extends Car {
  constructor(cost, capacity) {
    super(cost);
    this.capacity = capacity;
  }

  getCapacity() {
    return `Вместимость: ${this.capacity}`;
  }
}

const sport = new SportCar(16000, 300);
const truck = new Truck(20000, 1000);

// как мы видим, теперь у экземпляра класса SportCar
// есть метод getSpeed, а у экземпляра Truck метод
// getCapacity, но также у обоих инстансов есть
// метод абстрактного класса Car
console.log(sport.getCost()); // Стоимость автомобиля: 16000
console.log(sport.getSpeed()); // Скорость: 300
console.log(truck.getCapacity()); // Вместимость: 1000
~~~

### Visitor (Посетитель)

_Visitor_ позволяет добавлять в программу новые операции, не изменяя классы объектов, над которыми эти операции могут выполняться.

~~~
class Monkey {
  shout() {
    console.log("Ooh oo aa aa!");
  }

  accept(operation) {
    operation.visitMonkey(this);
  }
}

class Lion {
  roar() {
    console.log("Roaaar!");
  }

  accept(operation) {
    operation.visitLion(this);
  }
}

class Dolphin {
  speak() {
    console.log("Tuut tuttu tuutt!");
  }

  accept(operation) {
    operation.visitDolphin(this);
  }
}

// Visitor
const speak = {
  visitMonkey(monkey) {
    monkey.shout();
  },

  visitLion(lion) {
    lion.roar();
  },

  visitDolphin(dolphin) {
    dolphin.speak();
  },
};

const monkey = new Monkey();
const lion = new Lion();
const dolphin = new Dolphin();

monkey.accept(speak); // Ooh oo aa aa!
lion.accept(speak); // Roaaar!
dolphin.accept(speak); // Tuut tutt tuutt!
~~~
