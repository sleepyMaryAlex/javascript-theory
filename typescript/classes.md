# ?Classes

TypeScript предлагает полную поддержку ключевого слова `class`, представленного в ES2015.

Как и в других функциях языка JavaScript, TypeScript добавляет аннотации типов и другой синтаксис, позволяющий вам выражать отношения между классами и другими типами.

### Члены класса

#### Поля

Объявление поля создает общедоступное записываемое свойство в классе:

~~~
class Point {
  x: number;
  y: number;
}
 
const pt = new Point();
pt.x = 0;
pt.y = 0;
~~~

Как и в других местах, аннотация типа является необязательной, но будет неявным типом `any`, если не указана.

Поля также могут иметь инициализаторы. Инициализатор свойства класса будет использоваться для определения его типа:

~~~
class Point {
  x = 0;
  y = 0;
}
 
const pt = new Point();
pt.x = "0"; // Error! Type 'string' is not assignable to type 'number'.
~~~

#### `--strictPropertyInitialization`

Параметр `strictPropertyInitialization` определяет, нужно ли инициализировать поля класса в конструкторе.

~~~
class BadGreeter {
  name: string;
  // Property 'name' has no initializer and is not definitely assigned in the constructor.
}
~~~

Обратите внимание, что поле необходимо инициализировать в самом конструкторе.

~~~
class GoodGreeter {
  name: string;

  constructor() {
    this.name = "hello";
  }
}
~~~

Если вы намерены определенно инициализировать поле с помощью средств, отличных от конструктора (например, может быть, внешняя библиотека заполняет часть вашего класса для вас), вы можете использовать оператор утверждения определенного присваивания `!`:

~~~
class OKGreeter {
  // Not initialized, but no error
  name!: string;
}
~~~

#### `readonly`

~~~
class Greeter {
  readonly name: string = "world";

  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }

  err() {
    this.name = "not ok";
    // Cannot assign to 'name' because it is a read-only property.
  }
}

const g = new Greeter();
g.name = "also not ok";
// Cannot assign to 'name' because it is a read-only property.
~~~

#### Конструкторы

Конструкторы классов очень похожи на функции. Вы можете добавлять параметры с аннотациями типов, значениями по умолчанию и перегрузками:

~~~
class Point {
  x: number;

  // Normal signature with defaults
  constructor(x = 0) {
    this.x = x;
  }
}
~~~

~~~
class Point {
  // Overloads
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
  }
}
~~~

#### `Super`

Как и в JavaScript, если у вас есть базовый класс, вам нужно вызвать `super();` в вашем теле конструктора перед использованием любого `this`:

~~~
class Base {
  k = 4;
}

class Derived extends Base {
  constructor() {
    // Prints a wrong value in ES5; throws exception in ES6
    console.log(this.k);
    // 'super' must be called before accessing 'this' in the constructor of a derived class.
    super();
  }
}
~~~

#### Методы

Методы могут использовать все аннотации того же типа, что и функции и конструкторы:

~~~
class Point {
  x = 10;
  y = 10;

  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
~~~

#### Геттеры/сеттеры

TypeScript имеет некоторые специальные правила вывода для методов доступа:

* Если `get` существует, но нет `set`, свойство автоматически `readonly`
* Если тип параметра сеттера не указан, он выводится из типа возвращаемого значения геттера.
* Геттеры и сеттеры должны иметь одинаковую видимость членов.

Начиная с TypeScript 4.3, можно использовать методы доступа с разными типами для получения и установки.

~~~
class Thing {
  _size = 0;

  get size(): number {
    return this._size;
  }

  set size(value: string | number | boolean) {
    let num = Number(value);

    // Don't allow NaN, Infinity, etc
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }

    this._size = num;
  }
}
~~~

#### Подписи индекса

~~~
class MyClass {
  [s: string]: boolean | ((s: string) => boolean);
 
  check(s: string) {
    return this[s] as boolean;
  }
}
~~~

Как правило, индексированные данные лучше хранить в другом месте, а не в самом экземпляре класса.

### Class heritage

#### `implements`

Вы можете использовать `implements`, чтобы проверить, удовлетворяет ли класс определенному `interface`. Будет выдано сообщение об ошибке, если класс не сможет правильно его реализовать:

~~~
interface Pingable {
  ping(): void;
}

class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}

class Ball implements Pingable {
  // Class 'Ball' incorrectly implements interface 'Pingable'.
  // Property 'ping' is missing in type 'Ball' but required in type 'Pingable'.
  pong() {
    console.log("pong!");
  }
}
~~~

Классы также могут реализовывать несколько интерфейсов, например `class C implements A, B`.

Важно понимать, что `implements` — это всего лишь проверка того, что класс можно рассматривать как тип интерфейса. Он вообще не меняет тип класса или его методы.

Распространенным источником ошибки является предположение, что `implements` изменит тип класса, но это не так!

~~~
interface Checkable {
  check(name: string): boolean;
}

class NameChecker implements Checkable {
  check(s) {
    // Parameter 's' implicitly has an 'any' type.
    // Notice no error here
    return s.isFinite() === "ok";
  }
}
~~~

#### `extends`

Классы могут быть `extend` из базового класса. Производный класс имеет все свойства и методы своего базового класса, а также может определять дополнительные члены.

~~~
class Animal {
  move() {
    console.log("Moving along!");
  }
}

class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}

const d = new Dog();
// Base class method
d.move();
// Derived class method
d.woof(3);
~~~

#### Overriding Methods

Производный класс также может переопределять поле или свойство базового класса:

~~~
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}

const d = new Derived();
d.greet(); // Hello, world!
d.greet("reader"); // Hello, READER
~~~

Важно, чтобы производный класс следовал контракту базового класса.

Что, если `Derived` не будет следовать контракту `Base`?

~~~
class Base {
  greet() {
    console.log("Hello, world!");
  }
}

class Derived extends Base {
  // Make this parameter required
  greet(name: string) {
    // Property 'greet' in type 'Derived' is not assignable to the same property in base type 'Base'.
    // Type '(name: string) => void' is not assignable to type '() => void'.
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
~~~

#### Type-only Field Declarations

Когда `target >= ES2022` или `useDefineForClassFields` имеет значение `true`, поля класса инициализируются после завершения конструктора родительского класса, перезаписывая любое значение, установленное родительским классом. Это может быть проблемой, когда вы хотите только повторно объявить более точный тип для унаследованного поля. Чтобы обработать эти случаи, вы можете написать `declare`, чтобы указать TypeScript, что для этого объявления поля не должно быть никакого эффекта во время выполнения.

~~~
interface Animal {
  dateOfBirth: any;
}

interface Dog extends Animal {
  breed: any;
}

class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}

class DogHouse extends AnimalHouse {
  // Does not emit JavaScript code,
  // only ensures the types are correct
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
~~~

### Member Visibility

Вы можете использовать TypeScript для управления видимостью определенных методов или свойств для кода вне класса.

#### `public`

Член `public` может быть доступен в любом месте:

~~~
class Greeter {
  public greet() {
    console.log("hi!");
  }
}

const g = new Greeter();
g.greet();
~~~

Поскольку `public` это уже модификатор видимости по умолчанию, вам никогда не нужно писать его для члена класса, но вы можете сделать это из соображений стиля/удобочитаемости.

#### `protected`

Члены `protected` видны только подклассам того класса, в котором они объявлены.

~~~
class Greeter {
  protected getName() {
    return "hi";
  }
}

class SpecialGreeter extends Greeter {
  public howdy() {
    // OK to access protected member here
    console.log("Howdy, " + this.getName());
  }
}

const g = new SpecialGreeter();
g.getName();
// Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
~~~

Важно отметить, что в производном классе нам нужно быть осторожными, чтобы повторить защищенный модификатор, если это не является преднамеренным.

~~~
class Base {
  protected m = 10;
}
class Derived extends Base {
  // No modifier, so default is 'public'
  m = 15;
}

const d = new Derived();
console.log(d.m); // OK
~~~

Различные языки ООП расходятся во мнениях относительно того, разрешено ли обращаться к члену `protected` через ссылку на базовый класс:

~~~
class Base {
  protected x: number = 1;
}

class Derived1 extends Base {
  protected x: number = 5;
}

class Derived2 extends Base {
  f1(other: Derived2) {
    other.x = 10;
  }

  f2(other: Base) {
    other.x = 10;
    // Property 'x' is protected and only accessible through an instance of class 'Derived2'. This is an instance of class 'Base'.
  }
}
~~~

Здесь TypeScript выступает на стороне C# и C++, потому что доступ к `x` в `Derived2` должен быть разрешен только из подклассов `Derived2`.

#### `private`

`private` похож на `protected`, но не разрешает доступ к члену даже из подклассов:

~~~
class Base {
  private x = 0;
}

class Derived extends Base {
  showX() {
    // Can't access in subclasses
    console.log(this.x);
    // Property 'x' is private and only accessible within class 'Base'.
  }
}
~~~

Различные языки ООП расходятся во мнениях относительно того, могут ли разные экземпляры одного и того же класса обращаться к `private` членам друг друга.

TypeScript разрешает частный доступ между экземплярами:

~~~
class A {
  private x = 10;

  public sameAs(other: A) {
    // No error
    return other.x === this.x;
  }
}
~~~

#### Предостережения

Как и другие аспекты системы типов TypeScript, `private` или `protected` применяются только во время проверки типов.

Это означает, что конструкции среды выполнения JavaScript по-прежнему могут обращаться к члену `private` или `protected`:

~~~
class MySafe {
  private secretKey = 12345;
}

// In a JavaScript file...
const s = new MySafe();
// Will print 12345
console.log(s.secretKey);
~~~

`private` также разрешает доступ с использованием записи в квадратных скобках во время проверки типа. Это упрощает доступ к объявленным приватным полям для таких вещей, как модульные тесты, с тем недостатком, что эти поля являются мягкими приватными и строго не обеспечивают конфиденциальность.

~~~
class MySafe {
  private secretKey = 12345;
}

const s = new MySafe();
console.log(s["secretKey"]); // OK
~~~

В отличие от приватных полей TypeScript, приватные поля JavaScript (`#`) остаются приватными после компиляции и не предоставляют ранее упомянутых обходных путей, таких как доступ к скобочным обозначениям, что делает их строго приватными.

### Static Members

Классы могут иметь членов `static`. Эти члены не связаны с конкретным экземпляром класса. Доступ к ним можно получить через сам объект конструктора класса:

~~~
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}

console.log(MyClass.x);
MyClass.printX();
~~~

Статические элементы также могут использовать те же модификаторы видимости `public`, `protected` и `private`.

~~~
class MyClass {
  private static x = 0;
}

console.log(MyClass.x);
// Property 'x' is private and only accessible within class 'MyClass'.
~~~

Статические члены также наследуются:

~~~
class Base {
  static getGreeting() {
    return "Hello world";
  }
}

class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
~~~

Свойства функций, такие как `name`, `length` и `call` недопустимы для определения в качестве членов `static`.

### `static` blocks in classes

Статические блоки позволяют вам написать последовательность операторов с их собственной областью действия, которые могут получить доступ к закрытым полям в содержащем классе. Это означает, что мы можем написать код инициализации со всеми возможностями написания выражений, без утечки переменных и с полным доступом к внутренностям нашего класса.

~~~
class Foo {
  static #count = 0;

  get count() {
    return Foo.#count;
  }

  static {
    try {
      const lastInstances = [];
      Foo.#count += lastInstances.length;
    } catch {}
  }
}
~~~

### Generic classes

Классы могут использовать общие ограничения и значения по умолчанию так же, как и интерфейсы.

~~~
class Box<Type> {
  contents: Type;
  constructor(value: Type) {
    this.contents = value;
  }
}

const b = new Box("hello!");
// const b: Box<string>;
~~~

Этот код является незаконным, и может быть неясно, почему:

~~~
class Box<Type> {
  static defaultValue: Type;
  // Static members cannot reference class type parameters.
}
~~~

Помните, что типы всегда полностью стираются!

Статические члены `generic` класса никогда не могут ссылаться на параметры типа класса.

### `this` at runtime in classes

Важно помнить, что TypeScript не меняет поведение JavaScript во время выполнения, и что JavaScript несколько известен своим своеобразным поведением во время выполнения.

~~~
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName,
};

// Prints "obj", not "MyClass"
console.log(obj.getName());
~~~

Короче говоря, по умолчанию значение `this` внутри функции зависит от того, как функция была вызвана.

Это редко то, что мы хотим! TypeScript предоставляет несколько способов смягчить или предотвратить такого рода ошибки.

#### Стрелочные функции

Если у вас есть функция, которая часто будет вызываться способом, теряющим свой `this` контекст, может иметь смысл использовать свойство функции стрелки вместо определения метода:

~~~
class MyClass {
  name = "MyClass";
  getName = () => {
    return this.name;
  };
}
const c = new MyClass();
const g = c.getName;
// Prints "MyClass" instead of crashing
console.log(g());
~~~

#### `this` параметры

TypeScript проверяет, что вызов функции с параметром `this` выполняется в правильном контексте. Вместо использования стрелочной функции мы можем добавить параметр `this` к определениям методов, чтобы статически обеспечивать правильность вызова метода:

~~~
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}

const c = new MyClass();
// OK
c.getName(); // this === MyClass
 
// Error, would crash
const g = c.getName;
console.log(g()); // this === undefined
// The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
~~~

### `this` types

В классах специальный тип, называемый `this`, динамически ссылается на тип текущего класса:

~~~
class Box {
  contents: string = "";
  set(value: string) {
    // (method) Box.set(value: string): this

    this.contents = value;
    return this;
  }
}
~~~

Здесь TypeScript предположил, что возвращаемый тип set будет `this`, а не `Box`.

Теперь давайте создадим подкласс `Box`:

~~~
class Box {
  contents: string = "";
  set(value: string) {
    this.contents = value;
    return this;
  }
}

class ClearableBox extends Box {
  clear() {
    this.contents = "";
  }
}
 
const a = new ClearableBox();
const b = a.set("hello");
// const b: ClearableBox;
~~~

Вы также можете использовать `this` в аннотации типа параметра.

Это отличается от написания `other: Box` — если у вас есть производный класс, его метод `sameAs` теперь будет принимать только другие экземпляры того же производного класса:

~~~
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}

class DerivedBox extends Box {
  otherContent: string = "?";
}

const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
// Argument of type 'Box' is not assignable to parameter of type 'DerivedBox'.
// Property 'otherContent' is missing in type 'Box' but required in type 'DerivedBox'.
~~~

#### `this`-based type guards

Вы можете использовать `this is Type` в позиции `return` для методов в классах и интерфейсах. При смешивании с сужением типа (например, операторы `if`) тип целевого объекта будет сужен до указанного типа.

~~~
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");
console.log(fso); // FileRep { path: 'foo/bar.txt', networked: false, content: 'foo' }
if (fso.isFile()) {
  fso.content;
  // const fso: FileRep
  console.log("content " + fso.content); // content foo
} else if (fso.isDirectory()) {
  fso.children;
  // const fso: Directory
  console.log("children " + fso.children);
} else if (fso.isNetworked()) {
  fso.host;
  // const fso: Networked & FileSystemObject
  console.log("host " + fso.host);
}
~~~

Обычный вариант использования защиты типа на основе `this` — разрешить ленивую проверку определенного поля. Например, в этом случае `undefined` удаляется из значения, хранящегося внутри блока, когда `hasValue` проверено на `true`:

~~~
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box();
box.value = "Gameboy";

box.value;
// (property) Box<unknown>.value?: unknown

if (box.hasValue()) {
  box.value;
  // (property) value: unknown
}

~~~

### Parameter Properties

TypeScript предлагает специальный синтаксис для превращения параметра конструктора в свойство класса с тем же именем и значением. Они называются свойствами параметров и создаются путем добавления к аргументу конструктора префикса одного из модификаторов видимости `public`, `private`, `protected` или `readonly`:

~~~
class Params {
  constructor(
    public readonly x: number,
    protected y: number,
    private z: number
  ) {
    // No body necessary
  }
}
const a = new Params(1, 2, 3);
console.log(a.x);
// (property) Params.x: number

console.log(a.z);
// Property 'z' is private and only accessible within class 'Params'.
~~~

### Class expressions

Выражения класса очень похожи на объявления классов. Единственная реальная разница в том, что выражениям класса не нужно имя, хотя мы можем ссылаться на них через любой идентификатор, к которому они в конечном итоге привязаны:

~~~
const someClass = class<Type> {
  content: Type;
  constructor(value: Type) {
    this.content = value;
  }
};

const m = new someClass("Hello, world");
// const m: someClass<string>;
~~~

### `abstract` classes and members

Классы, методы и поля в TypeScript могут быть абстрактными.

Абстрактный метод или абстрактное поле — это метод, который не имеет реализации. Эти члены должны существовать внутри абстрактного класса , экземпляр которого нельзя создать напрямую.

Роль абстрактных классов состоит в том, чтобы служить базовым классом для подклассов, которые реализуют все абстрактные члены. Когда класс не имеет абстрактных членов, он называется конкретным.

~~~
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

const b = new Base();
// Cannot create an instance of an abstract class.
~~~

Мы не можем создать экземпляр `Base` с `new`, потому что он абстрактный. Вместо этого нам нужно создать производный класс и реализовать абстрактные члены:

~~~
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

class Derived extends Base {
  getName() {
    return "world";
  }
}

const d = new Derived();
d.printName();
~~~

Обратите внимание, что если мы забудем реализовать абстрактные члены базового класса, мы получим ошибку:

~~~
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

class Derived extends Base {
  // Non-abstract class 'Derived' does not implement inherited abstract member 'getName' from class 'Base'.
  // forgot to do anything
}
~~~

Иногда мы хотим принять некоторую функцию-конструктор класса, которая создает экземпляр класса, производного от некоторого абстрактного класса.

Для этого мы можем написать функцию, которая принимает что-то с сигнатурой конструкции:

~~~
abstract class Base {
  abstract getName(): string;

  printName() {
    console.log("Hello, " + this.getName());
  }
}

class Derived extends Base {
  getName() {
    return "world";
  }
}

function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}

greet(Derived);
greet(Base);
// Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
// Cannot assign an abstract constructor type to a non-abstract constructor type.
~~~

TypeScript правильно сообщает вам о том, какие функции конструктора класса можно вызывать — `Derived` может, потому что он конкретный, но `Base` не может.

### Отношения между классами

В большинстве случаев классы в TypeScript сравниваются структурно, как и другие типы.

Например, эти два класса можно использовать вместо друг друга, потому что они идентичны:

~~~
class Point1 {
  x = 0;
  y = 0;
}

class Point2 {
  x = 0;
  y = 0;
}

// OK
const p: Point1 = new Point2();
~~~

Точно так же отношения подтипов между классами существуют, даже если нет явного наследования:

~~~
class Person {
  name: string;
  age: number;
}

class Employee {
  name: string;
  age: number;
  salary: number;
}

// OK
const p: Person = new Employee();
~~~

Это звучит просто, но есть несколько случаев, которые кажутся более странными, чем другие.

Пустые классы не имеют членов. В системе структурных типов тип без элементов обычно является супертипом чего-либо еще. Поэтому, если вы пишете пустой класс (не делайте этого!), вместо него можно использовать что угодно:

~~~
class Empty {}

function fn(x: Empty) {
  // can't do anything with 'x', so I won't
}

// All OK!
fn(window);
fn({});
fn(fn);
~~~






### Краткий справочник

![Classes](../images/classes.png)
