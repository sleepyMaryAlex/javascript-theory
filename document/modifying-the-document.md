# ?Modifying the document

### Создание элемента

DOM-узел можно создать двумя методами:

1. `document.createElement(tag)` - создаёт новый элемент с заданным тегом:

~~~
const div = document.createElement('div');
~~~

2. `document.createTextNode(text)` - создаёт новый текстовый узел с заданным текстом:

~~~
const textNode = document.createTextNode('А вот и я');
~~~

### Методы вставки

* `node.append(...nodes or strings)` – добавляет узлы или строки в конец `node`,
* `node.prepend(...nodes or strings)` – вставляет узлы или строки в начало `node`,
* `node.before(...nodes or strings)` –- вставляет узлы или строки до `node`,
* `node.after(...nodes or strings)` –- вставляет узлы или строки после `node`,
* `node.replaceWith(...nodes or strings)` –- заменяет `node` заданными узлами или строками.

Вот пример использования этих методов, чтобы добавить новые элементы в список и текст до/после него:

~~~
<ol id="ol">
  <li>0</li>
  <li>1</li>
  <li>2</li>
</ol>

<script>
  ol.before('before');
  ol.after('after');

  const liFirst = document.createElement('li');
  liFirst.innerHTML = 'prepend';
  ol.prepend(liFirst);

  const liLast = document.createElement('li');
  liLast.innerHTML = 'append';
  ol.append(liLast);
</script>
~~~

Итоговый список будет таким:

~~~
before
<ol id="ol">
  <li>prepend</li>
  <li>0</li>
  <li>1</li>
  <li>2</li>
  <li>append</li>
</ol>
after
~~~

Эти методы могут вставлять несколько узлов и текстовых фрагментов за один вызов.

Например, здесь вставляется строка и элемент:

~~~
<div id="div"></div>
<script>
  div.before('<p>Привет</p>', document.createElement('hr'));
</script>
~~~

Весь текст вставляется как текст.

~~~
<p>Привет</p>
<hr>
<div id="div"></div>
~~~

А что, если мы хотим вставить HTML именно «как html», со всеми тегами и прочим?

### `insertAdjacentHTML`/`Text`/`Element`

С этим может помочь другой, довольно универсальный метод: `elem.insertAdjacentHTML(where, html)`.

Первый параметр – это специальное слово, указывающее, куда по отношению к `elem` производить вставку. Значение должно быть одним из следующих:

* `"beforebegin"` – вставить `html` непосредственно перед `elem`,
* `"afterbegin"` – вставить `html` в начало `elem`,
* `"beforeend"` – вставить `html` в конец `elem`,
* `"afterend"` – вставить `html` непосредственно после `elem`.

Второй параметр – это HTML-строка, которая будет вставлена именно «как HTML».

~~~
<div id="div"></div>
<script>
  div.insertAdjacentHTML('beforebegin', '<p>Привет</p>');
</script>
~~~

У метода есть два брата:

* `elem.insertAdjacentText(where, text)` – такой же синтаксис, но строка `text` вставляется «как текст», вместо HTML.
* `elem.insertAdjacentElement(where, elem)` – такой же синтаксис, но вставляет элемент `elem`, созданный с помощью `createElement`.

Они существуют, в основном, чтобы унифицировать синтаксис. На практике часто используется только `insertAdjacentHTML`. Потому что для элементов и текста у нас есть методы `append`/`prepend`/`before`/`after` – их быстрее написать, и они могут вставлять как узлы, так и текст.

### Удаление узлов

Для удаления узла есть методы `node.remove()`.

~~~
<script>
  const div = document.createElement('div');
  div.innerHTML = "Вы прочитали важное сообщение.";
  document.body.append(div);
  setTimeout(() => div.remove(), 1000);
</script>
~~~

Если нам нужно переместить элемент в другое место – нет необходимости удалять его со старого.

Все методы вставки автоматически удаляют узлы со старых мест.

~~~
<div id="first">Первый</div>
<div id="second">Второй</div>
<script>
  second.after(first); // берёт #second и после него вставляет #first
</script>
~~~

### Клонирование узлов: `cloneNode`

Вызов `elem.cloneNode(true)` создаёт «глубокий» клон элемента – со всеми атрибутами и дочерними элементами. Если мы вызовем `elem.cloneNode(false)`, тогда клон будет без дочерних элементов.

~~~
<div id="div">
  <strong>Всем привет!</strong> Вы прочитали важное сообщение.
</div>

<script>
  const div2 = div.cloneNode(true);
  div.after(div2);
</script>
~~~

### `DocumentFragment`

`DocumentFragment` является специальным DOM-узлом, который служит обёрткой для передачи списков узлов.

Мы можем добавить к нему другие узлы, но когда мы вставляем его куда-то, он «исчезает», вместо него вставляется его содержимое.

Например, `getListContent` ниже генерирует фрагмент с элементами <li>, которые позже вставляются в <ul>:

~~~
<ul id="ul"></ul>

<script>
  function getListContent() {
    const fragment = new DocumentFragment();

    for (let i = 1; i <= 3; i++) {
      const li = document.createElement('li');
      li.append(i);
      fragment.append(li);
    }

    return fragment;
  }

  ul.append(getListContent());
</script>
~~~

`DocumentFragment` редко используется. Зачем добавлять элементы в специальный вид узла, если вместо этого мы можем вернуть массив узлов? Переписанный пример:

~~~
<ul id="ul"></ul>

<script>
  function getListContent() {
    const result = [];

    for (let i = 1; i <= 3; i++) {
      const li = document.createElement('li');
      li.append(i);
      result.push(li);
    }

    return result;
  }

  ul.append(...getListContent());
</script>
~~~

Мы упоминаем `DocumentFragment` в основном потому, что он используется в некоторых других областях, например, для элемента `template`, который мы рассмотрим гораздо позже.

### Устаревшие методы вставки/удаления

Эта информация помогает понять старые скрипты, но не нужна для новой разработки.

Есть несколько других, более старых, методов вставки и удаления, которые существуют по историческим причинам.

Сейчас уже нет причин их использовать, так как современные методы более гибкие и удобные.

* `parentElem.appendChild(node)` - добавляет `node` в конец дочерних элементов `parentElem`.
* `parentElem.insertBefore(node, nextSibling)` - вставляет `node` перед `nextSibling` в `parentElem`.
* `parentElem.replaceChild(node, oldChild)` - заменяет `oldChild` на `node` среди дочерних элементов `parentElem`.
* `parentElem.removeChild(node)` - удаляет `node` из `parentElem` (предполагается, что он родитель `node`).

Все эти методы возвращают вставленный/удалённый узел. Другими словами, `parentElem.appendChild(node)` вернёт `node`. Но обычно возвращаемое значение не используют, просто вызывают метод.

### Несколько слов о `document.write`

Есть ещё один, очень древний метод добавления содержимого на веб-страницу: `document.write`.

~~~
<p>Где-то на странице...</p>
<script>
  document.write('<b>Привет из JS</b>');
</script>
<p>Конец</p>
~~~

Вызов document.write(html) записывает html на страницу «прямо здесь и сейчас». Строка html может быть динамически сгенерирована, поэтому метод достаточно гибкий. Мы можем использовать JavaScript, чтобы создать полноценную веб-страницу и записать её в документ.

Этот метод пришёл к нам со времён, когда ещё не было ни DOM, ни стандартов… Действительно старые времена. Он всё ещё живёт, потому что есть скрипты, которые используют его.

В современных скриптах он редко встречается из-за следующего важного ограничения:

##### Вызов `document.write` работает только во время загрузки страницы.

Если вызвать его позже, то существующее содержимое документа затрётся.

~~~
<p>Через одну секунду содержимое этой страницы будет заменено...</p>
<script>
  // document.write через 1 секунду
  // вызов происходит после того, как страница загрузится, поэтому метод затирает содержимое
  setTimeout(() => document.write('<b>...Этим.</b>'), 1000);
</script>
~~~

Так что после того, как страница загружена, он уже непригоден к использованию, в отличие от других методов DOM, которые мы рассмотрели выше. Это его недостаток.

Есть и преимущество. Технически, когда `document.write` запускается во время чтения HTML браузером, и что-то пишет в документ, то браузер воспринимает это так, как будто это изначально было частью загруженного HTML-документа.

Поэтому он работает невероятно быстро, ведь при этом нет модификации DOM. Метод пишет прямо в текст страницы, пока DOM ещё в процессе создания.

Так что, если нам нужно динамически добавить много текста в HTML, и мы находимся на стадии загрузки, и для нас очень важна скорость, это может помочь. Но на практике эти требования редко сочетаются. И обычно мы можем увидеть этот метод в скриптах просто потому, что они старые.
