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

Чтобы определить все типы, которые должно представлять переменная, все эти типы разделяются прямой чертой: `number | string`. В данном случае переменная `id` может представлять как тип `string`, то есть строку, так и число.

Иногда логика может различаться в зависимости от переданного типа. В этом случае можно использовать проверку на тип, чтобы разграничить логику для различных типов:

~~~
function printId(id: number | string) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}

printId("1345dgg5"); // 1345DGG5
printId(234); // 234
~~~

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

Обратите внимание, что псевдонимы — это всего лишь псевдонимы — вы не можете использовать псевдонимы типов для создания разных/отличных «версий» одного и того же типа. Когда вы используете псевдоним, это точно так же, как если бы вы написали тип с псевдонимом. Другими словами, этот код может выглядеть недопустимым, но в соответствии с TypeScript это нормально, потому что оба типа являются псевдонимами для одного и того же типа:

~~~
type UserInputSanitizedString = string;

function sanitizeInput(str: string): UserInputSanitizedString {
  return str;
}

// Create a sanitized input
let userInput = sanitizeInput("some string");

// Can still be re-assigned with a string though
userInput = "new string";
~~~

### Intersection Types. Расширение псевдонимов

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

### Только для чтения и необязательные параметры

`readonly` означает, что пользователь или кто-либо другой не может манипулировать этой переменной.

`?` означает, что эти параметры являются необязательными.

~~~
type User = {
  readonly id: string;
  name: string;
  email: string;
  dob?: string; // optional
};
~~~

### Generic Object Types

~~~
type Box<Type> = {
  contents: Type;
};

const boxA: Box<string> = { contents: "hello" };
~~~

Поскольку псевдонимы типов, в отличие от интерфейсов, могут описывать больше, чем просто типы объектов, мы также можем использовать их для написания других видов общих вспомогательных типов.

### Подписи индекса

~~~
type Name = {
  [index: string]: string;
}

const fullName: Name = { firstName: "Alex", lastName: "Smith" };
const firstName = fullName.firstName;

console.log(firstName); // Alex
~~~

### Модификаторы сопоставления (Mapped Types)

Если вы не хотите повторяться, иногда тип должен быть основан на другом типе.

Есть два дополнительных модификатора, которые можно применять во время сопоставления: `readonly` и `?` которые влияют на изменчивость и необязательность соответственно.

Вы можете удалить или добавить эти модификаторы, добавив префикс `-` или `+`. Если вы не добавите префикс, то `+` предполагается.

~~~
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
  readonly id: string;
  readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;

const account: UnlockedAccount = { id: "12345", name: "Tom" };

account.name = "Alex";
~~~

### Краткий справочник

![Type](../images/types.png)
