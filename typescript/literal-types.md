# ?Literal Types

В дополнение к общим типам `string` и `number`, мы можем ссылаться на определенные строки и числа в позициях типа.

Один из способов подумать об этом — рассмотреть, как в JavaScript существуют различные способы объявления переменных. `let` позволяет изменять то, что хранится внутри переменной, а `const` - нет.

~~~
let changingString = "Hello World";
// let changingString: string
 
const constantString = "Hello World";
// const constantString: "Hello World"
~~~

Сами по себе литеральные типы не очень ценны. Нет смысла иметь переменную, которая может иметь только одно значение!

Но объединяя литералы в союзы, вы можете выразить гораздо более полезную концепцию — например, функции, которые принимают только определенный набор известных значений:

~~~
function printText(s: string, alignment: "left" | "right") {
  // ...
}

printText("Hello, world", "left");
printText("G'day, mate", "center");
// Error: Argument of type '"center"' is not assignable to parameter of type '"left" | "right"'.
~~~

### Буквальный вывод

Когда вы инициализируете переменную объектом, TypeScript предполагает, что свойства этого объекта могут изменить значения позже. Например, если вы написали такой код:

~~~
const obj = { counter: 0 };
// const obj: { counter: number } = { counter: 0 }
if (true) {
  obj.counter = 1;
}
~~~

TypeScript не считает, что присвоение `1` полю, которое ранее имело `0`, является ошибкой.

То же самое относится и к строкам:

~~~
function handleRequest(url: string, method: "GET" | "POST") {
  // ...
}

const req = { url: "https://example.com", method: "GET" };
// const req: { url: string, method: string } = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// Error: Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
~~~

Есть два способа обойти это.

1. Вы можете изменить вывод, добавив утверждение типа в любом месте:

Change 1:

~~~
const req = { url: "https://example.com", method: "GET" as "GET" };
~~~

Change 2

~~~
handleRequest(req.url, req.method as "GET");
~~~

2. Вы можете использовать `as const` для преобразования всего объекта в литералы типов:

~~~
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
~~~

Суффикс `as const` действует так же, `const` но для системы типов, гарантируя, что всем свойствам будет присвоен литеральный тип, а не более общая версия, такая как `string` или `number`.
