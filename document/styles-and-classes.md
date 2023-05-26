# ?Styles and classes

Как правило, существует два способа задания стилей для элемента:

1. Создать класс в CSS и использовать его: `<div class="...">`
2. Писать стили непосредственно в атрибуте `style`: `<div style="...">`.

JavaScript может менять и классы, и свойство `style`.

Классы – всегда предпочтительный вариант по сравнению со `style`. Мы должны манипулировать свойством `style` только в том случае, если классы «не могут справиться».

Например, использование `style` является приемлемым, если мы вычисляем координаты элемента динамически и хотим установить их из JavaScript:

~~~
const top = /* сложные расчёты */;
const left = /* сложные расчёты */;

elem.style.left = left; // например, '123px', значение вычисляется во время работы скрипта
elem.style.top = top; // например, '456px'
~~~

В других случаях, например, чтобы сделать текст красным, добавить значок фона – описываем это в CSS и добавляем класс (JavaScript может это сделать). Это более гибкое и лёгкое в поддержке решение.

### `className` и `classList`

Изменение класса является одним из наиболее часто используемых действий в скриптах.

Когда-то давно в JavaScript существовало ограничение: зарезервированное слово типа `"class"` не могло быть свойством объекта. Это ограничение сейчас отсутствует, но в то время было невозможно иметь свойство `elem.class`.

Поэтому для классов было введено схожее свойство `"className"`: `elem.className` соответствует атрибуту `"class"`.

~~~
<body class="main page">
  <script>
    console.log(document.body.className); // main page
  </script>
</body>
~~~

Если мы присваиваем что-то `elem.className`, то это заменяет всю строку с классами. Иногда это то, что нам нужно, но часто мы хотим добавить/удалить один класс.

Для этого есть другое свойство: `elem.classList`.

`elem.classList` – это специальный объект с методами для добавления/удаления одного класса.

Методы `classList`:

* `elem.classList.add/remove("class")` – добавить/удалить класс.
* `elem.classList.toggle("class")` – добавить класс, если его нет, иначе удалить.
* `elem.classList.contains("class")` – проверка наличия класса, возвращает `true`/`false`.

~~~
<body class="main page">
  <script>
    document.body.classList.add('article');
    console.log(document.body.className); // main page article
  </script>
</body>
~~~

Кроме того, `classList` является перебираемым, поэтому можно перечислить все классы при помощи `for..of`:

~~~
<body class="main page">
  <script>
    for (let name of document.body.classList) {
      console.log(name); // main page
    }
  </script>
</body>
~~~

### Element `style`

Свойство `elem.style` – это объект, который соответствует тому, что написано в атрибуте `"style"`. Установка стиля `elem.style.width="100px"` работает так же, как наличие в атрибуте `style` строки `width:100px`.

Для свойства из нескольких слов используется `camelCase`:

~~~
document.body.style.backgroundColor = prompt('background color?', 'green');
~~~

Стили с браузерным префиксом, например, `-moz-border-radius` преобразуются по тому же принципу: дефис означает заглавную букву.

~~~
button.style.MozBorderRadius = '5px';
~~~

### Сброс стилей

Иногда нам нужно добавить свойство стиля, а потом, позже, убрать его.

Например, чтобы скрыть элемент, мы можем задать `elem.style.display = "none"`.

Затем мы можем удалить свойство `style.display`, чтобы вернуться к первоначальному состоянию.

~~~
<body>
  <div>Hello</div>
  <script>
    // если мы запустим этот код, <body> "мигнёт"
    document.body.style.display = "none"; // скрыть

    setTimeout(() => document.body.style.display = "", 1000); // возврат к нормальному состоянию
  </script>
</body>
~~~

Если мы установим в `style.display` пустую строку, то браузер применит CSS-классы и встроенные стили, как если бы такого свойства `style.display` вообще не было.

#### Полная перезапись `style.cssText`

Для задания нескольких стилей в одной строке используется специальное свойство `style.cssText`:

~~~
<div id="div">Button</div>

<script>
  // можем даже устанавливать специальные флаги для стилей, например, "important"
  div.style.cssText = `color: red !important;
    background-color: yellow;
    width: 100px;
    text-align: center;
  `;

  console.log(div.style.cssText);
</script>
~~~

Это свойство редко используется, потому что такое присваивание удаляет все существующие стили: оно не добавляет, а заменяет их. Но его можно использовать, к примеру, для новых элементов.

То же самое можно сделать установкой атрибута: `div.setAttribute('style', 'color: red...')`.

### Следите за единицами измерения

Мы должны устанавливать `10px`, а не просто `10` в свойство `elem.style.top`. Иначе это не сработает:

~~~
<body>
  <script>
    // не работает!
    document.body.style.margin = 20;
    console.log(document.body.style.margin); // '' (пустая строка, присваивание игнорируется)

    // сейчас добавим единицу измерения (px) - и заработает
    document.body.style.margin = '20px';
    console.log(document.body.style.margin); // 20px

    console.log(document.body.style.marginTop); // 20px
    console.log(document.body.style.marginLeft); // 20px
  </script>
</body>
~~~

Браузер «распаковывает» свойство `style.margin` в последних строках и выводит `style.marginLeft` и `style.marginTop` из него.

### Вычисленные стили: `getComputedStyle`

Итак, изменить стиль очень просто. Но как его прочитать?

Свойство `style` оперирует только значением атрибута `"style"`, без учёта CSS-каскада.

Поэтому, используя `elem.style`, мы не можем прочитать ничего, что приходит из классов CSS.

Например, здесь `style` не может видеть отступы:

~~~
<head>
  <style>
    body {
      color: red;
      margin: 5px;
    }
  </style>
</head>
<body>
  Красный текст
  <script>
    console.log(document.body.style.color); // пусто
    console.log(document.body.style.marginTop); // пусто
  </script>
</body>
~~~

…Но что, если нам нужно, скажем, увеличить отступ на `20px`? Для начала нужно его текущее значение получить.

Для этого есть метод: `getComputedStyle`.

Синтаксис:

~~~
getComputedStyle(element, [pseudo])
~~~

* `element` - элемент, значения для которого нужно получить
* `pseudo` - указывается, если нужен стиль псевдоэлемента, например `::before`. Пустая строка или отсутствие аргумента означают сам элемент.

Результат вызова – объект со стилями, похожий на `elem.style`, но с учётом всех CSS-классов.

~~~
<head>
  <style>
    body {
      color: red;
      margin: 5px;
    }
  </style>
</head>
<body>
  Красный текст
  <script>
    const computedStyle = getComputedStyle(document.body);
    console.log(computedStyle.marginTop); // 5px
    console.log(computedStyle.color); // rgb(255, 0, 0)
  </script>
</body>
~~~

#### Вычисленное (computed) и окончательное (resolved) значения

Есть две концепции в CSS:

* __Вычисленное (computed) значение__ – это то, которое получено после применения всех CSS-правил и CSS-наследования. Например, height:1em или font-size:125%.
* __Окончательное (resolved) значение__ – непосредственно применяемое к элементу. Значения `1em` или `125%` являются относительными. Браузер берёт вычисленное значение и делает все единицы измерения фиксированными и абсолютными, например, `height: 20px` или `font-size: 16px`. Для геометрических свойств разрешённые значения могут иметь плавающую точку, например, `width: 50.5px`.

Давным-давно `getComputedStyle` был создан для получения вычисленных значений, но оказалось, что окончательные значения гораздо удобнее, и стандарт изменился.

Так что, в настоящее время `getComputedStyle` фактически возвращает окончательное значение свойства, для геометрии оно обычно в пикселях.

#### `getComputedStyle` требует полное свойство!

Для правильного получения значения нужно указать точное свойство. Например: `paddingLeft`, `marginTop`, `borderTopWidth`. При обращении к сокращённому: `padding`, `margin`, `border` – правильный результат не гарантируется.

Например, если есть свойства `paddingLeft`/`paddingTop`, то что мы получим вызывая `getComputedStyle(elem).padding`? Ничего, или, может быть, «сгенерированное» значение из известных внутренних отступов? Стандарта для этого нет.

#### Стили, применяемые к посещённым `:visited` ссылкам, скрываются!

Посещённые ссылки могут быть окрашены с помощью псевдокласса `:visited`.

Но `getComputedStyle` не даёт доступ к этой информации, чтобы произвольная страница не могла определить, посещал ли пользователь ту или иную ссылку, проверив стили.

JavaScript не видит стили, применяемые с помощью `:visited`. Кроме того, в CSS есть ограничение, которое запрещает в целях безопасности применять к `:visited` CSS-стили, изменяющие геометрию элемента. Это гарантирует, что нет обходного пути для «злой» страницы проверить, была ли ссылка посещена и, следовательно, нарушить конфиденциальность.

~~~
<head>
  <style>
    a:link {
      background-color: rgb(255, 0, 0);
    }

    a:first-child {
      background-color: rgb(0, 128, 0);
    }

    a:visited {
      background-color: rgb(128, 0, 128);
    }
  </style>
</head>

<body>
  <a id="link" href="https://javascript.info/document">Link</a>
  <script>
    const computedStyle = getComputedStyle(link);
    console.log(computedStyle.backgroundColor); // rgb(0, 128, 0)
  </script>
</body>
~~~
