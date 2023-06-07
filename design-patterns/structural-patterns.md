# ?Structural Patterns

Согласно Википедии, _structural patterns_ (структурные шаблоны) — шаблоны проектирования, в которых рассматривается вопрос о том, как из классов и объектов образуются более крупные структуры.

Проще говоря, структурные паттерны связаны с композицией объектов или тем, как сущности могут использовать друг друга. К ним относятся:

* Adapter
* Bridge
* Composite
* Decorator
* Facade
* Flyweight
* Proxy

По мнению разработчиков MediaSoft Facade, Adaptor и Decorator — это самые используемые структурные паттерны в разработке.

### Adapter (Адаптер)

Согласно Википедии, _Adapter_ — структурный шаблон проектирования, предназначенный для организации использования функций объекта, недоступного для модификации, через специально созданный интерфейс.

Проще говоря, Adapter позволяет объектам с несовместимыми интерфейсами работать вместе.

Еще проще: европейские розетки отличаются от английских, поэтому, приезжая в Лондон, туристы обязательно берут переходник, или адаптер.

Часто используется, если мы работаем со сторонней библиотекой или компонентом, доступ к изменениями методов которого у нас отсутствует. Или когда есть несколько сторонних систем, доступ к которым должен осуществляться единообразно через одинаковый интерфейс.

~~~
// У нас есть класс калькулятора, который реализует единственный метод "operation"
class OldCalculator {
  operation(num1, num2, operation) {
    switch (operation) {
      case "multiplication":
        return num1 * num2;
      case "division":
        return num1 / num2;
      default:
        return NaN;
    }
  }
}

// Допустим нам нужно создать улучшенную версию калькулятора
class NewCalculator {
  add(num1, num2) {
    return num1 + num2;
  }
  div(num1, num2) {
    return num1 / num2;
  }
  mult(num1, num2) {
    return num1 * num2;
  }
}

// Но возникает проблема, что нет обратной совместимости со старым калькулятором,
// как раз в этом нам и поможет адаптер. Мы адаптируем новый калькулятор под функционал старого.
class CalculatorAdapter {
  constructor() {
    this.calculator = new NewCalculator();
  }

  operation(num1, num2, operation) {
    switch (operation) {
      case "add":
        return this.calculator.add(num1, num2);
      case "multiplication":
        return this.calculator.mult(num1, num2);
      case "division":
        return this.calculator.div(num1, num2);
      default:
        return NaN;
    }
  }
}

// Обратите внимание, мы используем адаптер вместо старого калькулятора
const calcAdapter = new CalculatorAdapter();
const sumAdapter = calcAdapter.operation(2, 2, "multiplication");
console.log(sumAdapter); // 4

// А новый функционал используем от экземпляра класса нового калькулятора
const newCalculator = new NewCalculator();
const sum = newCalculator.mult(2, 2);
console.log(sum); // 4
~~~

### Bridge (Мост)

_Bridge_ — это структурный шаблон проектирования, который отделяет абстракцию объекта от его реализации, чтобы их можно было разрабатывать независимо.

Он делает это, создавая мост между абстракцией и реализацией, позволяя им изменяться независимо друг от друга.

Паттерн Мост состоит из двух основных компонентов:

1. `Abstraction`: определяет высокоуровневый интерфейс объекта и делегирует детали реализации объекту `Implementor`.
2. `Implementor`: определяет интерфейс для конкретных реализаций, которые затем используются объектом `Abstraction`.

В целом, шаблон Bridge полезен в ситуациях, когда существует несколько реализаций абстракции и когда изменения в реализации не должны влиять на интерфейс абстракции.

~~~
// Implementation interface
class Implementor {
  operation() {}
}

// Concrete implementation classes
class ConcreteImplementorA extends Implementor {
  operation() {
    return "ConcreteImplementorA";
  }
}

class ConcreteImplementorB extends Implementor {
  operation() {
    return "ConcreteImplementorB";
  }
}

// Abstraction interface
class Abstraction {
  constructor(implementor) {
    this.implementor = implementor;
  }

  operation() {}
}

// Refined abstraction classes
class RefinedAbstraction extends Abstraction {
  operation() {
    return `RefinedAbstraction with ${this.implementor.operation()}`;
  }
}

// Usage
const implementorA = new ConcreteImplementorA();
const implementorB = new ConcreteImplementorB();
const abstraction1 = new RefinedAbstraction(implementorA);
const abstraction2 = new RefinedAbstraction(implementorB);

console.log(abstraction1.operation()); // RefinedAbstraction with ConcreteImplementorA
console.log(abstraction2.operation()); // RefinedAbstraction with ConcreteImplementorB
~~~

### Composite (Компоновщик)

_Composite_ — это структурный паттерн проектирования, который позволяет сгруппировать множество объектов в древовидную структуру, а затем работать с ней так, как будто это единичный объект.

Компоновщик рекурсивно запускает действие по всем элементам дерева — от корня к листьям.

~~~
class Component {
  constructor(props) {
    this.props = props;
  }

  update() {
    console.log("Default update");
  }
}

class Engine extends Component {
  update() {
    for (const child of this.props.children) {
      child.update();
    }
  }
}

class EntityManager extends Component {
  update() {
    for (const child of this.props.children) {
      child.update();
    }
  }
}

class PhysicsManager extends Component {
  update() {
    console.log(this.props.name + " updated");
  }
}

class Entity extends Component {
  update() {
    console.log(this.props.name + " entity updated");
  }
}

const physicsManager = new PhysicsManager({ name: "PHYSICS MANAGER" });
const playerEntity = new Entity({ name: "PLAYER" });
const nonPlayerEntity = new Entity({ name: "NON PLAYER" });

const entityManager = new EntityManager({
  name: "ENTITY MANAGER",
  children: [playerEntity, nonPlayerEntity],
});

const engine = new Engine({
  name: "ENGINE",
  children: [physicsManager, entityManager],
});

engine.update();
// PHYSICS MANAGER updated
// PLAYER entity updated
// NON PLAYER entity updated
~~~

Параметр `props` представляет собой литерал объекта, который мы используем для установки имени компонента и сохранения списка всех дочерних компонентов для составных объектов.

Метод `update` этого класса переопределен в каждом из наших подклассов.

Поскольку класс `PhysicsManager` и класс `Entity` являются простыми компонентами без дочерних элементов, их методы `update` должны явно определять их поведение.

### Decorator (Декоратор)

Согласно Википедии, _Decorator_ — структурный шаблон проектирования, предназначенный для динамического подключения дополнительного поведения к объекту.

Проще говоря, паттерн позволяет добавлять объектам новые функции с помощью обертки без создания отдельного класса.

Пример декоратора из жизни — это подключение мышки к ноутбуку, то есть вы добавляете новую функцию устройству, не меняя его. 

У этого паттерна широкая область применения. Им пользуются каждый раз, когда нужно добавить логику в уже созданные объекты и библиотеки, менять которые нельзя.

~~~
// создаем класс, с набором свойств и методов
class Car {
  constructor(cost) {
    this.cost = cost;
  }

  getCost() {
    return `Стоимость автомобиля: ${this.cost}`;
  }
}

// далее создаем декоратор, который принимает экземпляр класса
// и расширяет его, в данном случае добавляем свойство color
const colorCar = (car, color) => {
  car.color = color;
  return car;
};

const car = new Car(16000);

// оборачиваем инстанс car в декоратор, и получаем расширенный класс
colorCar(car, "orange");

console.log(car); //  Car { cost: 16000, color: 'orange' }
console.log(car.getCost()); // Стоимость автомобиля: 16000
~~~

### Facade (Фасад)

Согласно Википедии, _Facade_ — структурный шаблон проектирования, позволяющий скрыть сложность системы путём сведения всех возможных внешних вызовов к одному объекту, делегирующему их соответствующим объектам системы.

Facade предоставляет упрощенный интерфейс для сложной системы. 

Пример из жизни: при оплате покупки через Apple Pay вы подносите телефон к устройству и оплачиваете покупку. Кажется, что все просто. Но на самом деле внутри этого процесса происходит гораздо больше вещей. Этот упрощенный интерфейс называется фасадом.

Используется в библиотеках и позволяет описать их так, чтобы пользователю не нужно было вникать в их реализацию.

~~~
// Создаем классы, Doors и Body, которые отвечают за конфигурацию отдельных частей автомобиля
class Doors {
  getNumberOfDoors(type) {
    return type === "truck" ? 2 : 4;
  }
}

class Body {
  createBody(type) {
    return type === "truck";
  }
}

// Далее создаем класс Car, который и является нашим фасадом.
// В методе createCar мы создали экземпляры классов и вызвали нужные методы у них,
// чтобы получить готовую конфигурацию автомобиля.
// Таким образом мы оградили пользователя от нужды понимания всех классов и их взаимодействия.
// Взамен этого предложили удобный интерфейс.
class Car {
  createCar(type) {
    const doors = new Doors().getNumberOfDoors(type);
    const body = new Body().createBody(type);

    return { doors, body };
  }
}

console.log(new Car().createCar("truck")); // { doors: 2, body: true }
console.log(new Car().createCar("sport")); // { doors: 4, body: false }
~~~

### Flyweight (Легковес)

_Flyweight_ позволяет вместить бóльшее количество объектов в отведённую оперативную память. Легковес экономит память, разделяя общее состояние объектов между собой, вместо хранения одинаковых данных в каждом объекте.

~~~
class CarFlyweight {
  constructor(company, model, fuel) {
    this.company = company;
    this.model = model;
    this.fuel = fuel;
  }
}

class FlyweightFactory {
  constructor() {
    this.flyweights = {};
  }

  get(company, model, fuel) {
    const id = `${company}/${model}/${fuel}`;
    if (!this.flyweights[id]) {
      this.flyweights[id] = new CarFlyweight(company, model, fuel);
    }
    return this.flyweights[id];
  }

  getCount() {
    return Object.keys(this.flyweights).length;
  }

  getAll() {
    return this.flyweights;
  }
}

class Car {
  constructor(company, model, fuel, price, vin) {
    this.flyweight = flyWeightFactory.get(company, model, fuel);
    this.price = price;
    this.vin = vin;
  }
}

class CarsList {
  constructor() {
    this.cars = {};
  }

  add(company, model, fuel, price, vin) {
    this.cars[vin] = new Car(company, model, fuel, price, vin);
  }

  get(vin) {
    return this.cars[vin];
  }

  getAll() {
    return this.cars;
  }

  getCount() {
    return Object.keys(this.cars).length;
  }
}

const cars = new CarsList();
const flyWeightFactory = new FlyweightFactory();

cars.add("Ford", "Focus", "Gasoline", "20000", "5XYKT3A17BG157871");
cars.add("Ford", "Focus", "Gasoline", "40000", "JH4KA7660NC003110");
cars.add("Ford", "Focus", "Diesel", "27000", "JH4KA3140JC003021");
cars.add("Audi", "A4", "Diesel", "32000", "1B7GL22Z31S190315");

console.log(cars.getCount()); // 4

console.log(cars.getAll());
// {5XYKT3A17BG157871: Car, JH4KA7660NC003110: Car, JH4KA3140JC003021: Car, 1B7GL22Z31S190315: Car}

console.log(flyWeightFactory.getCount()); // 3

console.log(flyWeightFactory.getAll());
// {Ford/Focus/Gasoline: CarFlyweight, Ford/Focus/Diesel: CarFlyweight, Audi/A4/Diesel: CarFlyweight}

const car1 = cars.get("5XYKT3A17BG157871");
const car2 = cars.get("JH4KA7660NC003110");

console.log(car1.flyweight === car2.flyweight); // true
~~~

### Proxy (Заместитель)

_Proxy_ позволяет подставлять вместо реальных объектов специальные объекты-заменители. Эти объекты перехватывают вызовы к оригинальному объекту, позволяя сделать что-то до или после передачи вызова оригиналу.

~~~
// original object
const person = {
  firstName: 'Lena',
  lastName: 'Smith',
};

// use proxy add logic on person
const personProxy = new Proxy(person, {
  get: (target, prop) => {
    if(prop === 'fullName') {
      return `${target.firstName} ${target.lastName}`;
    }
    return target[prop];
  },
});

// throw the proxy object, we can get full name
console.log(personProxy.fullName); // "Lena Smith"
~~~

`get` имеет два параметра. Первый — это `target`, это исходный объект, поэтому в методе `get` мы можем получить доступ к исходному объекту через `target`. Второй — `prop`, это имя свойства, к которому мы хотим получить доступ.
