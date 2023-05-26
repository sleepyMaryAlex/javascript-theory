# ?Attributes and properties

Для узлов-элементов большинство стандартных HTML-атрибутов автоматически становятся свойствами DOM-объектов.

Например, для такого тега `<body id="page">` у DOM-объекта будет такое свойство `body.id="page"`.

### DOM-свойства

DOM-узлы – это обычные объекты JavaScript. Мы можем их изменять.

~~~
document.body.myData = {
  name: "Caesar",
  title: "Imperator",
};

console.log(document.body.myData.title); // Imperator
~~~

DOM-свойства и методы регистрозависимы.

### HTML-атрибуты

В HTML у тегов могут быть атрибуты. Когда браузер парсит HTML, чтобы создать DOM-объекты для тегов, он распознаёт стандартные атрибуты и создаёт DOM-свойства для них.

Таким образом, когда у элемента есть `id` или другой стандартный атрибут, создаётся соответствующее свойство. Но этого не происходит, если атрибут нестандартный.

~~~
<body id="test" something="non-standard">
  <script>
    console.log(document.body.id); // test
    // нестандартный атрибут не преобразуется в свойство
    console.log(document.body.something); // undefined
  </script>
</body>
~~~

Есть ли способ получить такие атрибуты?

Конечно. Все атрибуты доступны с помощью следующих методов:

* `elem.hasAttribute(name)` – проверяет наличие атрибута.
* `elem.getAttribute(name)` – получает значение атрибута.
* `elem.setAttribute(name, value)` – устанавливает значение атрибута.
* `elem.removeAttribute(name)` – удаляет атрибут.

Эти методы работают именно с тем, что написано в HTML.

Кроме этого, получить все атрибуты элемента можно с помощью свойства `elem.attributes`: коллекция объектов, которая принадлежит ко встроенному классу `Attr` со свойствами `name` и `value`.

~~~
<body something="non-standard">
  <script>
    console.log(document.body.getAttribute('something')); // non-standard
  </script>
</body>
~~~

У HTML-атрибутов есть следующие особенности:

* Их имена регистронезависимы (`id` то же самое, что и `ID`).
* Их значения всегда являются строками.

~~~
<body>
  <div id="elem" about="Elephant"></div>

  <script>
    console.log(elem.getAttribute('About')); // Elephant

    elem.setAttribute('Test', 123);

    console.log(elem.outerHTML); // <div id="elem" about="Elephant" test="123"></div>

    for (const attr of elem.attributes) { // (4) весь список
      console.log(`${attr.name} = ${attr.value}`);
    }
    // id = elem
    // main.html:29 about = Elephant
    // main.html:29 test = 123
  </script>
</body>
~~~

### Синхронизация между атрибутами и свойствами

Когда стандартный атрибут изменяется, соответствующее свойство автоматически обновляется. Это работает и в обратную сторону (за некоторыми исключениями).

В примере ниже `id` модифицируется как атрибут, и можно увидеть, что свойство также изменено. То же самое работает и в обратную сторону:

~~~
<label>Text
  <input type="text" value="значение">
</label>

<script>
  const input = document.querySelector('input');

  // атрибут => свойство
  input.setAttribute('id', 'id');
  console.log(input.id); // id (обновлено)

  // свойство => атрибут
  input.id = 'newId';
  console.log(input.getAttribute('id')); // newId (обновлено)
</script>
~~~

Но есть и исключения, например, `input.value` синхронизируется только в одну сторону – атрибут → значение, но не в обратную:

~~~
<label>Text
  <input type="text">
</label>

<script>
  const input = document.querySelector('input');

  // атрибут => значение
  input.setAttribute('value', 'text');
  console.log(input.value); // text

  // свойство => атрибут
  input.value = 'newValue';
  console.log(input.getAttribute('value')); // text (не обновилось!)
</script>
~~~

Иногда эта «особенность» может пригодиться, потому что действия пользователя могут приводить к изменениям `value`, и если после этого мы захотим восстановить «оригинальное» значение из HTML, оно будет в атрибуте.

### DOM-свойства типизированы

DOM-свойства не всегда являются строками. Например, свойство `input.checked` (для чекбоксов) имеет логический тип:

Есть и другие примеры. Атрибут `style` – строка, но свойство `style` является объектом:
~~~
<div id="div" style="color:red; font-size:120%">Hello</div>

<script>
  // строка
  console.log(div.getAttribute('style')); // color:red;font-size:120%

  // объект
  console.log(div.style); // [object CSSStyleDeclaration]
  console.log(div.style.color); // red
</script>
~~~

Хотя большинство свойств, всё же, строки.

При этом некоторые из них, хоть и строки, могут отличаться от атрибутов. Например, DOM-свойство `href` всегда содержит полный URL, даже если атрибут содержит относительный URL или просто `#hash`.

~~~
<a id="a" href="#hello">Link</a>
<script>
  // атрибут
  console.log(a.getAttribute('href')); // #hello

  // свойство
  console.log(a.href); // полный URL в виде http://site.com/page#hello
</script>
~~~

### Нестандартные атрибуты, `dataset`

Иногда нестандартные атрибуты используются для передачи пользовательских данных из HTML в JavaScript, или чтобы «помечать» HTML-элементы для JavaScript.

~~~
<div show-info="name"></div>
<div show-info="age"></div>

<script>
  const user = {
    name: "Pete",
    age: 25
  };

  for (const div of document.querySelectorAll('[show-info]')) {
    const field = div.getAttribute('show-info');
    div.innerHTML = user[field]; // Pete 25
  }
</script>
~~~

Также они могут быть использованы, чтобы стилизовать элементы.

Например, здесь для состояния заказа используется атрибут `order-state`:

~~~
<style>
  .order[order-state="new"] {
    color: green;
  }
</style>

<div class="order" order-state="new">
  A new order.
</div>
~~~

Почему атрибут может быть предпочтительнее таких классов, как `.order-state-new`?

Это потому, что атрибутом удобнее управлять. Состояние может быть изменено достаточно просто:

~~~
// немного проще, чем удаление старого/добавление нового класса
div.setAttribute('order-state', 'new');
~~~

Но с пользовательскими атрибутами могут возникнуть проблемы. Что если мы используем нестандартный атрибут для наших целей, а позже он появится в стандарте и будет выполнять какую-то функцию? Язык HTML живой, он растёт, появляется больше атрибутов, чтобы удовлетворить потребности разработчиков. В этом случае могут возникнуть неожиданные эффекты.

Чтобы избежать конфликтов, существуют атрибуты вида `data-*`.

Все атрибуты, начинающиеся с префикса `«data-»`, зарезервированы для использования программистами. Они доступны в свойстве `dataset`.

Например, если у `elem` есть атрибут `"data-about"`, то обратиться к нему можно как `elem.dataset.about`.

~~~
<body data-about="Elephants">
  <script>
    console.log(document.body.dataset.about); // Elephants
  </script>
</body>
~~~

Атрибуты, состоящие из нескольких слов, к примеру `data-order-state`, становятся свойствами, записанными с помощью верблюжьей нотации: `dataset.orderState`.

Использование `data-*` атрибутов – валидный, безопасный способ передачи пользовательских данных.

Мы можем не только читать, но и изменять `data`-атрибуты.
