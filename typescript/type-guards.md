# ?Type guards

В TypeScript защита типа используется для определения типа переменной, часто внутри условного или функционального блока. Охранники типов обычно принимают переменную и возвращают логическое значение или тип переменной.

Определяемые пользователем охранники типа могут быть созданы в TypeScript, но он также имеет встроенные операторы, такие как оператор `typeof`, `in` и `instanceof`.

### `typeof` type guard

С помощью оператора `typeof` мы можем проверить тип переменной. Это может быть необходимо, когда мы хотим выполнить некоторые операции с переменной, но нам неизвестен ее точный тип (например, переменная представляет тип `any`).

~~~
let sum: any;
sum = 1200;

if (typeof sum === "number") {
  const result: number = sum / 12;
  console.log(result);
} else {
  console.log("Invalid operation");
}
~~~

Оператор `typeof` может возвращать следующие значения:

* `"string"`
* `"number"`
* `"bigint"`
* `"boolean"`
* `"symbol"`
* `"undefined"`
* `"object"`
* `"function"`

### `in` type guard

В JavaScript есть оператор для определения того, имеет ли объект или его цепочка прототипов свойство с именем: оператор `in`. TypeScript учитывает это как способ сузить число возможных типов.

~~~
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}

console.log(move({ fly: () => console.log("fly") })); // fly
~~~

### `instanceof` narrowing

В JavaScript есть оператор для проверки того, является ли значение «экземпляром» другого значения.

~~~
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
    // /(parameter) x: Date
  } else {
    console.log(x.toUpperCase());
    // (parameter) x: string
  }
}

logValue("string"); // STRING
~~~

### Truthiness narrowing

В JavaScript такие конструкции, как `if` сначала конвертируют свои условия к `booleans`, чтобы понять их смысл, а затем выбирают свои ветви в зависимости от того, является ли результат результатом `true` или `false`.

Довольно популярно использовать это поведение, особенно для защиты от таких значений, как `null` или `undefined`.

~~~
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
~~~

### Equality narrowing

TypeScript также использует операторы `switch` и проверки на равенство, такие как `===`, `!==`, `==` и `!=`, для узких типов.

~~~
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x);
    console.log(y);
  }
}
~~~

Более слабые проверки равенства в JavaScript с помощью `==` и `!=` также корректно сужаются. Проверка того, действительно ли что-то `==` `null`, не только проверяет, является ли это именно значением `null`, но также проверяет, является ли оно потенциально `undefined`. То же самое относится и к `==` `undefined`: он проверяет, является ли значение `null` или `undefined`.

~~~
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  if (container.value != null) {
    console.log(container.value);
    // (property) Container.value: number

    container.value *= factor;
  }
}
~~~

### Assignments

TypeScript смотрит на правую часть присваивания и соответствующим образом сужает левую сторону.

~~~
let x = Math.random() < 0.5 ? 10 : "hello world!";
// let x: string | number

x = 1;
console.log(x);
// let x: number

x = "goodbye!";
console.log(x);
// let x: string
~~~

Обратите внимание, что каждое из этих присвоений допустимо. Несмотря на то, что наблюдаемый тип x изменился на число после нашего первого присваивания, мы по-прежнему могли присвоить x строку.

### Control flow analysis

TypeScript использует этот анализ потока для сужения типов по мере того, как он сталкивается с защитой типов и присваиваниями. Когда переменная анализируется, поток управления может разделяться и объединяться снова и снова, и можно наблюдать, что эта переменная имеет разный тип в каждой точке.

~~~
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;

  console.log(x);
  // let x: boolean

  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x);
    // let x: string
  } else {
    x = 100;
    console.log(x);
    // let x: number
  }

  return x;
  // let x: string | number
}
~~~

### Using type predicates

До сих пор мы работали с существующими конструкциями JavaScript, чтобы справиться с сужением, однако иногда вам нужен более прямой контроль над тем, как типы меняются в вашем коде.

Чтобы определить определяемую пользователем защиту типа, нам просто нужно определить функцию, возвращаемый тип которой является предикатом типа:

~~~
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function getSmallPet(): Fish | Bird {
  return { fly: () => console.log("fly") };
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
  // let pet: Fish
} else {
  pet.fly(); // fly
  // let pet: Bird
}
~~~

`pet is Fish` наш предикат типа в этом примере.

Обратите внимание, что TypeScript не только знает, что `pet` — это `Fish` в ветке `if`; он также знает, что в другой ветке у вас нет `Fish`, поэтому у вас должна быть `Bird`.

`boolean` - это просто тип данных, а оператор `is` используется для проверки типов.

~~~
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function getSmallPet(): Fish | Bird {
  return { fly: () => console.log("fly") };
}

function isFish(pet: Fish | Bird): boolean {
  return (pet as Fish).swim !== undefined;
}

// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim(); // Error! Property 'swim' does not exist on type 'Fish | Bird'.
  // let pet: Fish | Bird
} else {
  pet.fly(); // Error! Property 'fly' does not exist on type 'Fish | Bird'.
  // let pet: Fish | Bird
}
~~~

### The never type. Exhaustiveness checking

Тип `never` присваивается каждому типу; однако никакому типу нельзя присвоить значение `never` (кроме самого `never`). Это означает, что вы можете использовать сужение и полагаться на `never` для выполнения исчерпывающей проверки в операторе `switch`.

Например, добавление значения по умолчанию в нашу функцию `getArea`, которая пытается присвоить форме значение `never`, вызовет ошибку, если все возможные случаи не были обработаны.

~~~
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
~~~

### Краткий справочник

![Control flow analysis](../images/control-flow-analysis.png)
