# ?Enum

Тип `enum` или перечисление позволяет определить набор именнованных констант, которые описывают определенные состояния.

Для определения перечисления применяется ключевое слово `enum`. Например, объявим следующее перечисление:

~~~
enum Season { Winter, Spring, Summer, Autumn };
~~~

Перечисление называется `Season` и имеет четыре элемента. Теперь используем перечисление:

~~~
enum Season {
  Winter,
  Spring,
  Summer,
  Autumn,
}

let current: Season = Season.Summer;
console.log(current); // 2
current = Season.Autumn; // изменение значения
console.log(Season.Summer); // 2
console.log(current); // 3
console.log(Season);
// {
//   '0': 'Winter',
//   '1': 'Spring',
//   '2': 'Summer',
//   '3': 'Autumn',
//   Winter: 0,
//   Spring: 1,
//   Summer: 2,
//   Autumn: 3
// }
~~~

Здесь создается переменная `current`, которая имеет тип `Season`. При этом консоль браузера выведет нам число `2` - значение константы `Season.Summer`.

TypeScript компилирует это в следующий JavaScript:

~~~
var Season;
(function (Season) {
    Season[Season["Winter"] = 0] = "Winter";
    Season[Season["Spring"] = 1] = "Spring";
    Season[Season["Summer"] = 2] = "Summer";
    Season[Season["Autumn"] = 3] = "Autumn";
})(Season || (Season = {}));

// Запись типа Season["Winter"] = 0 возвращает 0
~~~

В этом сгенерированном коде перечисление компилируется в объект, который хранит как прямое `(name -> value)`, так и обратное `(value -> name)` сопоставления.

### Числовые перечисления

По умолчанию константы перечисления, как в примере выше, представляют числовые значения. То есть это так называемое числовое перечисление, в котором каждой константе сопоставляется числовое значение.

Так, созданное выше в примере перечисление

~~~
enum Season { Winter, Spring, Summer, Autumn };
~~~

фактически эквивалентно следующему:

~~~
enum Season {
  Winter = 0,
  Spring = 1,
  Summer = 2,
  Autumn = 3,
}
~~~

Хотя мы можем явным образом переопределить эти значения. Так, мы можем задать значение одной константы, тогда значения следующих констант будет увеличиваться на единицу:

~~~
enum Season {
  Winter = 5,
  Spring,
  Summer,
  Autumn,
}

console.log(Season);
// {
//   '5': 'Winter',
//   '6': 'Spring',
//   '7': 'Summer',
//   '8': 'Autumn',
//   Winter: 5,
//   Spring: 6,
//   Summer: 7,
//   Autumn: 8
// }
~~~

Такое поведение с автоинкрементом полезно в случаях, когда нас могут не интересовать сами значения членов, но важно, чтобы каждое значение отличалось от других значений в том же перечислении.

Либо можно каждой константе задать свое значение:

~~~
enum Season {
  Winter = 4,
  Spring = 8,
  Summer = 16,
  Autumn = 32,
}
~~~

Также мы можем получить непосредственно текстовое значение:

~~~
enum Season {
  Winter = 0,
  Spring = 1,
  Summer = 2,
  Autumn = 3,
}

let current: string = Season[2]; // 2 - числовое значение Summer
console.log(current); // Summer
~~~

Числовые перечисления могут быть смешаны в вычисляемых и постоянных членах. Перечисления без инициализаторов либо должны быть первыми, либо следовать за числовыми перечислениями, инициализированными числовыми константами или другими постоянными элементами перечисления.

Другими словами, запрещено следующее:

~~~
function getSomeValue() {
  return 5;
}

enum E {
  A = getSomeValue(),
  B,
  // Enum member must have initializer.
}
~~~

#### Вычисляемые и постоянные (константные) члены

Константное выражение перечисления — это подмножество выражений TypeScript, которые можно полностью вычислить во время компиляции. Выражение является константным выражением перечисления, если оно:

1. Буквальное выражение перечисления (в основном строковый литерал или числовой литерал)
2. Ссылка на ранее определенный константный член перечисления (который может происходить из другого перечисления)
3. Константное перечисляемое выражение в скобках
4. Один из унарных операторов `+`, `-`, `~`, примененный к константному выражению перечисления
5. `+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^` бинарные операторы с константными выражениями перечисления в качестве операндов

Это ошибка времени компиляции для постоянных выражений перечисления, которые должны оцениваться как `NaN` или `Infinity`.

Во всех остальных случаях член перечисления считается вычисленным.

~~~
enum FileAccess {
  // constant members
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // computed member
  G = "123".length,
}
~~~

#### Перечисления `union` и типы членов `enum`

Когда все члены перечисления имеют буквальные значения перечисления, в игру вступает некоторая особая семантика.

Во-первых, члены перечисления также становятся типами! Например, мы можем сказать, что некоторые элементы могут иметь значение только члена `enum`:

~~~
enum ShapeKind {
  Circle,
  Square,
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

const c: Circle = {
  kind: ShapeKind.Square,
  // Type 'ShapeKind.Square' is not assignable to type 'ShapeKind.Circle'.
  radius: 100,
};
~~~

Другое изменение заключается в том, что сами типы `enum` фактически становятся `union` каждого члена `enum`. С `union` `enums` система типов может использовать тот факт, что она знает точный набор значений, которые существуют в самом `enum`. Из-за этого TypeScript может обнаруживать ошибки, из-за которых мы можем неправильно сравнивать значения. Например:

~~~
enum E {
  Foo,
  Bar,
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) { // всегда будет true
    // This comparison appears to be unintentional because the types 'E.Foo' and 'E.Bar' have no overlap.
  }
}
~~~

Такая ошибка будет и без `enum`:

~~~
function f(x: number) {
  if (x !== 0 || x !== 1) {
    // This comparison appears to be unintentional because the types '0' and '1' have no overlap.
  }
}
~~~

### Строковые перечисления

Кроме числовых перечислений в TypeScript есть строковые перечисления, константы которых принимают строковые значения:

~~~
enum Season {
  Winter = "Зима",
  Spring = "Весна",
  Summer = "Лето",
  Autumn = "Осень",
}

let current: Season = Season.Summer;
console.log(current); // Лето
~~~

Строковые перечисления не имеют автоинкрементного поведения.

В строковом перечислении каждый член должен быть инициализирован константой строковым литералом или другим членом строкового перечисления.

Члены строкового перечисления вообще не генерируют обратное сопоставление.

~~~
enum Season {
  Morning = "Good morning",
  Evening = "Good evening",
}

console.log(Season); // { Morning: 'Good morning', Evening: 'Good evening' }
~~~

TypeScript компилирует это в следующий JavaScript:

~~~
var Season;
(function (Season) {
    Season["Morning"] = "Good morning";
    Season["Evening"] = "Good evening";
})(Season || (Season = {}));
~~~

### Смешанные гетерогенные перечисления

Также можно определять смешанные перечисления, константы которых могут числа и строки.

~~~
enum Season {
  Winter = 1,
  Spring = "Весна",
  Summer = 3,
  Autumn = "Осень",
}

let current: Season = Season.Summer;
console.log(current); // 3
console.log(Season.Autumn); // Осень
~~~

### Перечисления в функциях

Перечисление может выступать в качестве параметра функции.

~~~
enum DayTime {
  Morning,
  Evening,
}

function welcome(dayTime: DayTime) {
  if (dayTime === DayTime.Morning) {
    console.log("Good morning");
  } else {
    console.log("Good evening");
  }
}

let current: DayTime = DayTime.Morning;
welcome(current); // Good morning
welcome(DayTime.Evening); // Good evening
~~~

Каждая константа перечисления описывает некоторое состояние. И функция `welcome()` в виде параметра `dayTime` принимает это состояние и в зависимости от полученного значения выводит на консоль определенное значение.

Однако стоит отметить, что поскольку здесь перечисление `DayTime` представляет числовое перечисление, то в реальности в функцию `welcome()` мы можем передать числовые значения:

~~~
welcome(1); // Good evening
~~~

Либо даже определить параметр функции как числовой и передавать константы числового перечисления:

~~~
function welcome(dayTime: number) {
  // ...
}
~~~

Пример параметра-строкового перечисления:

~~~
enum DayTimeMessage {
  Morning = "Good morning",
  Evening = "Good evening",
}

function welcome(message: DayTimeMessage) {
  console.log(message);
}

let mes: DayTimeMessage = DayTimeMessage.Morning;
welcome(mes); // Good morning
welcome(DayTimeMessage.Evening); // Good evening
~~~

Из-за того, что члены строкового перечисления вообще не генерируют обратное сопоставление, мы не можем сделать так:

~~~
enum DayTimeMessage {
  Morning = "Good morning",
  Evening = "Good evening",
}

function welcome(message: DayTimeMessage) {
  console.log(message);
}

welcome("Good evening"); // Argument of type '"Good evening"' is not assignable to parameter of type 'DayTimeMessage'.
~~~

### Перечисления во время выполнения

`Enums` — это реальные объекты, которые существуют во время выполнения. Например, следующий `enum` на самом деле может быть передан функциям:

~~~
enum E {
  X,
  Y,
  Z,
}

function f(obj: { X: number }) {
  return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
~~~

### Объекты против `enum`

В современном TypeScript вам может не понадобиться `enum`, когда может быть достаточно объекта с `as const`:

~~~
const enum EDirection {
  Up,
  Down,
  Left,
  Right,
}

const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const;

// Using the enum as a parameter
function walk(dir: EDirection) {
  console.log(dir); // 2
}

// It requires an extra line to pull out the values
type Direction = (typeof ODirection)[keyof typeof ODirection];
function run(dir: Direction) {
  console.log(dir); // 3
}

walk(EDirection.Left);
run(ODirection.Right);
~~~
