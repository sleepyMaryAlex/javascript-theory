# ?Modules

JavaScript имеет долгую историю различных способов обработки модульного кода. Существующий с 2012 года, TypeScript реализовал поддержку многих из этих форматов, но со временем сообщество и спецификация JavaScript сошлись на формате, называемом модулями ES (или модулями ES6). Возможно, вы знаете его как синтаксис `import`/`export`.

Модули ES были добавлены в спецификацию JavaScript в 2015 году и к 2020 году получили широкую поддержку в большинстве веб-браузеров и сред выполнения JavaScript.

### Как определяются модули JavaScript

В TypeScript, как и в ECMAScript 2015, любой файл, содержащий файл верхнего уровня `import` или `export` считается модулем.

И наоборот, файл без каких-либо объявлений импорта или экспорта верхнего уровня рассматривается как `script`, содержимое которого доступно в глобальной области (и, следовательно, также для модулей).

Модули выполняются в своей собственной области видимости, а не в глобальной области видимости. Это означает, что переменные, функции, классы и т. д., объявленные в модуле, не видны за пределами модуля, если только они не экспортированы явным образом с помощью одной из форм экспорта.

И наоборот, чтобы использовать переменную, функцию, класс, интерфейс и т. д., экспортированные из другого модуля, их необходимо импортировать с помощью одной из форм импорта.

### Не модули

Внутри файла `script` переменные и типы объявляются в общей глобальной области видимости, и предполагается, что вы будете либо использовать параметр компилятора `outFile` для объединения нескольких входных файлов в один выходной файл, либо использовать несколько тегов `<script>` в своем HTML. чтобы загрузить эти файлы (в правильном порядке!).

### Модули в TypeScript

#### Синтаксис модуля ES

Файл может объявить основной экспорт через `export default`:

hello.ts
~~~
export default function helloWorld() {
  console.log("Hello, world!");
}
~~~

Затем это импортируется через:

~~~
import helloWorld from "./hello.js";
helloWorld();
~~~

В дополнение к экспорту по умолчанию вы можете иметь более одного экспорта переменных и функций через `export`, опуская `default`:

maths.ts
~~~
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;
 
export class RandomNumberGenerator {}
 
export function absolute(num: number) {
  if (num < 0) {
    return num * -1;
  }
  return num;
}
~~~

Их можно использовать в другом файле с помощью `import`:

~~~
import { pi, phi, absolute } from "./maths.js";

console.log(pi);
const absPhi = absolute(phi);
// const absPhi: number;
~~~

Импорт можно переименовать, используя следующий формат `import {old as new}`:

~~~
import { pi as π } from "./maths.js";
 
console.log(π);
// (alias) var π: number
~~~

Вы можете смешивать и сопоставлять приведенный выше синтаксис в один `import`:

maths.ts
~~~
export const pi = 3.14;
export default class RandomNumberGenerator {}
~~~

app.ts
~~~
import RandomNumberGenerator, { pi as π } from "./maths.js";
// (alias) class RandomNumberGenerator

console.log(π);     
// (alias) const π: 3.14
~~~

Вы можете взять все экспортированные объекты и поместить их в одно пространство имен, используя `* as name`:

app.ts
~~~
import * as math from "./maths.js";
 
console.log(math.pi);
~~~

Вы можете импортировать файл и не включать какие-либо переменные в свой текущий модуль через `import "./file"`:

app.ts
~~~
import "./maths.js";
 
console.log("3.14");
~~~

В этом случае `import` ничего не делает. Однако был оценен весь код `maths.ts`, который мог вызвать побочные эффекты, влияющие на другие объекты.

#### Синтаксис модуля `ES`, специфичный для TypeScript

Типы можно экспортировать и импортировать, используя тот же синтаксис, что и значения JavaScript:

~~~
// animal.ts
export type Cat = { breed: string; yearOfBirth: number };
 
export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}
 
// app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
~~~

TypeScript расширил синтаксис `import` двумя концепциями для объявления импорта типа:

~~~
import type
~~~

Это оператор импорта, который может импортировать только типы.

TypeScript 4.5 также позволяет использовать префикс `type` для отдельных импортов, чтобы указать, что импортируемая ссылка является типом:

~~~
import { type Cat, type Dog } from "./script.js";
~~~

Они позволяют транспилятору, отличному от TypeScript, такому как _Babel_, _swc_ или _esbuild_, знать, какие импорты можно безопасно удалить.


### Синтаксис CommonJS

CommonJS — это формат, в котором поставляется большинство модулей `npm`. Даже если вы пишете с использованием приведенного выше синтаксиса модулей ES, краткое понимание того, как работает синтаксис CommonJS, поможет вам легче выполнять отладку.

#### Экспорт

Идентификаторы экспортируются путем установки свойства `exports` в глобальном вызываемом модуле.

~~~
function absolute(num: number) {
  if (num < 0) {
    return num * -1;
  }
  return num;
}

module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
~~~

Затем эти файлы можно импортировать с помощью инструкции `require`:

~~~
const maths = require("./maths");
~~~

Или вы можете немного упростить, используя деструктурирование в JavaScript:

~~~
const { squareTwo } = require("./maths");
~~~

Функция `require` — это глобальная функция, предоставляемая средой выполнения Node.js. Он используется для загрузки и выполнения модулей в приложении Node.js. Однако `require` недоступна в веб-браузерах, поскольку веб-браузеры не поддерживают модульную систему Node.js.

#### Взаимодействие модулей CommonJS и ES

Существует несоответствие функций между модулями CommonJS и ES в отношении различия между импортом по умолчанию и импортом объекта пространства имен модуля. TypeScript имеет флаг компилятора `esModuleInterop`, чтобы уменьшить трение между двумя различными наборами ограничений.

### TypeScript’s Module Output Options

Есть два параметра, которые влияют на вывод JavaScript:

* `target` - определяет, какие функции JS будут преобразованы для работы в более старых средах выполнения JavaScript, а какие останутся нетронутыми. Определяется в `tsconfig.json`.
* `module` - определяет, какой код используется для взаимодействия модулей друг с другом. Определяется в `tsconfig.json`.

Например, вот файл TypeScript с использованием синтаксиса модулей ES, демонстрирующий несколько различных вариантов для `module`:

~~~
import { valueOfPi } from "./constants.js";
 
export const twoPi = valueOfPi * 2;
~~~

ES2020

~~~
import { valueOfPi } from "./constants.js";
export const twoPi = valueOfPi * 2;
~~~

CommonJS

~~~
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.twoPi = void 0;
const constants_js_1 = require("./constants.js");
exports.twoPi = constants_js_1.valueOfPi * 2;
~~~

UMD

~~~
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./constants.js"], factory);
    }
})(function (require, exports) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    const constants_js_1 = require("./constants.js");
    exports.twoPi = constants_js_1.valueOfPi * 2;
});
~~~

AMD

~~~
define(["require", "exports", "./constants"], function (require, exports, constants_1) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    exports.twoPi = constants_1.valueOfPi * 2;
});
~~~

### TypeScript namespaces

TypeScript имеет собственный формат модуля `namespaces`, который предшествует стандарту ES Modules. Этот синтаксис имеет множество полезных функций для создания файлов сложных определений и до сих пор активно используется в `DefinitelyTyped`.

Везде, где при объявлении внутреннего модуля использовалось ключевое слово `module`, вместо него можно и нужно использовать ключевое слово `namespace`. Это позволяет не сбивать с толку новых пользователей, перегружая их терминами с похожими названиями.

Для определения пространств имен используется ключевое слово `namespace`:

~~~
namespace Personnel {
  export class Employee {
    constructor(public name: string) {}
  }
}
~~~

В данном случае пространство имен называется `Personnel`, и оно содержит класс `Employee`. Чтобы типы и объекты, определенные в пространстве имен, были видны извне, они определяются с ключевым словом `export`. В этом случае во вне мы сможем использовать класс `Employee`:

~~~
namespace Personnel {
  export class Employee {
    constructor(public name: string) {}
  }
}
 
const alice = new Personnel.Employee("Alice");
console.log(alice.name); // Alice
~~~

При этом пространства имен могут содержать и интерфейсы, и объекты, и функции, и даже обычные переменные и константы:

~~~
namespace Personnel {
  export interface IUser {
    displayInfo(): void;
  }

  export class Employee {
    constructor(public name: string) {}
  }

  export function work(emp: Employee): void {
    console.log(emp.name, "is working");
  }

  export const defaultUser = { name: "Kate" };

  export const value = "Hello";
}

const tom = new Personnel.Employee("Tom");
Personnel.work(tom); // Tom is working

console.log(Personnel.defaultUser.name); // Kate
console.log(Personnel.value); // Hello
~~~

#### Пространство имен в отдельном файле

Нередко пространства имен определяются в отдельных файлах. Например, определим файл `personnel.ts` со следующим кодом:

~~~
namespace Personnel {
  export class Employee {
    constructor(public name: string) {}
  }
  export class Manager {
    constructor(public name: string) {}
  }
}
~~~

И в той же папке определим главный файл приложения `app.ts`:

~~~
/// <reference path="personnel.ts" />
 
const tom = new Personnel.Employee("Tom");
console.log(tom.name);

const sam = new Personnel.Manager("Sam");
console.log(sam.name);
~~~

С помощью директивы `/// <reference path="personnel.ts" />` подключается файл `personnel.ts`.

Далее нам надо объединить оба файла в один файл, который затем можно подключать на веб-страницу. Для этого при компиляции указывается опция:

~~~
--outFile target.js sourse1.ts source2.ts source3.ts ...
~~~

Опции `outFile` в качестве первого параметра передается название файла, который будет генерироваться. А последующие параметры - файлы с кодом TypeScript, которые будут компилироваться.

То есть в данном случае нам надо выполнить в консоли команду:

~~~
tsc --outFile app.js app.ts personnel.ts
~~~

В итоге будет создан один файл `app.js`.

#### Вложенные пространства имен

Пространства имен могут быть вложенными:

~~~
namespace Data {
  export namespace Personnel {
    export class Employee {
      constructor(public name: string) {}
    }
  }
  export namespace Clients {
    export class VipClient {
      constructor(public name: string) {}
    }
  }
}

const tom = new Data.Personnel.Employee("Tom");
console.log(tom.name); // Tom

const sam = new Data.Clients.VipClient("Sam");
console.log(sam.name); // Sam
~~~

Причем вложенные пространства имен определяются со словом `export`. Соответственно при обращении к типам надо использовать все пространства имен.

#### Псевдонимы

Возможно, нам приходится создавать множество объектов `Data.Personnel.Employee`, но каждый раз набирать полное имя класса с учетом пространств имен, вероятно, не всем понравиться, особенно когда модули имеют глубокую вложенность по принципу матрешки.

Чтобы сократить объем кода, мы можем использовать псевдонимы, задаваемые с помощью ключевого слова `import`. Например:

~~~
namespace Data {
  export namespace Personnel {
    export class Employee {
      constructor(public name: string) {}
    }
  }
}

import employee = Data.Personnel.Employee;
const tom = new employee("Tom");
console.log(tom.name); // Tom
~~~

После объявления псевдонима `employee` будет рассматриваться как краткое имя для `Data.Personnel.Employee`.
