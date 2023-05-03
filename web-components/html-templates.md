# ?HTML Templates

Встроенный элемент `<template>` предназначен для хранения шаблона HTML. Браузер полностью игнорирует его содержимое, проверяя лишь синтаксис, но мы можем использовать этот элемент в JavaScript, чтобы создать другие элементы.

В теории, для хранения разметки мы могли бы создать невидимый элемент в любом месте HTML. Что такого особенного в `<template>`?

Во-первых, его содержимым может быть любой корректный HTML-код, даже такой, который обычно нуждается в специальном родителе.

К примеру, мы можем поместить сюда строку таблицы `<tr>`:

~~~
<template>
  <tr>
    <td>Содержимое</td>
  </tr>
</template>
~~~

Обычно, если элемент `<tr>` мы поместим, скажем, в `<div>`, браузер обнаружит неправильную структуру DOM и «исправит» её, добавив снаружи `<table>`. Это может оказаться не тем, что мы хотели. `<template>` же оставит разметку ровно такой, какой мы её туда поместили.

Также внутри `<template>` можно поместить стили и скрипты:

~~~
<template>
  <style>
    p {
      font-weight: bold;
    }
  </style>
  <script>
    console.log("Привет");
  </script>
</template>
~~~

Браузер рассматривает содержимое `<template>` как находящееся «вне документа»: стили, определённые в нём, не применяются, скрипты не выполнятся, `<video autoplay>` не запустится и т.д.

Содержимое оживёт (скрипт выполнится), когда мы поместим его в нужное нам место.

### Использование template

Содержимое шаблона доступно по его свойству `content` в качестве `DocumentFragment` – особый тип DOM-узла.

Можно обращаться с ним так же, как и с любыми другими DOM-узлами, за исключением одной особенности: когда мы его куда-то вставляем, то в это место вставляется не он сам, а его дети.

Пример:

~~~
<body>
  <template id="tmpl">
    <script>
      console.log("Привет");
    </script>
    <tr>
      <td>Содержимое</td>
    </tr>
  </template>

  <script>
    const elem = document.createElement('section');

    // Клонируем содержимое шаблона для того, чтобы переиспользовать его несколько раз
    elem.append(tmpl.content.cloneNode(true));

    document.body.append(elem);
    // Сейчас скрипт из <template> выполнится
  </script>
</body>
~~~

Пример Shadow DOM с помощью `<template>`:

~~~
<body>
  <template id="tmpl">
    <script>
      console.log("Привет");
    </script>
    <tr>
      <td id="message"></td>
    </tr>
  </template>

  <script>
    const elem = document.createElement('div');

    elem.attachShadow({ mode: 'open' });

    elem.shadowRoot.append(tmpl.content.cloneNode(true));

    elem.shadowRoot.getElementById('message').innerHTML = "Привет из теней!";

    document.body.append(elem);
  </script>
</body>
~~~

Когда мы клонируем и вставляем `tmpl.content`, то, так как это `DocumentFragment`, вместо него вставляются его потомки. Именно они и формируют теневой DOM.
