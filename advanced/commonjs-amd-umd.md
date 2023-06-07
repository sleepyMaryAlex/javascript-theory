# ?CommonJS / AMD / UMD

_ESM_ — лучший формат модуля благодаря простому синтаксису, асинхронному характеру и возможности тряски дерева. (Рассматривается в ECMAScript/modules).
_UMD_ работает везде и обычно используется как запасной вариант на случай, если ESM не работает.
_CJS_ синхронен и хорош для серверной части.
_AMD_ асинхронна и хороша для внешнего интерфейса.

### CommonJS

CommonJS в основном используется в серверных JS-приложениях с Node, поскольку браузеры не поддерживают использование CommonJS.

С точки зрения структуры модуль CommonJS — это часть JavaScript-кода, которая экспортирует определенные переменные, объекты или функции, делая их доступными для любого зависимого кода.

Модули CommonJS состоят из двух частей: `module.exports` содержит объекты, которые модуль хочет сделать доступными, а функция `require()` используется для импорта других модулей.

Пример модуля в формате CommonJS:

~~~
// определяем функцию, которую хотим экспортировать
function foobar() {
  this.foo = function () {
    console.log("Hello foo");
  };

  this.bar = function () {
    console.log("Hello bar");
  };
}

// делаем ее доступной для других модулей
module.exports.foobar = foobar;
~~~

Пример кода, использующего модуль:

~~~
const foobar = require("./foobar").foobar,
  test = new foobar();

test.bar(); // Hello bar
~~~

### AMD

В основе формата AMD (Asynchronous Module Definition) лежат две функции: `define()` для определения именованных или безымянных модулей и `require()` для импорта зависимостей.

Функция `define()` имеет следующую сигнатуру:

~~~
define(
  module_id /* необязательный */,
  [dependencies] /* необязательный */,
  definition function /* функция для создания экземпляра модуля или объекта */
);
~~~

Параметр `module_id` необязательный, он обычно требуется только при использовании не-AMD инструментов объединения. Когда этот аргумент опущен, модуль называется анонимным. Параметр `dependencies` представляет собой массив зависимостей, которые требуются определяемому модулю, а третий аргумент `definition function` — это функция, которая выполняется для создания экземпляра модуля.

Пример модуля:

main.js
~~~
define(function (require) {
  const myteam = require("./team");
  const mylogger = require("./player");
  console.log("Player Name : " + myteam.player);
  mylogger.myfunc();
});
~~~

team.js
~~~
define({
  player: "Sachin Tendulkar",
  team: "India",
});
~~~

player.js
~~~
define(function (require) {
  const myteam = require("./team");

  return {
    myfunc: function () {
      document.write("Name: " + myteam.player + ", Country: " + myteam.team);
    },
  };
});
~~~

Функция `require()` используется для импорта модулей. Также с помощью `require()` можно динамически импортировать зависимости в модуль.

index.html
~~~
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script data-main="main" src="require.js"></script>
</head>

<body>
</body>

</html>
~~~

require.js

Также нужно настроить среду для RequireJS. Для этого вам необходимо скачать последнюю версию библиотеки RequireJS. Вы можете скачать как [минимизированную версию](https://requirejs.org/docs/release/2.3.3/minified/require.js), так и [подробную версию](https://requirejs.org/docs/release/2.3.3/comments/require.js).

### UMD

Существование двух форматов модулей, несовместимых друг с другом, не способствовало развитию экосистемы JavaScript. Для решения этой проблемы был разработан формат UMD (Universal Module Definition). Этот формат позволяет использовать один и тот же модуль и с инструментами AMD, и в средах CommonJS.

Суть подхода UMD заключается в проверке поддержки того или иного формата и объявлении модуля соответствующим образом. Пример такой реализации:

index.js
~~~
(function (global, factory) {
  if (typeof exports === "object" && typeof module !== "undefined") {
    factory(exports);
  } else if (typeof define === "function" && define.amd) {
    define(["exports"], factory);
  } else {
    factory(global.mymodule = global.mymodule || {});
  }
})(this, function (exports) {
  "use strict";

  function myFunction() {
    console.log("Hello world");
  }
  // expose the inner function on the module to use it
  exports.myFunction = myFunction;
});
~~~

Вы можете вызвать этот метод в своем `.html` файле как:

~~~
<script>
  mymodule.myFunction();
</script>
~~~

UMD — это скорее подход, а не конкретный формат. Различных реализаций может быть множество.
