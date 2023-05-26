# ?Finding DOM elements

### `document.getElementById` или просто `id`

Если у элемента есть атрибут `id`, то мы можем получить его вызовом `document.getElementById(id)`, где бы он ни находился.

Также есть глобальная переменная с именем, указанным в `id`:

~~~
<div id="elem">
  <div id="elem-content">Элемент</div>
</div>

<script>
  // elem - ссылка на элемент с id="elem"
  elem.style.background = 'red';

  // внутри id="elem-content" есть дефис, так что такой id не может служить именем переменной
  // ...но мы можем обратиться к нему через квадратные скобки: window['elem-content']
  window['elem-content'].style.color = 'green';
</script>
~~~

…Но это только если мы не объявили в JavaScript переменную с таким же именем, иначе она будет иметь приоритет.

Лучше не использовать такие глобальные переменные для доступа к элементам. В реальной жизни лучше использовать `document.getElementById`.

Значение `id` должно быть уникальным. В документе может быть только один элемент с данным `id`.

Метод `getElementById` можно вызвать только для объекта `document`. Он осуществляет поиск по `id` по всему документу.

### `querySelectorAll`

Самый универсальный метод поиска – это `elem.querySelectorAll(css)`, он возвращает все элементы внутри `elem`, удовлетворяющие данному CSS-селектору.

Этот метод действительно мощный, потому что можно использовать любой CSS-селектор.

Например, `document.querySelectorAll(':hover')` вернёт коллекцию (в порядке вложенности: от внешнего к внутреннему) из текущих элементов под курсором мыши.

~~~
<ul>
  <li>Этот</li>
  <li>тест</li>
</ul>
<ul>
  <li>полностью</li>
  <li>пройден</li>
</ul>

<script>
  const elements = document.querySelectorAll('ul > li:last-child');

  for (const elem of elements) {
    console.log(elem.innerHTML); // "тест", "пройден"
  }

  setTimeout(() => console.log(document.querySelectorAll(':hover')), 1000); // [html, body, ul, li]
</script>
~~~

### `querySelector`

Метод `elem.querySelector(css)` возвращает первый элемент, соответствующий данному CSS-селектору.

Иначе говоря, результат такой же, как при вызове `elem.querySelectorAll(css)[0]`, но он сначала найдёт все элементы, а потом возьмёт первый, в то время как `elem.querySelector` найдёт только первый и остановится. Это быстрее, кроме того, его короче писать.

### `matches`

Предыдущие методы искали по DOM.

Метод `elem.matches(css)` ничего не ищет, а проверяет, удовлетворяет ли `elem` CSS-селектору, и возвращает `true` или `false`.

Этот метод удобен, когда мы перебираем элементы (например, в массиве или в чём-то подобном) и пытаемся выбрать те из них, которые нас интересуют.

~~~
<a href="http://example.com/file.zip">...</a>
<a href="http://ya.ru">...</a>

<script>
  // может быть любая коллекция вместо document.body.children
  for (const elem of document.body.children) {
    if (elem.matches('a[href$="zip"]')) {
      console.log("Ссылка на архив: " + elem.href );
    }
  }
</script>
~~~

### `closest`

Предки элемента – родитель, родитель родителя, его родитель и так далее. Вместе они образуют цепочку иерархии от элемента до вершины.

Метод `elem.closest(css)` ищет ближайшего предка, который соответствует CSS-селектору. Сам элемент также включается в поиск.

Другими словами, метод `closest` поднимается вверх от элемента и проверяет каждого из родителей. Если он соответствует селектору, поиск прекращается. Метод возвращает либо предка, либо `null`, если такой элемент не найден.

~~~
<h1>Содержание</h1>

<div class="contents">
  <ul class="book">
    <li class="chapter">Глава 1</li>
    <li class="chapter">Глава 2</li>
  </ul>
</div>

<script>
  const chapter = document.querySelector('.chapter'); // LI

  console.log(chapter.closest('.book')); // UL
  console.log(chapter.closest('.contents')); // DIV

  console.log(chapter.closest('h1')); // null (потому что h1 - не предок)
</script>
~~~

### getElementsBy*

Существуют также другие методы поиска элементов по тегу, классу и так далее.

На данный момент, они скорее исторические, так как `querySelector` более чем эффективен.

Здесь мы рассмотрим их для полноты картины, также вы можете встретить их в старом коде.

* `elem.getElementsByTagName(tag)` ищет элементы с данным тегом и возвращает их коллекцию. Передав `"*"` вместо тега, можно получить всех потомков.
* `elem.getElementsByClassName(className)` возвращает элементы, которые имеют данный CSS-класс.
* `document.getElementsByName(name)` возвращает элементы с заданным атрибутом `name`. Очень редко используется.

~~~
<body id="body">
  <label>
    <input type="radio" name="age" value="young" class="input"> младше 18
  </label>
  <label>
    <input type="radio" name="age" value="mature" class="input"> от 18 до 50
  </label>
  <label>
    <input type="radio" name="age" value="senior" class="input"> старше 60
  </label>

  <script>
    console.log(body.getElementsByTagName('input')); // HTMLCollection(3)
    console.log(document.getElementsByName('age')); // NodeList(3)
    console.log(body.getElementsByClassName('input')); // HTMLCollection(3)
  </script>
</body>
~~~

##### Все методы "getElementsBy*" возвращают живую коллекцию. Напротив, `querySelectorAll` возвращает статическую коллекцию. Это похоже на фиксированный массив элементов.

~~~
<div>First div</div>

<script>
  const divs = document.querySelectorAll('div');
  const divsByTagName = document.getElementsByTagName('div');
</script>

<div>Second div</div>

<script>
  console.log(divs.length); // 1
  console.log(divsByTagName.length); // 2
</script>
~~~

Итого:

| Метод	| Ищет по...	| Ищет внутри элемента? |	Возвращает живую коллекцию? |
|---|---|---|---|
| `querySelector`	| CSS-selector | ✔	| - |
| `querySelectorAll` | CSS-selector | ✔ | - |
| `getElementById`| id | - | - |
| `getElementsByName`	| name | - | ✔ |
| `getElementsByTagName`	| tag or '*' | ✔| ✔ |
| `getElementsByClassName` | class| ✔ | ✔ |
