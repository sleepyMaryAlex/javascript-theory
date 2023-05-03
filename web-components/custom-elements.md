# ?Custom elements

Мы можем создавать пользовательские HTML-элементы, описываемые нашим классом, со своими методами и свойствами, событиями и так далее.

Существует два вида пользовательских элементов:

1. Автономные пользовательские элементы – «полностью новые» элементы, расширяющие абстрактный класс `HTMLElement`.
2. Пользовательские встроенные элементы – элементы, расширяющие встроенные, например кнопку `HTMLButtonElement` и т.п.

Чтобы создать пользовательский элемент, нам нужно сообщить браузеру ряд деталей о нём: как его показать, что делать, когда элемент добавляется или удаляется со страницы и т.д.

Это делается путём создания класса со специальными методами:

~~~
class MyElement extends HTMLElement {
  constructor() {
    super();
    // элемент создан
  }

  connectedCallback() {
    // браузер вызывает этот метод при добавлении элемента в документ
    // (может вызываться много раз, если элемент многократно добавляется/удаляется)
  }

  disconnectedCallback() {
    // браузер вызывает этот метод при удалении элемента из документа
    // (может вызываться много раз, если элемент многократно добавляется/удаляется)
  }

  static get observedAttributes() {
    return [/* массив имён атрибутов для отслеживания их изменений */];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // вызывается при изменении одного из перечисленных выше атрибутов
  }

  adoptedCallback() {
    // вызывается, когда элемент перемещается в новый документ
    // (происходит в document.adoptNode, используется очень редко)
  }

  // у элемента могут быть ещё другие методы и свойства
}
~~~

После этого нам нужно зарегистрировать элемент:

~~~
// сообщим браузеру, что <my-element> обслуживается нашим новым классом
customElements.define("my-element", MyElement);
~~~

Теперь для любых HTML-элементов с тегом `<my-element>` создаётся экземпляр `MyElement` и вызываются вышеупомянутые методы. Также, чтобы создать элемент, мы можем использовать `document.createElement('my-element'); ...` в JavaScript.

> Имя пользовательского элемента должно содержать дефис -, например, `my-element` и `super-button` – валидные имена, а `myelement` – нет. Это чтобы гарантировать отсутствие конфликтов имён между встроенными и пользовательскими элементами HTML.

### Пример: «time-formatted»

Например, элемент `<time>` уже существует в HTML для даты/времени. Но сам по себе он не выполняет никакого форматирования. Давайте создадим элемент `<time-formatted>`, который отображает время в удобном формате с учётом языка:

~~~
<body>
  <time-formatted datetime="2019-12-01" year="numeric" month="long" day="numeric" hour="numeric" minute="numeric"
    second="numeric" time-zone-name="short"></time-formatted>
  <script>
    class TimeFormatted extends HTMLElement {
      connectedCallback() {
        const date = new Date(this.getAttribute("datetime") || Date.now());

        this.innerHTML = new Intl.DateTimeFormat("default", {
          year: this.getAttribute("year") || undefined,
          month: this.getAttribute("month") || undefined,
          day: this.getAttribute("day") || undefined,
          hour: this.getAttribute("hour") || undefined,
          minute: this.getAttribute("minute") || undefined,
          second: this.getAttribute("second") || undefined,
          timeZoneName: this.getAttribute("time-zone-name") || undefined,
        }).format(date);
      }
    }

    customElements.define("time-formatted", TimeFormatted);
  </script>

</body>

// December 1, 2019 at 1:00:00 AM GMT+1
~~~

##### Обновление пользовательских элементов

Если браузер сталкивается с элементами `<time-formatted>` до `customElements.define`, то это не ошибка. Но элемент пока неизвестен, как и любой нестандартный тег.

Такие «неопределённые» элементы могут быть стилизованы с помощью CSS селектора `:not(:defined)`.

Когда вызывается `customElements.define`, они «обновляются»: для каждого создаётся новый экземпляр `TimeFormatted` и вызывается `connectedCallback`. Они становятся `:defined`.

Чтобы получить информацию о пользовательских элементах, есть следующие методы:

* `customElements.get(name)` – возвращает класс пользовательского элемента с указанным именем `name`,
* `customElements.whenDefined(name)` – возвращает промис, который переходит в состояние «успешно выполнен» (без значения), когда определён пользовательский элемент с указанным именем `name`.

##### Рендеринг происходит в `connectedCallback`, не в `constructor`

Причина проста: когда вызывается `constructor`, делать это слишком рано. Экземпляр элемента создан, но на этом этапе браузер ещё не обработал/назначил атрибуты: вызовы `getAttribute` вернули бы `null`. Так что мы не можем рендерить здесь.

Кроме того, если подумать, это лучше с точки зрения производительности – отложить работу до тех пор, пока она действительно не понадобится.

### Наблюдение за атрибутами

В текущей реализации `<time-formatted>` после того, как элемент отрендерился, дальнейшие изменения атрибутов не дают никакого эффекта. 

~~~
document.querySelector('time-formatted').setAttribute('month', 'numeric'); // December 1, 2019 at 1:00:00 AM GMT+1
~~~

Это странно для HTML-элемента. Обычно, когда мы изменяем атрибут, например `a.href`, мы ожидаем, что изменение будет видно сразу. Так что давайте исправим это.

Мы можем наблюдать за атрибутами, поместив их список в статический геттер `observedAttributes()`. При изменении таких атрибутов вызывается `attributeChangedCallback`. Он срабатывает не для любого атрибута по соображениям производительности.

Вот новый `<time-formatted>`, который автоматически обновляется при изменении атрибутов:

~~~
<body>
  <script>
    class TimeFormatted extends HTMLElement {
      render() {
        const date = new Date(this.getAttribute('datetime') || Date.now());

        this.innerHTML = new Intl.DateTimeFormat("default", {
          year: this.getAttribute('year') || undefined,
          month: this.getAttribute('month') || undefined,
          day: this.getAttribute('day') || undefined,
          hour: this.getAttribute('hour') || undefined,
          minute: this.getAttribute('minute') || undefined,
          second: this.getAttribute('second') || undefined,
          timeZoneName: this.getAttribute('time-zone-name') || undefined,
        }).format(date);
      }

      connectedCallback() {
        if (!this.rendered) {
          this.render();
          this.rendered = true;
        }
      }

      static get observedAttributes() {
        return ['datetime', 'year', 'month', 'day', 'hour', 'minute', 'second', 'time-zone-name'];
      }

      attributeChangedCallback(name, oldValue, newValue) {
        this.render();
      }
    }

    customElements.define("time-formatted", TimeFormatted);
  </script>

  <time-formatted id="elem" datetime="2019-12-01" year="numeric" month="long" day="numeric" hour="numeric"
    minute="numeric" second="numeric" time-zone-name="short"></time-formatted>

  <script>
    setInterval(() => elem.setAttribute('month', 'numeric'), 1000); // 12/1/2019, 1:00:00 AM GMT+1
  </script>
</body>
~~~

При изменении атрибута, указанного в `observedAttributes()`, вызывается `attributeChangedCallback`.

### Порядок рендеринга

Когда HTML-парсер строит DOM, элементы обрабатываются друг за другом, родители до детей. Например, если у нас есть `<outer><inner></inner></outer>`, то элемент `<outer>` создаётся и включается в DOM первым, а затем `<inner>`.

Это приводит к важным последствиям для пользовательских элементов.

Например, если пользовательский элемент пытается получить доступ к `innerHTML` в `connectedCallback`, он ничего не получает:

~~~
<script>
customElements.define('user-info', class extends HTMLElement {
  connectedCallback() {
    console.log(this.innerHTML); // пусто (*)
  }
});
</script>

<user-info>Джон</user-info>
~~~

Если вы запустите это, `console.log` будет пуст.

Это происходит именно потому, что на этой стадии ещё не существуют дочерние элементы, DOM не завершён. HTML-парсер подключил пользовательский элемент `<user-info>` и теперь собирается перейти к его дочерним элементам, но пока не сделал этого.

Если мы хотим передать информацию в пользовательский элемент, мы можем использовать атрибуты. Они доступны сразу.

Или, если нам действительно нужны дочерние элементы, мы можем отложить доступ к ним, используя `setTimeout` с нулевой задержкой.

Это работает:

~~~
<script>
customElements.define('user-info', class extends HTMLElement {
  connectedCallback() {
    setTimeout(() => console.log(this.innerHTML)); // Джон (*)
  }
});
</script>

<user-info>Джон</user-info>
~~~

Теперь `console.log` в строке `(*)` показывает «Джон», поскольку мы запускаем его асинхронно, после завершения парсинга HTML. Мы можем обработать дочерние элементы при необходимости и завершить инициализацию.

С другой стороны, это решение также не идеально. Если вложенные пользовательские элементы тоже используют `setTimeout` для инициализации, то они встают в очередь: первым запускается внешний `setTimeout`, а затем внутренний.

Так что внешний элемент завершает инициализацию раньше внутреннего.

Продемонстрируем это на примере:

~~~
<script>
customElements.define('user-info', class extends HTMLElement {
  connectedCallback() {
    console.log(`${this.id} connected.`);
    setTimeout(() => console.log(`${this.id} initialized.`));
  }
});
</script>

<user-info id="outer">
  <user-info id="inner"></user-info>
</user-info>

// outer connected.
// inner connected.
// outer initialized.
// inner initialized.
~~~

Мы ясно видим, что внешний элемент `outer` завершает инициализацию до внутреннего `inner`.

Нет встроенного колбэка, который срабатывает после того, как вложенные элементы готовы. Если нужно, мы можем реализовать подобное самостоятельно. Например, внутренние элементы могут отправлять события наподобие `initialized`, а внешние могут слушать и реагировать на них.

### Модифицированные встроенные элементы

Новые элементы, которые мы создаём, такие как `<time-formatted>`, не имеют связанной с ними семантики. Они не известны поисковым системам, а устройства для людей с ограниченными возможностями не могут справиться с ними.

Но такие вещи могут быть важны. Например, поисковой системе было бы интересно узнать, что мы показываем именно время. А если мы делаем специальный вид кнопки, почему не использовать существующую функциональность `<button>`?

Мы можем расширять и модифицировать встроенные HTML-элементы, наследуя их классы.

Например, кнопки `<button>` являются экземплярами класса `HTMLButtonElement`, давайте построим элемент на его основе.

1. Унаследуем `HTMLButtonElement` нашим классом.

2. Предоставим третий аргумент в `customElements.define`, указывающий тег.

Бывает, что разные теги имеют одинаковый DOM-класс, поэтому указание тега необходимо.

3. В конце, чтобы использовать наш пользовательский элемент, вставим обычный тег `<button>`, но добавим к нему `is="hello-button"`.

Вот пример:

~~~
<body>
  <script>
    class HelloButton extends HTMLButtonElement {
      constructor() {
        super();
        this.addEventListener('click', () => console.log("Hello!"));
      }
    }
    customElements.define('hello-button', HelloButton, { extends: 'button' });
  </script>

  <button is="hello-button">Click</button>

  <button is="hello-button" disabled>Disabled</button>
</body>
~~~

Наша новая кнопка расширяет встроенную. Так что она сохраняет те же стили и стандартные возможности, наподобие атрибута `disabled`.
