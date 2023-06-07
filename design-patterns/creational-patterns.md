# ?Creational Patterns

Согласно Википедии, _creational patterns_ (порождающие шаблоны) — шаблоны проектирования, которые позволяют сделать систему независимой от способа создания, композиции и представления объектов.

Проще говоря, порождающие паттерны предназначены для создания экземпляра объекта или группы связанных объектов. К ним относятся:

* Abstract Factory
* Builder
* Factory Method
* Prototype
* Singleton

По мнению разработчиков MediaSoft Singleton, Builder и Factory Method — это самые используемые порождающие паттерны в разработке.

### Abstract Factory (Абстрактная фабрика)

_Abstract Factory_ можно использовать для определения конкретных экземпляров или классов без указания точного объекта, который создается.

~~~
class Car {
  constructor() {
    this.name = "Car";
    this.wheels = 4;
  }
}

class Tractor {
  constructor() {
    this.name = "Tractor";
    this.wheels = 4;
  }
}

class Bike {
  constructor() {
    this.name = "Bike";
    this.wheels = 2;
  }
}

const vehicleFactory = {
  createVehicle: function (type) {
    switch (type.toLowerCase()) {
      case "car":
        return new Car();
      case "tractor":
        return new Tractor();
      case "bike":
        return new Bike();
      default:
        return null;
    }
  },
};

const car = vehicleFactory.createVehicle("Car");
console.log(car); // Car { name: "Car", wheels: 4 }
~~~

### Builder (Строитель)

_Builder_ позволяет создавать разные объекты с заданным состоянием, используя один и тот же код.

Пример этого паттерна в жизни — покупка компьютера в магазине. При выборе мы указываем, какими характеристиками должна обладать техника (например, память 16 ГБ, процессор Intel core i7 и так далее). Таким образом, мы можем создавать различные виды одного объекта.

Пригодится при составлении SQL-запроса, а также в юнит-тестах.

~~~
class Apartment {
  constructor(options) {
    for (const option in options) {
      this[option] = options[option];
    }
  }
  
  getOptions() {
    return `Количество комнат: ${this.roomsNumber}, площадь: ${this.square}, этаж: ${this.floor}`;
  }
}

class ApartmentBuilder {
  setRoomsNumber(roomsNumber) {
    this.roomsNumber = roomsNumber;
    return this;
  }

  setFloor(floor) {
    this.floor = floor;
    return this;
  }

  setSquare(square) {
    this.square = square;
    return this;
  }

  build() {
    return new Apartment(this);
  }
}

const bigApartment = new ApartmentBuilder()
  .setFloor(10)
  .setRoomsNumber(5)
  .setSquare(120)
  .build();

console.log(bigApartment.getOptions()); // Количество комнат: 5, площадь: 120, этаж: 10
~~~

Ключевая идея паттерна в том, чтобы вынести сложную логику создания объекта в отдельный класс «Строитель».

Этот класс позволяет создавать сложные объекты поэтапно, на выходе получая готовый объект с нужными нам характеристиками и предоставляя классу упрощенный интерфейс для работы.

Несмотря на удобство в использовании, сам класс строителя в некоторых случаях будет тяжеловесным, усложняя код программы и, при определенных обстоятельствах, будет излишним. В таких случаях будет разумно использовать отдельный конструктор/инициализатор.

### Factory Method (Фабричный метод)

Паттерн позволяет использовать один класс для создания объектов разной реализации интерфейса и делегировать логику дочерним классам.

При заказе авиабилета мы указываем только информацию из паспорта, номер рейса и кресла. Такие данные, как номер терминала, время вылета, модель самолета инициализируются без нашего участия. Это экономит время пассажиров и сокращает количество ошибок.

Паттерн подходит для ситуаций, когда нам необходимо выполнить действие (например, отправить запрос), но заранее неизвестны все данные, так как они зависят от входных параметров (например, от протокола передачи данных — rest, soap или socket).

~~~
class Apartment {
  constructor(roomsNumber, square, floor) {
    this.roomsNumber = roomsNumber;
    this.floor = floor;
    this.square = square;
  }

  getOptions() {
    return `Количество комнат: ${this.roomsNumber}, площадь: ${this.square}, этаж: ${this.floor}`;
  }
};

class ApartmentFactory {
  createApartament(type, floor) {
    switch(type) {
      case('oneRoom'):
        return new Apartment(1, 35, floor);
      case('twoRooms'):
        return new Apartment(2, 55, floor);
      case('threeRooms'):
        return new Apartment(3, 75, floor);
      default:
        throw new Error('Такой квартиры не найдено');
    }
  }
}

const oneRoomAparnament = new ApartmentFactory().createApartament('oneRoom', 10);

console.log(oneRoomAparnament.getOptions()); // Количество комнат: 1, площадь: 35, этаж: 10
~~~

### Prototype (Прототип)

_Prototype_ фокусируется на создании объекта, который можно использовать в качестве схемы для других объектов посредством прототипного наследования.

С этим шаблоном легко работать в JavaScript из-за встроенной поддержки прототипного наследования в JS.

~~~
class Welcome {
  constructor(name) {
    this.name = name;
  }
}

Welcome.prototype.sayHello = function () {
  return "Hello, " + this.name + "!";
};

const welcome = new Welcome("Shyam");
const output = welcome.sayHello();
console.log(output); // Hello, Shyam!
~~~

### Singleton (Одиночка)

_Singleton_ гарантирует, что созданный объект будет единственным в приложении и позволяет получать к нему доступ из любой части приложения.

Хороший пример этого паттерна из жизни — это классный журнал в школе. У каждого класса он только один, и если учитель спросит журнал, то всегда получит один и тот же экземпляр.

Пригодится, если в приложении есть управляющий объект, в котором хранится весь контекст приложения. Например, в сервисах хранения данных.

~~~
// создаем класс Singleton, который проверяет имеется ли у него свойство instance
// если нет, тогда создается новый экземпляр, но если свойство есть
// тогда мы возвращаем ранее созданный экземпляр класса
class Singleton {
  constructor(data) {
    if (Singleton.instance) {
      return Singleton.instance;
    }

    Singleton.instance = this;
    this.data = data;
  }

  consoleData() {
    console.log(this.data);
  }
}

// в работе Singleton мы можем убедиться создав 2 экземпляра класса
const firstSingleton = new Singleton("firstSingleton");
const secondSingleton = new Singleton("secondSingleton");

// в обоих случаях в консоли выведется одинаковое сообщение
firstSingleton.consoleData(); //firstSingleton
secondSingleton.consoleData(); //firstSingleton
~~~

Есть мнение, что одиночка является анти-паттерном — его противники утверждают, что одиночка привносит глобальное состояние в приложение и трудно тестируется.

Несмотря на то, что в этом есть своя правда, в действительности же, этот паттерн не вызывает проблем, если помнить о следующем: суть одиночки заключается в том, чтобы в определенный момент выполнения программы обеспечить наличие одного и только одного экземпляра определенного класса.
