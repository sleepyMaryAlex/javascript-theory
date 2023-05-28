# ?Objects

Кроме простых типов данных, как и в Javascript, можно создавать сложные комплексные объекты, которые состоят из других объектов и примитивных данных.

В силу строготипизированности TS мы имеем некоторые ограничения. В частности, если у нас будет следующий код:

~~~
let person = { name: "Tom", age: 23 };
person = { name: "Bob" }; // ! Ошибка
~~~

То на второй строке мы получим ошибку, поскольку компилятор после первой строки предполагает, что объект `person` будет иметь два свойства `name` и `age`, которые имеют тип `string` и `number` соответственно. То есть в данном случае переменная `person` представляет тип `{ name: string; age: number }`. И мы могли написать так:

~~~
let person: { name: string; age: number } = { name: "Tom", age: 23 };
~~~

Поэтому данной переменной мы можем присвоить другой объект, который соответствует типу `{ name: string; age: number }` по названиям, количеству и типу свойств.

### Необязательные свойства

TypeScript позволяет сделать свойства необязательными. Для этого после названия свойства указывается знак вопроса `?`:

~~~
let person: { name: string; age?: number }; // Свойство age - необязательное
~~~

При обращении к неустановленному свойству мы получим `undefined`:

~~~
let person: { name: string; age?: number } = { name: "Tom", age: 23 };

console.log(person.age); // 23
person = { name: "Bob" };
console.log(person.age); // undefined
~~~

Поэтому при операциях с таким свойством мы можем проверять его на значение `undefined`.

### Объекты в функциях

Объект, передаваемый в качестве параметра, может быть более широким - содержать больше свойств, но главное, чтобы он содержал те свойства, которые предусмотрены для параметра функции:

~~~
function printUser(user: { name: string; age: number }) {
  console.log(`name: ${user.name}  age: ${user.age}`);
}

const bob = { name: "Bob", age: 44, isMarried: true };
printUser(bob); // name: Bob  age: 44
~~~

Здесь переменная `bob` представляет тип `{ name: string; age: number; isMarried: boolean }`, тем не менее он также соответствует определению типа `{ name: string; age: number }`.

Объект как результат функции:

~~~
function defaultUser(): { name: string; age: number } {
  return { name: "Tom", age: 37 };
}

const user = defaultUser();
console.log(`name: ${user.name}  age: ${user.age}`);
~~~

### Декомпозиция объектов-параметров

Если функция принимает объект в качестве параметра, то TypeScript позволяет автоматически разложить его на свойства:

~~~
function printUser({ name, age }: { name: string; age: number }) {
  console.log(`name: ${name}  age: ${age}`);
}

const tom = { name: "Tom", age: 36 };
printUser(tom); // name: Tom  age: 36
~~~

Здесь функция `printUser()` в качестве параметра принимает объект из двух свойств. Запись `{ name, age }` указывает, что свойства объекта автоматически будут разложены на два параметра: `name` и `age`. Главное, чтобы названия этих параметров соответствовали названиям свойств объекта.

Некоторые параметры могут необязательными:

~~~
function printUser({ name, age }: { name: string; age?: number }) {
  if (age !== undefined) {
    console.log(`name: ${name}  age: ${age}`);
  } else {
    console.log(`name: ${name}`);
  }
}

const tom = { name: "Tom" };
printUser(tom); // name: Tom
~~~

Также они могут принимать значения по умолчанию:

~~~
function printUser({ name, age = 25 }: { name: string; age?: number }) {
  console.log(`name: ${name}  age: ${age}`);
}

const tom = { name: "Tom" };
printUser(tom); // name: Tom  age: 25
~~~

### Типизированный пустой объект

Используйте _type assertion_ для инициализации типизированного пустого объекта в TypeScript.

~~~
interface Employee {
  id: number;
  name: string;
  salary: number;
}

const emp = {} as Employee;

emp.id = 1;
emp.name = 'Bobby Hadz';
emp.salary = 100;
~~~

Тип TypeScript `object` представляет все значения, которые не относятся к примитивным типам.

~~~
let employee: object = {};
employee = ["Jane"];
employee = "Jane"; // Type 'string' is not assignable to type 'object'.
~~~

В TypeScript есть еще один тип, называемый пустым типом, обозначаемый `{}`.

Пустой тип `{}` описывает объект, который не имеет собственного свойства.

~~~
let vacant: {} = {};
vacant.firstName = 'John';
// Property 'firstName' does not exist on type '{}'.
~~~
