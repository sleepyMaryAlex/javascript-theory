# ?Functions

### Определение функции

Функция может иметь параметры, которые указываются после названия функции в скобках через запятую. Через двоеточие после имени параметра указывается его тип:

~~~
function add(a: number, b: number): number {
  return a + b;
}
~~~

Функция может возвращать значение определенного типа, который еще называется типом функции. Возвращаемый тип функции ставится после списка параметров через двоеточие.

В данном случае функция будет возвращать значение типа `number`.

Если функция ничего не возвращает, то указывается тип `void`.

В принципе мы можем и не указывать тип, тогда он будет выводиться неявно на основе возвращаемого значения.

### Необязательные параметры

В TypeScript при вызове в функцию должно передаваться ровно столько значений, сколько в ней определено параметров:

~~~
function getName(firstName: string, lastName: string) {
  return firstName + " " + lastName;
}
 
const fullName = getName("Tina", "Smith", "Mila");  // Expected 2 arguments, but got 3.
~~~

Чтобы иметь возможность передавать различное число значений в функцию, в TS некоторые параметры можно объявить как необязательные. Необязательные параметры должны быть помечены вопросительным знаком `?`. Причем необязательные параметры должны идти после обязательных:

~~~
function getName(firstName: string, lastName?: string) {
  if (lastName) {
    return firstName + " " + lastName;
  } else {
    return firstName;
  }
}
~~~

### Значения параметров по умолчанию

Параметры позволяют задать начальное значение по умолчанию. И если для такого параметра не передается значение, то он использует значение по умолчанию:

~~~
function getName(firstName: string, lastName: string = "Smith") {
  return firstName + " " + lastName;
}
~~~

Причем в качестве значения можно передавать результат другого выражения:

~~~
function getName(firstName: string, lastName: string = defaultLastName()) {
  return firstName + " " + lastName;
}
~~~

### Тип функции

Используя тип функции, мы можем определить переменные, константы и параметры этого типа. Например:

~~~
function hello() {
  console.log("Hello TypeScript");
}

const message: () => void = hello;
message();
~~~

Другой пример - функция, которая принимает параметры и возвращает некоторый результат:

~~~
function sum(x: number, y: number): number {
  return x + y;
}
~~~

Она имеет тип `(x: number, y: number) => number;`, то есть принимает два параметра `number` и возвращает значение типа `number`.

Также мы можем определять значения этого типа функции:

~~~
let op: (x:number, y:number) => number;
~~~

То есть переменная `op` представляет любую функцию, которая принимает два числа и которая возвращает число. Например:

~~~
function sum(x: number, y: number): number {
  return x + y;
}

function subtract(a: number, b: number): number {
  return a - b;
}

let op: (x: number, y: number) => number;

op = sum;
console.log(op(2, 4)); // 6

op = subtract;
console.log(op(6, 4)); // 2
~~~

Здесь вначале переменная `op` указывает на функцию `sum`. И соответственно вызов `op(2, 4)` фактически будет представлять вызов `sum(2, 4)`. А затем `op` указывает на функцию `subtract`.

### Функции как параметры других функций

Тип функции можно использовать как тип переменной, но он также может применяться для определения типа параметра другой функции:

~~~
function sum(x: number, y: number): number {
  return x + y;
}

function multiply(a: number, b: number): number {
  return a * b;
}

function mathOp(
  x: number,
  y: number,
  op: (a: number, b: number) => number
): number {
  return op(x, y);
}

console.log(mathOp(10, 20, sum)); // 30
console.log(mathOp(10, 20, multiply)); // 200
~~~

Здесь в функции `mathOp()` третий параметр представляет функцию, которая принимает два параметра типа `number` и возвращает значение типа `number`. Соответственно при вызове функции `mathOp()` мы можем передать в нее, например, определенные здесь функции `sum()` или `multiply()`, которые соответствуют типу `(a: number, b: number) => number`.

Если определенный тип функции предстоит очень часто использовать, то для него оптимальнее определить псевдоним и обращаться к типу по этому псевдониму:

~~~
type Operation = (a: number, b: number) => number;

function mathOp(x: number, y: number, op: Operation): number {
  return op(x, y);
}

const sum: Operation = function (x: number, y: number): number {
  return x + y;
};

console.log(mathOp(10, 20, sum)); // 30
~~~

В данном случае тип `(a: number, b: number) => number` проецируется на псевдоним `Operation`, который может использоваться для определения переменных и параметров.
