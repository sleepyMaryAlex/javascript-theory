# ?Interfaces

Объявление интерфейса — это еще один способ назвать тип объекта:

~~~
interface Point {
  x: number;
  y: number;
}

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 100, y: 100 });
~~~

TypeScript заботится только о структуре значения, которому мы передали `printCoord`. Он заботится только о том, чтобы оно имело ожидаемые свойства.

### Различия между псевдонимами типов и интерфейсами

Псевдонимы типов и интерфейсы очень похожи, и во многих случаях вы можете свободно выбирать между ними. Почти все функции `interface` доступны в `type`, ключевое отличие состоит в том, что тип нельзя открыть повторно для добавления новых свойств по сравнению с интерфейсом, который всегда расширяем.

Расширение интерфейса:

~~~
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
~~~

Расширение типа через `intersections`:

~~~
type Animal = {
  name: string;
};

type Bear = Animal & {
  honey: boolean;
};
~~~

Добавление новых полей в существующий интерфейс:

~~~
interface Animal {
  name: string;
}

interface Animal {
  honey: boolean;
}

const bear: Animal = { name: "Bear", honey: false };
~~~

Тип нельзя изменить после создания:

~~~
type Animal = {
  name: string;
}

type Animal = {
  honey: boolean;
}

// Error: Duplicate identifier 'Animal'
~~~


Интерфейсы могут использоваться только для объявления форм объектов, а не для переименования примитивов.

~~~
interface AnObject1 {
  value: string;
}

type AnObject2 = {
  value: string;
};

// Using type we can create custom names
// for existing primitives:
type SanitizedString = string;
type EvenNumber = number;
~~~

По большей части вы можете выбирать на основе личных предпочтений, и TypeScript сообщит вам, нужно ли ему что-то, что может быть другим типом объявления. Если вам нужна эвристика используйте `interface` пока вам не понадобятся функции из `type`.
