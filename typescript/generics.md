# ?Generics

Без дженериков нам пришлось бы либо присвоить функции идентификации определенный тип. Или мы могли бы описать функцию идентификации, используя `any`.

~~~
function identity(arg: any): any {
  return arg;
}
~~~

Хотя использование `any`, безусловно, является универсальным в том смысле, что оно заставит функцию принимать любые и все типы для типа `arg`, мы фактически теряем информацию о том, что это был за тип, когда функция возвращается.

Если мы передали число, единственная информация, которую мы имеем, это то, что может быть возвращен любой тип.

Вместо этого нам нужен способ захвата типа аргумента таким образом, чтобы мы могли также использовать его для обозначения того, что возвращается. Здесь мы будем использовать _переменную типа_:

~~~
function identity<Type>(arg: Type): Type {
  return arg;
}
~~~

Эта версия функции `identity` является универсальной, поскольку она работает с целым рядом типов.

После того, как мы написали общую функцию идентификации, мы можем вызвать ее одним из двух способов.

1. Первый способ — передать функции все аргументы, включая аргумент типа:

~~~
function identity<Type>(arg: Type): Type {
  return arg;
}

console.log(identity<string>("myString")); // myString
~~~

2. Второй способ также, пожалуй, самый распространенный. Здесь мы используем вывод аргумента типа, то есть мы хотим, чтобы компилятор автоматически устанавливал для нас значение `Type` на основе типа аргумента, который мы передаем:

~~~
function identity<Type>(arg: Type): Type {
  return arg;
}

console.log(identity("myString")); // myString
~~~

### Работа с переменными универсального типа

Что, если мы хотим также регистрировать длину аргумента `arg` в консоли при каждом вызове? У нас может возникнуть соблазн написать это:

~~~
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
  //Property 'length' does not exist on type 'Type'.
  return arg;
}
~~~

Допустим, мы на самом деле предполагали, что эта функция будет работать с массивами. Мы можем описать это так же, как мы создавали бы массивы других типов:

~~~
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // Array has a .length, so no more error
  return arg;
}

console.log(loggingIdentity<number>([1, 2, 3, 4])); // [ 1, 2, 3, 4 ]
~~~

Или так:

~~~
function loggingIdentity<Type>(arg: Type[]): Type[] {
  // ...
}
~~~

### Generics types

Тип `generic` функций такой же, как у `non-generic` функций, с параметрами типа, перечисленными первыми, аналогично объявлениям функций:

~~~
function identity<Type>(arg: Type): Type {
  return arg;
}

const myIdentity: <Type>(arg: Type) => Type = identity;
~~~

Мы также могли бы использовать другое имя:

~~~
function identity<Input>(arg: Input): Input {
  return arg;
}

const myIdentity: <Input>(arg: Input) => Input = identity;
~~~

Мы также можем записать общий тип как сигнатуру вызова литерального типа объекта:

~~~
function identity<Type>(arg: Type): Type {
  return arg;
}

const myIdentity: { <Type>(arg: Type): Type } = identity;
~~~

Что приводит нас к написанию нашего первого универсального интерфейса. Давайте возьмем литерал объекта из предыдущего примера и переместим его в интерфейс:

~~~
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

const myIdentity: GenericIdentityFn = identity;

myIdentity("Mary");
~~~

В аналогичном примере мы можем захотеть переместить общий параметр в параметр всего интерфейса.

~~~
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}

function identity<Type>(arg: Type): Type {
  return arg;
}

const myIdentity: GenericIdentityFn<string> = identity;

myIdentity("Mary");
~~~

### `Generic` ограничения

Иногда вам может понадобиться написать универсальную функцию, которая работает с набором типов, если у вас есть некоторые знания о том, какими возможностями будет обладать этот набор типов.

В нашем примере `loggingIdentity` мы хотели иметь доступ к свойству `.length`, но компилятор предупреждает нас, что мы не можем делать это:

~~~
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length); // Property 'length' does not exist on type 'Type'.
  // Property 'length' does not exist on type 'Type'.
  return arg;
}
~~~

Вместо того, чтобы работать со всеми типами, мы хотели бы ограничить эту функцию для работы со всеми типами, которые имеют `.length`. 

Для этого мы создадим интерфейс, описывающий наше ограничение. Здесь мы создадим интерфейс с одним свойством `.length`, а затем будем использовать этот интерфейс и ключевое слово `extends` для обозначения нашего ограничения:

~~~
interface Lengthwise {
  length: number;
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}

console.log(loggingIdentity("Mary")); // Mary
~~~

Поскольку универсальная функция теперь ограничена, она больше не будет работать со всеми типами.

~~~
console.log(loggingIdentity(3));
// Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.
~~~

### Использование параметров типа в `generic` ограничениях

Вы можете объявить параметр типа, который ограничен другим параметром типа. Например, здесь мы хотели бы получить свойство от объекта, которому дано его имя. Мы хотели бы убедиться, что мы случайно не захватим свойство, которое не существует в `obj`, поэтому мы поместим ограничение между двумя типами:

~~~
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

const x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m"); // Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
~~~

Оператор `keyof` принимает тип объекта и создает строковое или числовое литеральное объединение его ключей. Следующий тип `P` является тем же типом, что и `type P = "x" | "y"`:

~~~
type Point = { x: number; y: number };
type P = keyof Point;
~~~

Если тип имеет подпись строкового или числового индекса, `keyof` вместо этого вернет эти типы.

~~~
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
// type A = number

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
// type M = string | number
~~~

Обратите внимание, что в этом примере `M` это `string | number` — это потому, что ключи объекта JavaScript всегда приводятся к строке, поэтому `obj[0]` всегда совпадает с `obj["0"]`.
