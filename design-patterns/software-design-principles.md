# ?Software design principles

### DRY

DRY — основополагающий принцип разработки. Он расшифровывается как Don’t repeat yourself — «не повторяйтесь».

Когда пишете код, важно думать о том, как можно переиспользовать тот или иной фрагмент, что можно выделить в универсальную функцию или класс, сделать модулем.

При этом речь не идёт о создании библиотек под каждую неодноразовую функцию — имеется в виду очень похожая логика, которая встречается в нескольких местах, которая, возможно, должна быть вынесена в функцию.

А если в нескольких местах определена одна и та же функция, то её можно вынести в общий модуль.

Ну и, наконец, если нужно часто использовать один и тот же модуль, вероятно, из него можно сделать библиотеку.

Этот принцип полезен всегда, вне зависимости от платформы или языка. Допустим, вам надо автоматизировать некое поведение. Чтобы не прописывать несколько раз одну и ту же логику и не раздувать код без нужды, можно попробовать обобщить её и вынести в отдельный элемент.

### KISS

Эта аббревиатура Keep it simple, stupid («Сделай это проще, тупица») или, есть вариант Keep it stupid simple («Пусть всё будет простым до безобразия»), который ещё лучше передаёт смысл аббревиатуры.

Решая какую-нибудь проблему, можно так увлечься, что сам не заметишь, как уже занялся оверинжинирингом. Задача в итоге, конечно, будет решена — но её можно было бы выполнить куда проще и изящнее.

В контексте программирования есть несколько моментов, на которые следует обратить внимание всякий раз, когда мы хотим уменьшить сложность.

* Убедитесь, что ваши имена переменных правильно описывают переменную, которую они содержат.
* Убедитесь, что имена ваших методов соответствуют назначению этого метода.
* Напишите комментарии в вашем методе, где это необходимо.
* Убедитесь, что ваши классы имеют единую ответственность.
* Насколько это возможно, избегайте глобальных состояний и поведения.
* Удалите экземпляры, методы или избыточные процессы в кодовой базе, которые не используются.

Почему принцип KISS важен в программировании?

* Принцип KISS обеспечивает непрерывность, когда это необходимо, и дает возможность другим людям понять процесс.
* Более простые процессы позволяют повысить эффективность автоматизированного тестирования. Простую систему легче протестировать, чем сложную.

Наконец, важно стремиться максимально уменьшить сложность, сделать наш процесс кодирования максимально прозрачным, эффективным и безопасным.

### YAGNI

Принцип, иначе известный как You ain’t gonna need it («Вам это не понадобится»), пришёл из экстремального программирования. Согласно ему создавать какую-то функциональность следует только тогда, когда она действительно нужна.

Дело в том, что в рамках Agile-методологий нужно фокусироваться только на текущей итерации проекта. Работать на опережение, добавляя в проект больше функциональности, чем требуется в данный момент, — не очень хорошая идея, учитывая, как быстро могут меняться планы.

Итерации в Agile довольно короткие — то есть вы будете получать какой-то фидбэк уже на ранних этапах разработки, и потенциально он может изменить направление всей работы над проектом. Так зачем вам тратить время на функцию, которая в итоге окажется совершенно ненужной?

Однако, если использовать методы каскадной разработки, где вся работа планируется заранее и все стараются чётко придерживаться плана, принцип YAGNI неприменим.

### SOLID

Принципы SOLID — это набор программных разработок, которые помогают разработчикам создавать надежные, удобные в сопровождении приложения при минимальных затратах на внесение изменений.

Хотя принципы SOLID часто используются в объектно-ориентированном программировании, мы можем использовать их и в других языках, таких как JavaScript.

Каковы принципы SOLID?

1. Single responsibility principle, принцип единственной ответственности
2. Open-closed principle, принцип открытости/закрытости
3. Liskov substitution principle, принцип подстановки Лисков
4. Interface segregation principle, принцип разделения интерфейса
5. Dependency inversion principle, принцип инверсии зависимостей

#### Single responsibility principle

Каждый класс должен иметь одну ответственность (функционал) и все его методы должны быть связаны с этой ответственностью.

Рассмотрим плохой пример.

~~~
class ConcertLineups {
  constructor() {
    this.lineups = [];
  }

  addBandsToLineup(bandName) {
    this.lineups.push(bandName);
  }

  displayLineups() {
    console.log(this.lineups);
  }
}

const concertLineups = new ConcertLineups();
concertLineups.addBandsToLineup("Warfaze");
concertLineups.addBandsToLineup("Karnival");
concertLineups.addBandsToLineup("SBC");
concertLineups.displayLineups(); // ['Warfaze', 'Karnival', 'SBC']
~~~

Лучший подход с использованием принципов единственной ответственности:

~~~
class Logger {
  log(message) {
    console.log(message);
  }
}

class ConcertLineups {
  constructor() {
    this.lineups = [];
    this.logger = new Logger();
  }

  addBandsToLineup(bandName) {
    this.lineups.push(bandName);
  }

  displayLineups() {
    this.logger.log(this.lineups);
  }
}

const concertLineups = new ConcertLineups();
concertLineups.addBandsToLineup("Warfaze");
concertLineups.addBandsToLineup("Karnival");
concertLineups.addBandsToLineup("SBC");
concertLineups.displayLineups(); // ['Warfaze', 'Karnival', 'SBC']
~~~

Здесь мы выделяем логирование в отдельный модуль.

#### Open-closed principle

Принцип открытости-закрытости гласит, что программные объекты (классы, модули, функции и т. д.) должны быть открыты для расширения, но закрыты для модификации.

Первый пример принципа открытости-закрытости, — это класс, использующий `switch` или несколько операторов `if`. В подобном коде существует вполне реальная возможность изменить класс с помощью операторов `switch` или `if`. И это нарушает принцип открытого-закрытого процесса.

~~~
class Vehicle {
  constructor(fuelCapacity, fuelEfficiency) {
    this.fuelCapacity = fuelCapacity;
    this.fuelEfficiency = fuelEfficiency;
  }

  getRange() {
    let range = this.fuelCapacity * this.fuelEfficiency;

    if (this instanceof HybridVehicle) {
      range += this.electricRange;
    }
    return range;
  }
}

class HybridVehicle extends Vehicle {
  constructor(fuelCapacity, fuelEfficiency, electricRange) {
    super(fuelCapacity, fuelEfficiency);
    this.electricRange = electricRange;
  }
}

const standardVehicle = new Vehicle(10, 15);
const hybridVehicle = new HybridVehicle(10, 15, 50);

console.log(standardVehicle.getRange()); // 150
console.log(hybridVehicle.getRange()); // 200
~~~

Это нарушает принцип открытости-закрытости, потому что при добавлении нашего нового класса `HybridVehicle` нам пришлось вернуться и изменить код нашего класса `Vehicle`, чтобы заставить его работать.

В дальнейшем каждый раз, когда мы добавляем новый тип транспортного средства, у которого могут быть другие параметры для своего диапазона, нам придется постоянно изменять эту существующую функцию `getRange`.

Вместо этого мы могли бы переопределить метод `getRange` в классе `HybridVehicle`, выдав правильный вывод для обоих типов транспортных средств, не изменяя при этом исходный код:

~~~
class Vehicle {
  constructor(fuelCapacity, fuelEfficiency) {
    this.fuelCapacity = fuelCapacity;
    this.fuelEfficiency = fuelEfficiency;
  }

  getRange() {
    return this.fuelCapacity * this.fuelEfficiency;
  }
}

class HybridVehicle extends Vehicle {
  constructor(fuelCapacity, fuelEfficiency, electricRange) {
    super(fuelCapacity, fuelEfficiency);
    this.electricRange = electricRange;
  }

  getRange() {
    return this.fuelCapacity * this.fuelEfficiency + this.electricRange;
  }
}

const standardVehicle = new Vehicle(10, 15);
const hybridVehicle = new HybridVehicle(10, 15, 50);

console.log(standardVehicle.getRange()); // 150
console.log(hybridVehicle.getRange()); // 200
~~~

Вот еще один пример, в котором вместо классов JavaScript используются функции:

~~~
function calculatePrice(price, discount) {
  if (discount === "10%") {
    return price * 0.9;
  } else if (discount === "20%") {
    return price * 0.8;
  } else if (discount === "30%") {
    return price * 0.7;
  } else {
    throw new Error("Invalid discount");
  }
}

const discountedPrice = calculatePrice(100, "10%");
console.log(`Your discounted price is $${discountedPrice}`); // Your discounted price is $90
~~~

Чтобы исправить это, вы можете извлечь все свои скидки в объект и использовать его в функции следующим образом:

~~~
const discounts = {
  "10%": 0.9,
  "20%": 0.8,
  "30%": 0.7,
};

function calculatePrice(price, discountType) {
  const discount = discounts[discountType];
  if (discount === undefined) {
    throw new Error("Invalid discount");
  }
  return price * discount;
}

const discountedPrice = calculatePrice(100, "30%");
console.log(`Your discounted price is $${discountedPrice}`); // Your discounted price is $70
~~~

### Liskov substitution principle

Принцип определяет, что при наследовании объекты суперкласса (или родительского класса) должны заменяться объектами его подклассов (или дочернего класса) без нарушения работы приложения или возникновения какой-либо ошибки.

Проще говоря, вы хотите, чтобы объекты ваших подклассов вели себя так же, как объекты вашего суперкласса.

Очень распространенным примером является сценарий «прямоугольник-квадрат».

Ясно, что все квадраты являются прямоугольниками, потому что они четырехугольники, у которых все четыре угла прямые. Но не всякий прямоугольник является квадратом. Чтобы быть квадратом, его стороны должны иметь одинаковую длину.

Имея это в виду, предположим, что у вас есть класс прямоугольника для вычисления площади прямоугольника и выполнения других операций, таких как установка цвета. Прекрасно зная, что все квадраты являются прямоугольниками, вы можете наследовать свойства прямоугольника:

~~~
class Rectangle {
  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  setColor(color) {
    this.color = color;
  }

  getColor() {
    return this.color;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width) {
    this.width = width;
    this.height = width;
  }

  setHeight(height) {
    this.width = height;
    this.height = height;
  }
}

const rectangle = new Rectangle();
rectangle.setWidth(10);
rectangle.setHeight(5);
console.log(rectangle.getArea()); // 50

const square = new Square();
square.setWidth(10);
console.log(square.getArea()); // 100
square.setHeight(5);
console.log(square.getArea()); // 25
~~~

Взгляните на пример, он должен работать правильно.

Но согласно LSP вы хотите, чтобы объекты ваших подклассов вели себя так же, как объекты вашего суперкласса.

Этот пример нарушает LSP. Чтобы исправить это, должен быть общий класс для всех фигур, который будет содержать все общие методы, к которым вы хотите, чтобы объекты ваших подклассов имели доступ.

~~~
class Shape {
  setColor(color) {
    this.color = color;
  }

  getColor() {
    return this.color;
  }
}

class Rectangle extends Shape {
  setWidth(width) {
    this.width = width;
  }

  setHeight(height) {
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }
}

class Square extends Shape {
  setSide(side) {
    this.side = side;
  }

  getArea() {
    return this.side * this.side;
  }
}

const rectangle = new Rectangle();
rectangle.setWidth(10);
rectangle.setHeight(5);
console.log(rectangle.getArea()); // 50

const square = new Square();
square.setSide(10);
console.log(square.getArea()); // 100
square.setSide(5);
console.log(square.getArea()); // 25
~~~

#### Interface segregation principle

Принцип разделения интерфейсов (ISP) гласит, что «клиент никогда не должен быть вынужден реализовывать интерфейс, который он не использует, или клиенты не должны зависеть от методов, которые они не используют».

Целью этого принципа является удаление ненужного кода из классов, чтобы уменьшить количество непредвиденных ошибок, когда класс не может выполнить действие. Мы разделяем действия на более мелкие наборы, чтобы класс выполнял только те действия, которые ему требуются.

Проясним концепцию с помощью примера.

Допустим, у нас есть 2 типа пользователей: бесплатные пользователи и премиум-пользователи.

Премиум-пользователи могут смотреть как трейлер, так и фильм. Однако бесплатные пользователи могут смотреть только трейлер.

~~~
class User {
  constructor(name) {
    this.name = name;
  }

  watchTrailer() {
    console.log("Watching trailer...");
  }

  watchMovie() {
    console.log("Watching movie...");
  }
}

class PremiumUser extends User {}

class FreeUser extends User {
  watchMovie() {
    return null;
  }
}

const premiumUser = new PremiumUser("Alex");
premiumUser.watchTrailer(); // Watching trailer...
premiumUser.watchMovie(); // Watching movie...

const freeUser = new FreeUser("Mary");
freeUser.watchTrailer(); // Watching trailer...
freeUser.watchMovie();
~~~

Внутри `FreeUser` мы переопределяем метод `watchMovie` , и здесь он возвращает `null`, что означает, что он ничего не делает.

Что не так с этим решением?

Мы по-прежнему можем вызывать этот метод, но он не делает ничего значимого.

И здесь мы нарушаем принцип разделения интерфейса, потому что наш класс `FreeUser` зависит от метода, который он не использует.

Решение:

~~~
class User {
  constructor(name) {
    this.name = name;
  }

  watchTrailer() {
    console.log("Watching trailer...");
  }
}

class PremiumUser extends User {
  watchMovie() {
    console.log("Watching movie...");
  }
}

class FreeUser extends User {}

const premiumUser = new PremiumUser("Alex");
premiumUser.watchTrailer(); // Watching trailer...
premiumUser.watchMovie(); // Watching movie...

const freeUser = new FreeUser("Mary");
freeUser.watchTrailer(); // Watching trailer...
freeUser.watchMovie(); // TypeError: freeUser.watchMovie is not a function
~~~

#### Dependency inversion principle

Принцип инверсии зависимостей гласит, что сущности должны зависеть от абстракций. Модули высокого уровня не должны зависеть от модулей низкого уровня.

Когда мы говорим о модулях высокого уровня, мы имеем в виду класс, который выполняет действие, реализующее инструмент или библиотеку, а когда мы говорим о модулях низкого уровня, мы имеем в виду инструменты или библиотеки, необходимые для выполнения действия.

~~~
class Checkout {
  constructor() {
    this.paymentProcessor = new Stripe("USD");
  }

  makePayment(amount) {
    this.paymentProcessor.createTransaction(amount * 100);
  }
}

class Stripe {
  constructor(currency) {
    this.currency = currency;
  }

  createTransaction(amount) {
    console.log(`Payment made for $${amount / 100}`);
  }
}

const checkout = new Checkout();
checkout.makePayment(200); // Payment made for $200
~~~

Обратите внимание, что мы создали зависимость между нашим классом `Checkout` (модуль высокого уровня) и `Stripe` (модуль низкого уровня), нарушая принцип инверсии зависимостей. Зависимость особенно заметна, когда мы конвертируем сумму в центы.

`Checkout` не должен заботиться о том, какой платежный процессор используется, он должен заботится только о совершении транзакции.

Чтобы разделить эти два модуля, нужно реализовать посредника между `Checkout` и платежным процессором, создав абстракцию, чтобы независимо от того, какой платежный процессор мы используем, класс Checkout всегда работал с одними и теми же вызовами методов.

Новый класс `PaymentProcessor` будет отвечать за адаптацию всего к используемому платежному процессору (в данном случае `Stripe`).

Промежуточный класс будет иметь следующий код:

~~~
class PaymentProcessor {
  constructor(processor, currency) {
    this.processor = processor;
    this.currency = currency;
  }

  createPaymentIntent(amount) {
    const amountInCents = amount * 100;
    this.processor.createTransaction(amountInCents);
  }
}

class Checkout {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
  }

  makePayment(amount) {
    this.paymentProcessor.createPaymentIntent(amount);
  }
}

class Stripe {
  constructor(currency) {
    this.currency = currency;
  }

  createTransaction(amount) {
    console.log(`Payment made for $${amount / 100}`);
  }
}

const paymentProcessor = new PaymentProcessor(new Stripe("USD"), "USD");
const checkout = new Checkout(paymentProcessor);
checkout.makePayment(200); // Payment made for $200
~~~

Представьте, что теперь нас просят заменить `Stripe` другим платежным процессором, который не требует конвертации суммы в центы, но при каждой транзакции запрашивает валюту, которая будет использоваться. Результирующий код будет следующим:

~~~
class PaymentProcessor {
  constructor(processor, currency) {
    this.processor = processor;
    this.currency = currency;
  }

  createPaymentIntent(amount) {
    this.processor.createTransaction(amount, this.currency);
  }
}

class Checkout {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
  }

  makePayment(amount) {
    this.paymentProcessor.createPaymentIntent(amount);
  }
}

class BetterProcessor {
  createTransaction(amount, currency) {
    console.log(`Payment made for ${amount} ${currency}`);
  }
}

const paymentProcessor = new PaymentProcessor(new BetterProcessor(), "USD");
const checkout = new Checkout(paymentProcessor);
checkout.makePayment(200); // Payment made for 200 USD
~~~

Мы изменили только процессор, переданный классу `PaymentProcessor`, а класс `Checkout` остался нетронутым. Мы адаптировали промежуточный класс `PaymentProcessor` к потребностям процессора.
