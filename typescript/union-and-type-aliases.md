# ?Union & type aliases

### Union

Объединения или `union` не являются собственно типом данных, но они позволяют комбинировать или объединить другие типы.

Так, с помощью объединений можно определить переменную, которая может хранить значение двух или более типов:

~~~
let id: number | string;
id = "1345dgg5";
console.log(id); // 1345dgg5
id = 234;
console.log(id); // 234
~~~

Чтобы определить все типы, которые должно представлять переменная, все эти типы разделяются прямой чертой: `number | string`. В данном случае переменная `id` может представлять как тип string, то есть строку, так и число.

### Type aliases

TypeScript позволяет определять псевдонимы типов с помощью ключевого слова `type`:

~~~
type id = number | string;

let userId: id = 2;
console.log(`Id: ${userId}`);
userId = "qwerty";
console.log(`Id: ${userId}`);
~~~

Здесь для объединения `number | string` определяется псевдоним `id`. Далее мы можем использовать этот псевдоним для определения переменных.

В том числе псевдонимы могут применяться для определения типа параметров и результата функции:

~~~
type id = number | string;

function printId(inputId: id) {
  console.log(`Id: ${inputId}`);
}

function getId(isNumber: boolean): id {
  if (isNumber) {
    return 1;
  } else {
    return "1";
  }
}

printId(12345); // Id: 12345
printId("qwerty"); // Id: qwerty
console.log(getId(true)); // 1
~~~

Особенно полезны могут псевдонимы, когда мы имеем дело со сложными объектами:

~~~
type Person = { name: string; age: number };

const tom: Person = { name: "Tom", age: 36 };
const bob: Person = { name: "Bob", age: 41 };

function printPerson(user: Person) {
  console.log(`Name: ${user.name}  Age: ${user.age}`);
}

printPerson(tom);
printPerson(bob);
~~~

В данном случае определяется псевдоним `Person` для типа `{ name: string; age: number }`, то есть типа, который представляет сложный объект. Затем этот псевдоним применяется для определения переменных и параметров функции.

### Расширение псевдонимов

Одни псевдонимы могут заимствовать или расширять код других. Для этого применяется операция `&`. Например:

~~~
type Person = { name: string; age: number };
type Employee = Person & { company: string };
~~~

В данном случае псевдоним `Employee` расширяет псевдоним `Person`, добавляя к нему свойство `company`, которое представляет тип `string`. То есть фактически мы имеем дело с типом:

~~~
type Employee = { name: string; age: number; company: string };
~~~

Применение будет аналогично применению обычных псевдонимов:

~~~
type Person = { name: string; age: number };
type Employee = Person & { company: string };

const tom: Person = { name: "Tom", age: 36 };
const bob: Employee = { name: "Bob", age: 41, company: "Microsoft" };

function printPerson(user: Person) {
  console.log(`Name: ${user.name}  Age: ${user.age}`);
}

printPerson(tom);
printPerson(bob); // bob представляет Employee, но он также соответствует псевдониму Person
~~~
