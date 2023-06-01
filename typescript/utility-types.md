# ?Utility Types

TypeScript предоставляет несколько служебных типов для облегчения преобразования распространенных типов.

* `Awaited<Type>` - предназначен для моделирования таких операций, как ожидание в асинхронных функциях или метод `.then()` для промисов, в частности, способ рекурсивного развертывания промисов.

~~~
type A = Awaited<Promise<string>>;
// type A = string

async function f1() {
  const value: A = await Promise.resolve("string");
}
~~~

* `Partial<Type>` - cоздает тип со всеми свойствами `Type`, установленными как необязательные.

~~~
interface Todo {
  title: string;
  description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};

const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});

console.log(todo2); // { title: 'organize desk', description: 'throw out trash' }
~~~

* `Required<Type>` - создает тип, состоящий из всех свойств `Type`, для которых установлено значение `required`. Противоположность `Partial`.

~~~
interface Props {
  a?: number;
  b?: string;
}

const obj: Props = { a: 5 };

const obj2: Required<Props> = { a: 5 };
// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
~~~

* `Readonly<Type>` - создает тип со всеми свойствами `Type`, установленными только для чтения, что означает, что свойства созданного типа нельзя переназначить.

~~~
interface Todo {
  title: string;
}

const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};

todo.title = "Hello";
// Cannot assign to 'title' because it is a read-only property.
~~~

* `Record<Keys, Type>` - создает тип объекта, ключами свойств которого являются `Keys`, а значениями свойств — `Type`. Эту утилиту можно использовать для отображения свойств одного типа на другой тип.

~~~
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
~~~

* `Pick<Type, Keys>` - создает тип, выбирая набор свойств `Keys` (строковый литерал или объединение строковых литералов) из `Type`.

~~~
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
~~~

* `Omit<Type, Keys>` - cоздает тип, выбирая все свойства из `Type` и затем удаляя `Keys` (строковый литерал или объединение строковых литералов). Противоположность `Pick`.

~~~
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

type TodoPreview = Omit<Todo, "title" | "completed">;

const todo: TodoPreview = {
  description: "Kindergarten closes at 5pm",
};
~~~

* `Exclude<UnionType, ExcludedMembers>` - создает тип, исключая из `UnionType` все члены объединения, которые могут быть назначены `ExcludedMembers`.

~~~
type T0 = Exclude<"a" | "b" | "c", "a">;
// type T0 = "b" | "c"

type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T3 = Exclude<Shape, { kind: "circle" }>;
// type T3 = {
//     kind: "square";
//     x: number;
// } | {
//     kind: "triangle";
//     x: number;
//     y: number;
// }
~~~

* `Extract<Type, Union>` - создает тип, извлекая из `Type` все члены объединения, которые можно присвоить `Union`.

~~~
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
// type T0 = "a";

type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T2 = Extract<Shape, { kind: "circle" }>;
// type T2 = {
//   kind: "circle";
//   radius: number;
// };
~~~

* `NonNullable<Type>` - создает тип, исключая `null` и `undefined` из `Type`.

~~~
type T1 = NonNullable<string[] | null | undefined>;
// type T1 = string[];
~~~

* `Parameters<Type>` - создает тип кортежа из типов, используемых в параметрах типа функции `Type`.

~~~
type T1 = Parameters<(s: string) => void>;
// type T1 = [s: string];

type T2 = Parameters<<T>(arg: T) => T>;
// type T2 = [arg: unknown];
~~~

* `ConstructorParameters<Type>` - создает тип кортежа или массива из типов типа функции-конструктора. Он создает тип кортежа со всеми типами параметров (или тип `never`, если `Type` не является функцией).

~~~
class C {
  constructor(a: number, b: string) {}
}

type T1 = ConstructorParameters<typeof C>;
// type T1 = [a: number, b: string]
~~~

* `ReturnType<Type>` - создает тип, состоящий из возвращаемого типа функции `Type`.

~~~
function f1(): { name: string; age: number } {
  return { name: "Tom", age: 34 };
}

type T1 = ReturnType<typeof f1>;
// type T1 = {
//   name: string;
//   age: number;
// }
~~~

* `InstanceType<Type>` - создает тип, состоящий из типа экземпляра функции-конструктора в `Type`.

~~~
class C {
  x = 0;
  y = 0;
}

type T0 = InstanceType<typeof C>;
// type T0 = C;
~~~

* `ThisParameterType<Type>` - извлекает тип параметра `this` для типа функции или `unknown`, если тип функции не имеет этого параметра.

~~~
function toHex(this: Number) {
  return this.toString(16);
}

function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
~~~

* `OmitThisParameter<Type>` - удаляет параметр `this` из `Type`. Если для `Type` явно не объявлен этот параметр, результатом будет просто `Type`. В противном случае из `Type` создается новый тип функции без параметра `this`. Дженерики стираются, и только последняя сигнатура перегрузки распространяется на новый тип функции.

~~~
function toHex(this: Number) {
  return this.toString(16);
}

const tenToHex: OmitThisParameter<typeof toHex> = toHex.bind(10);

console.log(tenToHex()); // a
~~~

* `ThisType<Type>` - эта утилита не возвращает преобразованный тип. Вместо этого он служит маркером контекстуального типа. Обратите внимание, что для использования этой утилиты должен быть включен флаг `noImplicitThis`.

~~~
type User = {
  firstName: string;
  lastName: string;
};

const user: User = {
  firstName: "Test",
  lastName: "User",
};

function addUserMethods<Methods extends Record<string, Function>>(
  user: User,
  // ThisType is used to indicate that the methods should type `this` with the type of `User`
  methods: Methods & ThisType<User>
) {
  return { ...user, ...methods } as User & Methods;
}

const newUser = addUserMethods(user, {
  getFullName() {
    // These properties correctly resolve because `this` is a `User`.
    // This code would fail to compile without the `ThisType<User>`
    return `${this.firstName} ${this.lastName}`;
  },
});

console.log(newUser);
// {
//   firstName: 'Test',
//   lastName: 'User',
//   getFullName: [Function: getFullName]
// }

console.log(newUser.getFullName()); // Test User
~~~

### Внутренние типы манипуляций со строками

* `Uppercase<StringType>` - преобразует каждый символ в строке в верхний регистр.

~~~
type Greeting = "Hello, world";
type ShoutyGreeting = Uppercase<Greeting>;
// type ShoutyGreeting = "HELLO, WORLD";
~~~

* `Lowercase<StringType>` - преобразует каждый символ в строке в эквивалент нижнего регистра.

~~~
type Greeting = "Hello, world";
type QuietGreeting = Lowercase<Greeting>;
// type QuietGreeting = "hello, world";
~~~

* `Capitalize<StringType>` - преобразует первый символ строки в эквивалент в верхнем регистре.

~~~
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
// type Greeting = "Hello, world";
~~~

* `Uncapitalize<StringType>` - преобразует первый символ строки в эквивалент нижнего регистра.

~~~
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
// type UncomfortableGreeting = "hELLO WORLD";
~~~
