# ?Events

### Event concept

_Событие_ – это сигнал от браузера о том, что что-то произошло. Все DOM-узлы могут подавать такие сигналы (хотя события бывают и не только в DOM).

### Event types

Вот некоторые из наиболее распространенных типов событий:

* Mouse Events
* Drag Events
* Keyboard Events
* Form / Input Events
* Focus Events
* CSS Events
* Clipboard Events
* Page Load Events
* Window Events

##### Mouse Events

`click`	Событие происходит, когда пользователь нажимает на элемент
`contextmenu`	Событие происходит, когда пользователь щелкает правой кнопкой мыши элемент, чтобы открыть контекстное меню
`dblclick`	Событие происходит, когда пользователь дважды щелкает элемент
`mousedown`	Событие происходит, когда пользователь нажимает кнопку мыши над элементом
`mouseenter`	Событие происходит, когда указатель перемещается на элемент
`mouseleave`	Событие происходит, когда указатель перемещается за пределы элемента
`mousemove`	Событие происходит, когда указатель перемещается, когда он находится над элементом
`mouseover`	Событие происходит, когда указатель перемещается на элемент или на один из его дочерних элементов
`mouseout`	Событие происходит, когда пользователь перемещает указатель мыши за пределы элемента или одного из его дочерних элементов
`mouseup`	Событие происходит, когда пользователь отпускает кнопку мыши над элементом
`auxclick`	Событие происходит, когда кнопка указывающего устройства (любая неосновная кнопка) была нажата и отпущена на элементе
`pointerlockchange`	Событие происходит, когда pointer блокируется/разблокируется

index.html
~~~
<canvas id="test-canvas" width="600" height="480"></canvas>
~~~

script.js
~~~
const canv = document.querySelector("#test-canvas");
const ctx = canv.getContext("2d");

const CIRCLE_RAD = 16;

let x = 30;
let y = 30;

function draw() {
  ctx.fillStyle = "black";
  ctx.fillRect(0, 0, canv.width, canv.height);
  ctx.fillStyle = "#f00";
  ctx.beginPath();
  ctx.arc(x, y, CIRCLE_RAD, 0, Math.PI * 2, true);
  ctx.fill();
}

draw(ctx, canv);

canv.onclick = function () {
  canv.requestPointerLock();
};

document.addEventListener("pointerlockchange", lockStatusChange, false);

function lockStatusChange() {
  if (document.pointerLockElement === canv) {
    document.addEventListener("mousemove", updateCirclePosition, false);
  } else {
    document.removeEventListener("mousemove", updateCirclePosition, false);
  }
}

function updateCirclePosition(e) {
  x += e.movementX;
  y += e.movementY;

  if (x > canv.width + CIRCLE_RAD) {
    x = -CIRCLE_RAD;
  } else if (x < -CIRCLE_RAD) {
    x = canv.width + CIRCLE_RAD;
  }

  if (y > canv.height + CIRCLE_RAD) {
    y = -CIRCLE_RAD;
  } else if (y < -CIRCLE_RAD) {
    y = canv.height + CIRCLE_RAD;
  }

  requestAnimationFrame(function () {
    draw();
  });
}
~~~

`pointerlockerror` Событие происходит, когда блокировка pointer не удалась (по техническим причинам или из-за того, что разрешение было отклонено)
`select`	Событие происходит, когда выбирается какой-то текст. Событие `select` происходит только тогда, когда текст определяется в поле `input` с `type='text'` или в поле `textarea`
`wheel`	Событие происходит, когда колесо указывающего устройства вращается в любом направлении

##### Drag Events

`drag` Событие происходит при перетаскивании элемента
`dragend`	Событие происходит, когда пользователь закончил перетаскивать элемент
`dragenter`	Событие происходит, когда перетаскиваемый элемент попадает в цель перетаскивания
`dragleave`	Событие происходит, когда перетаскиваемый элемент покидает цель перетаскивания
`dragover` Событие происходит, когда перетаскиваемый элемент находится над целью перетаскивания
`dragstart`	Событие происходит, когда пользователь начинает перетаскивать элемент
`drop` Событие происходит, когда перетаскиваемый элемент отбрасывается на цель перетаскивания

##### Keyboard Events

`keydown`	Событие происходит, когда пользователь нажимает клавишу
`keypress` Событие происходит, когда пользователь нажимает клавишу
`keyup`	Событие происходит, когда пользователь отпускает клавишу

##### Form / Input Events

`change` Событие происходит, когда измененяется значение элементов формы. Возникает после потери элементом фокуса, т.е. после события `blur`
`input`	Событие происходит, когда элемент получает ввод пользователя
`reset`	Событие происходит, когда данные формы сбрасываются
`select` Событие происходит, когда текст в текущем элементе выделяется
`submit` Событие происходит, когда данные формы отправляются
`abort`	Событие происходит, когда загрузка изображения прерывается

##### Focus Events

`focus`	Событие происходит, когда элемент получает фокус
`blur`	Событие происходит, когда элемент теряет фокус
`focusin`	Событие происходит, когда элемент собирается получить фокус
`focusout` Событие происходит, когда элемент собирается потерять фокус

##### CSS Events

`animationend` Событие происходит, когда анимация CSS завершена
`animationiteration` Событие происходит при повторении CSS анимации
`animationstart` Событие происходит при запуске CSS анимации
`transitionend`	Событие происходит, когда переход CSS завершен

##### Clipboard Events

`copy` Событие происходит, когда пользователь копирует содержимое элемента
`cut`	Событие происходит, когда пользователь вырезает содержимое элемента
`paste`	Событие происходит, когда пользователь вставляет некоторый контент в элемент

##### Touch Events

> События касания аналогичны событиям мыши, за исключением того, что они поддерживают одновременные касания и в разных местах сенсорной поверхности.

`touchcancel` Событие происходит, когда касание прерывается
`touchend` Событие происходит, когда палец убирается с сенсорного экрана
`touchmove`	Событие возникает при перетаскивании пальца по экрану
`touchstart` Событие происходит, когда палец помещается на сенсорный экран

### Page Load Events

`DOMContentLoaded` – Событие происходит, когда браузер полностью загрузил HTML, было построено DOM-дерево, но внешние ресурсы, такие как картинки `<img>` и стили, могут быть ещё не загружены.
`load` – Событие происходит, когда браузер загрузил HTML и внешние ресурсы (картинки, стили и т.д.).
`beforeunload/unload` – Событие происходит, когда пользователь покидает страницу.

##### Window Events and Others

`abort`	Событие происходит, когда загрузка мультимедиа прерывается
`afterprint` Событие происходит, когда страница начала печать или если диалоговое окно печати было закрыто
`beforeprint`	Событие происходит перед печатью страницы
`canplay`	Событие возникает, когда браузер может начать воспроизведение мультимедиа (когда он буферизован достаточно, чтобы начать)
`canplaythrough` Событие возникает, когда браузер может воспроизводить мультимедиа без остановки для буферизации
`durationchange` Событие происходит при изменении продолжительности медиа
`ended`	Событие происходит, когда носитель подошел к концу (полезно для сообщений типа "спасибо за прослушивание")
`error`	Событие возникает при возникновении ошибки при загрузке внешнего файла
`fullscreenchange` Событие происходит, когда элемент отображается в полноэкранном режиме
`fullscreenerror`	Событие возникает, когда элемент не может отображаться в полноэкранном режиме
`hashchange` Событие происходит, когда были внесены изменения в якорную часть URL
`invalid`	Событие происходит, когда элемент недействителен
`loadeddata` Событие происходит при загрузке медиа-данных
`loadedmetadata` Событие происходит при загрузке метаданных (например, размеров и продолжительности)
`loadstart`	Событие происходит, когда браузер начинает поиск указанного носителя
`message`	Событие происходит, когда сообщение получено через источник события
`offline`	Событие происходит, когда браузер начинает работать в автономном режиме
`online` Событие происходит, когда браузер начинает работать в сети
`open` Событие происходит при открытии соединения с источником события
`pagehide` Событие происходит, когда пользователь уходит с веб страницы
`pageshow` Событие происходит, когда пользователь переходит на веб страницу
`pause`	Событие происходит, когда воспроизведение мультимедиа приостановлено пользователем или программно
`play` Событие возникает, когда носитель был запущен или больше не приостановлен
`playing`	Событие происходит, когда медиа воспроизводится после приостановки или остановки для буферизации
`popstate` Событие происходит при изменении истории окна
`progress` Событие происходит, когда браузер находится в процессе получения медиа данные (загрузка медиа)
`ratechange` Событие происходит при изменении скорости воспроизведения мультимедиа
`resize` Событие происходит при изменении размера представления документа
`scroll` Событие возникает при прокрутке полосы прокрутки элемента
`search` Событие возникает, когда пользователь что-то вводит в поле поиска (для `<input="search">`)
`seeked` Событие происходит, когда пользователь завершает перемещение/переход к новой позициив СМИ
`seeking`	Событие возникает, когда пользователь начинает перемещаться/пропускать новую позицию в СМИ
`show` Событие происходит, когда `<menu>` элемент отображается как контекстное меню
`stalled`	Событие возникает, когда браузер пытается получить данные мультимедиа, но данные недоступны
`storage`	Событие происходит при обновлении области веб хранилища
`suspend`	Событие происходит, когда браузер намеренно не получает медиаданные
`timeupdate` Событие происходит, когда позиция воспроизведения изменилась (например, когда пользователь перемотка вперед к другому месту в медиа)
`toggle` Событие возникает, когда пользователь открывает или закрывает элемент `<details>`
`volumechange` Событие возникает при изменении громкости носителя (включая настройкугромкость, чтобы "отключить звук")
`waiting`	Событие происходит, когда воспроизведение мультимедиа приостановлено, но ожидается, что оно возобновится (например, когда медиа останавливается для буферизации дополнительных данных)

### HTML DOM Event Properties and Methods

##### Mouse Event Properties and Methods

`altKey` Возвращает, указывает клавиша "ALT", была ли нажата при срабатывании события мыши
`button` Возвращает, какая кнопка мыши была нажата при срабатывании события мыши
`buttons`	Возвращает, какие кнопки мыши были нажаты при срабатывании события мыши
`clientX`	Возвращает горизонтальную координату указателя мыши относительно текущего окна, когда было инициировано событие мыши
`clientY`	Возвращает вертикальную координату указателя мыши относительно текущего окна, когда было инициировано событие мыши
`ctrlKey`	Возвращает, указывает "CTRL" клавиша была ли нажата при срабатывании события мыши
`screenX`	Возвращает горизонтальную координату указателя мыши относительно экрана при возникновении события
`screenY`	Возвращает вертикальную координату указателя мыши относительно экрана, когда событие было инициировано
`shiftKey` Возвращает, указывает "SHIFT" была ли нажата клавиша при возникновении события
`which`	Возвращает, какая кнопка мыши была нажата при срабатывании события мыши
`metaKey`	Возвращает, указывает "META" была ли нажата клавиша при срабатывании события
`offsetX`	Возвращает горизонтальную координату указателя мыши относительно положение края целевого элемента
`offsetY`	Возвращает вертикальную координату указателя мыши относительно положение края целевого элемента
`getModifierState()`	Возвращает массив, содержащий целевые диапазоны, на которые будет влиять вставка/удаление
`pageX`	Возвращает горизонтальную координату указателя мыши относительно документа при срабатывании события мыши
`pageY`	Возвращает вертикальную координату указателя мыши относительно документа, когда было инициировано событие мыши
`MovementX`	Возвращает горизонтальную координату указателя мыши относительно позиция последнего события перемещения мыши
`MovementY`	Возвращает вертикальную координату указателя мыши относительно позиция последнего события перемещения мыши
`relatedTarget`	Возвращает элемент, связанный с элементом, вызвавшим событие мыши

##### Keyboard Event Properties and Methods

`altKey` Возвращает, указывает "ALT", была ли нажата клавиша при срабатывании ключевого события
`charCode` Возвращает код символа Юникода клавиши, вызвавшей событие onkeypress
`code` Возвращает код ключа, вызвавшего событие
`ctrlKey`	Возвращает, указывает "CTRL" была ли нажата клавиша при срабатывании ключевого события
`key`	Возвращает значение ключа для ключа, представленного событием onkeydown или onkeyup
`keyCode`	Возвращает код символа Unicode для клавиши, вызвавшей событие onkeypress, или код клавиши Unicode для клавиши, вызвавшей нажатие клавиши или событие onkeyup
`location` Возвращает расположение клавиши на клавиатуре или устройстве
`metaKey`	Возвращает, указывает "meta" была ли нажата клавиша при срабатывании ключевого события
`isComposing`	Возвращает, создается ли состояние события или нет
`repeat` Возвращает, удерживается ли клавиша повторно или нет
`shiftKey` Возвращает, указывает была ли нажата клавиша "SHIFT" при срабатывании ключевого события
`which`	Возвращает код символа Unicode ключа, вызвавшего событие onkeypress, или код ключа Unicode ключа, вызвавшего событие 

##### Touch Event Properties and Methods

`altKey` Возвращает, указывает "ALT", была ли нажата клавиша при срабатывании ключевого события
`changeTouches`	Возвращает список всех сенсорных объектов, состояние которых изменилось между предыдущее касание и это касание
`clientX`	Возвращает горизонтальную координату указателя мыши относительно текущего окна, когда было инициировано событие мыши
`clientY`	Возвращает вертикальную координату указателя мыши относительно текущего окна, когда было инициировано событие мыши
`ctrlKey`	Возвращает, указывает "CTRL" была ли нажата клавиша при срабатывании ключевого события
`touches`	Возвращает список всех сенсорных объектов, которые в данный момент находятся в контакте с поверхностью
`metaKey`	Возвращает, указывает "meta" была ли нажата клавиша при срабатывании ключевого события
`targetTouches`	Возвращает список всех сенсорных объектов, которые контактируют с поверхность и где событие touchstart произошло на том же целевом элементе, что и текущий целевой элемент
`shiftKey` Возвращает, указывает "SHIFT" была ли нажата клавиша при срабатывании ключевого события Returns whether the "SHIFT" key was pressed when the key event was triggered

##### CSS Event Properties and Methods

`animationName`	Возвращает имя анимации
`elapsedTime`	Возвращает количество секунд, в течение которых была запущена анимация
`propertyName` Возвращает имя свойства CSS, связанного с анимацией или переходом
`pseudoElement`	Возвращает имя псевдоэлемента анимации или перехода

##### Clipboard Event Properties and Methods

`clipboardData`	Возвращает объект, содержащий данные, на которые влияет операция буфер обмена

##### Wheel Event Properties and Methods

`deltaX` Возвращает величину горизонтальной прокрутки колеса мыши (ось x)
`deltaY` Возвращает величину вертикальной прокрутки колеса мыши (ось y)
`deltaZ` Возвращает величину прокрутки колеса мыши для оси z
`deltaMode`	Возвращает число, представляющее единицу измерения для значений дельты (пиксели, строки или страницы)

##### Input Event Properties and Methods

`dataTransfer` Возвращает объект, содержащий перетаскиваемые/отбрасываемые данные, или вставлено/удалено
`data` Возвращает вставленные символы
`getTargetRanges()`	Возвращает массив, содержащий целевые диапазоны, на которые будет влиять вставка/удаление
`inputType`	Возвращает тип изменения (например, "вставка" или "удаление")
`isComposing`	Возвращает, создается ли состояние события или нет

##### Drag Event Properties and Methods

`dataTransfer` Возвращает объект, содержащий перетаскиваемые/отбрасываемые данные, или вставлено/удалено

##### Storage Event Properties and Methods

`key`	Возвращает ключ измененного элемента хранилища
`newValue` Возвращает новое значение измененного элемента хранения
`oldValue` Возвращает старое значение измененного элемента хранилища
`storageArea`	Возвращает объект, представляющий затронутый объект хранилища
`url`	Возвращает URL документа измененного элемента

##### Other Event Properties and Methods

`bubbles`	Возвращает, является ли конкретное событие восходящим или нетt
`cancelable` Возвращает, можно ли для события предотвратить действие по умолчанию
`composed` Возвращает, составлено ли событие или нет
`createEvent()`	Создает новое событие
`currentTarget`	Возвращает элемент, прослушиватели событий которого инициировали событие
`defaultPrevented` Возвращает, был ли вызван метод `preventDefault()` для события
`detail` Возвращает число, указывающее, сколько раз была нажата мышь
`eventPhase` Возвращает, какая фаза потока событий сейчас оценивается
`isTrusted`	Возвращает, является ли событие надежным
`lengthComputable`	Возвращает, можно ли вычислить длину прогресса
`loaded` Возвращает объем загруженной работы
`newURL` Возвращает URL документа после изменения хэша
`oldURL` Возвращает URL документа до изменения хэша
`onemptied`	Событие возникает, когда происходит что-то плохое и медиафайл неожиданно недоступен (например, неожиданно отключается)	 
`persisted`	Возвращает, была ли веб страница кэширована браузером
`preventDefault()` Отменяет событие, если оно может быть отменено, что означает, что действие по умолчанию, относящееся к событию, не произойдет
`relatedTarget`	Возвращает элемент, связанный с элементом, вызвавшим событие
`state`	Возвращает объект, содержащий копию записей истории
`stopImmediatePropagation()` Предотвращает вызов других слушателей того же события
`stopPropagation()`	Предотвращает дальнейшее распространение события во время потока событий
`target` Возвращает элемент, вызвавший событие
`timeStamp`	Возвращает время (в миллисекундах относительно эпохи), в которое было создано событие
`total`	Возвращает общий объем работы, которая будет загружена
`type` Возвращает имя события
`view` Возвращает ссылку на объект Window, в котором произошло событие

### Event listeners

Событию можно назначить обработчик, который сработает, как только событие произошло.

Есть несколько способов назначить событию обработчик. Сейчас мы их рассмотрим, начиная с самого простого.

1. Атрибут HTML: onclick="...".

~~~
<script>
  function countRabbits(n) {
    console.log('Rabbit ' + n);
  }
</script>

<input type="button" onclick="countRabbits(1)" value="Считать кроликов!">
~~~

Обратите внимание, атрибут находится в двойных кавычках.

> Как мы помним, атрибут HTML-тега не чувствителен к регистру, поэтому ONCLICK будет работать так же, как onClick и onCLICK… Но, как правило, атрибуты пишут в нижнем регистре: onclick.

2. DOM-свойство: elem.onclick = function.

~~~
<input id="elem" type="button" value="Нажми меня!">
<script>
  elem.onclick = function() {
  console.log('Спасибо');
};
</script>
~~~

Используйте `elem.onclick`, а не `elem.ONCLICK`, потому что DOM-свойства чувствительны к регистру.

Так как у элемента DOM может быть только одно свойство с именем onclick, то назначить более одного обработчика так нельзя.

В примере ниже назначение через JavaScript перезапишет обработчик из атрибута:

~~~
<input type="button" id="elem" onclick="alert('Было')" value="Нажми меня">
<script>
  elem.onclick = function() { // перезапишет существующий обработчик
    alert('Станет'); // выведется только это
  };
</script>
~~~

> Внутри обработчика события, неважно каким способом он назначен, this ссылается на текущий элемент, то есть на тот, на котором, как говорят, «висит» (т.е. назначен) обработчик.

3. Специальные методы: elem.addEventListener(event, handler[, phase]) для добавления, removeEventListener для удаления.

Фундаментальный недостаток описанных выше способов назначения обработчика – невозможность повесить несколько обработчиков на одно событие. Разработчики стандартов достаточно давно это поняли и предложили альтернативный способ назначения обработчиков при помощи специальных методов `addEventListener` и `removeEventListener`. Они свободны от указанного недостатка. Есть несколько типов событий, которые работают только через него, к примеру `transitionend`.

~~~
function handler() {
  console.log('Спасибо!');
}

input.addEventListener("click", handler);
~~~

Мы можем назначить обработчиком не только функцию, но и объект при помощи `addEventListener`.

~~~
<button id="elem">Нажми меня</button>

<script>
  elem.addEventListener('click', {
    handleEvent(event) {
      alert(event.type + " на " + event.currentTarget);
    }
  });
</script>
~~~

Как видим, если `addEventListener` получает объект в качестве обработчика, он вызывает `object.handleEvent(event)`, когда происходит событие.

Мы также можем использовать класс для этого:

~~~
<button id="elem">Нажми меня</button>

<script>
  class Menu {
    handleEvent(event) {
      switch(event.type) {
        case 'mousedown':
          elem.innerHTML = "Нажата кнопка мыши";
          break;
        case 'mouseup':
          elem.innerHTML += "...и отжата.";
          break;
      }
    }
  }

  let menu = new Menu();
  elem.addEventListener('mousedown', menu);
  elem.addEventListener('mouseup', menu);
</script>
~~~

### Event Phases

Три фазы прохода события:

* Фаза погружения (capturing phase) — событие сначала идёт сверху вниз.
* Фаза цели (target phase) — событие достигло целевого (исходного) элемента.
* Фаза всплытия (bubbling stage) — событие начинает всплывать.

##### Event propagation cycle

Предположим, что событие щелчка запускается для любого тега внутри HTML, затем это событие начинает переход сверху к этому тегу, и где бы оно ни встречалось с обработчиком для этого события, оно также срабатывает, пока не достигнет тега, в котором было вызвано событие, и возвращается обратно к тегу HTML, но необходимо отметить, что либо он будет запускать события на фазе погружения, либо на фазе всплытия.

##### Всплытие

Когда на элементе происходит событие, обработчики сначала срабатывают на нём, потом на его родителе, затем выше и так далее, вверх по цепочке предков.

~~~
<form onclick="alert('form')">FORM
  <div onclick="alert('div')">DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
~~~

> Почти все события срабатывают на стадии всплытия. Чтобы поймать событие на стадии перехвата, нужно использовать третий аргумент addEventListener.
> Бывают события, которые можно поймать только на стадии перехвата, а на стадии всплытия – нельзя… Например, таково событие фокусировки на элементе focus. Конечно, это большая редкость, такое исключение существует по историческим причинам.
>Обработчики, добавленные через on...-свойство, ничего не знают о стадии перехвата, а начинают работать со всплытия.

Всплытие идёт с «целевого» элемента прямо наверх. По умолчанию событие будет всплывать до элемента <html>, а затем до объекта document, а иногда даже до window, вызывая все обработчики на своём пути.

Но любой промежуточный обработчик может решить, что событие полностью обработано, и остановить распространение на любой фазе.

Для этого нужно вызвать метод `event.stopPropagation()`.

Например, здесь при клике на кнопку `<button>` обработчик `body.onclick` не сработает:

~~~
<body onclick="alert(`сюда всплытие не дойдёт`)">
  <button onclick="event.stopPropagation()">Кликни меня</button>
</body>
~~~

Но чтобы полностью остановить обработку, существует метод `event.stopImmediatePropagation()`. Он не только предотвращает всплытие, но и останавливает обработку событий на текущем элементе.

~~~
const button = document.querySelector('.button');

function handler(e) {
  e.stopImmediatePropagation();
}

button.addEventListener('click', handler);
button.addEventListener('click', () => console.log('event'));
~~~

_event.target_ - самый глубокий элемент, который вызывает событие, называется целевым элементом, и он доступен через event.target.

Отличия от this (=event.currentTarget):

* event.target – это «целевой» элемент, на котором произошло событие, в процессе всплытия он неизменен.
* this – это «текущий» элемент, до которого дошло всплытие, на нём сейчас выполняется обработчик.

##### Погружение

Ранее мы говорили только о всплытии, потому что другие стадии, как правило, не используются и проходят незаметно для нас.

Обработчики, добавленные через `on<event>`-свойство или через HTML-атрибуты, или через `addEventListener(event, handler)` с двумя аргументами, ничего не знают о фазе погружения, а работают только на 2-ой и 3-ей фазах.

Чтобы поймать событие на стадии погружения, нужно использовать третий аргумент capture вот так:

~~~
elem.addEventListener(..., ..., true);
~~~

Существуют два варианта значений опции capture:

* Если аргумент false (по умолчанию), то событие будет поймано при всплытии.
* Если аргумент true, то событие будет перехвачено при погружении.

Обратите внимание, что хоть и формально существует 3 фазы, 2-ую фазу («фазу цели»: событие достигло элемента) нельзя обработать отдельно, при её достижении вызываются все обработчики: и на всплытие, и на погружение.

~~~
<style>
  body * {
    margin: 10px;
    border: 1px solid blue;
  }
</style>

<form>FORM
  <div>DIV
    <p>P</p>
  </div>
</form>

<script>
  for(let elem of document.querySelectorAll('*')) {
    elem.addEventListener("click", e => alert(`Погружение: ${elem.tagName}`), true);
    elem.addEventListener("click", e => alert(`Всплытие: ${elem.tagName}`));
  }
</script>

// HTML => Body => Form => Div => P => P => Div => Form => Body => HTML
~~~

Существует свойство `event.eventPhase`, содержащее номер фазы, на которой событие было поймано (0 - none, 1 - capturing phase, 2 - target phase, 3 - bubbling phase).

### Default browser behavior

Многие события автоматически влекут за собой действие браузера.

Например:

* Клик по ссылке инициирует переход на новый URL.
* Нажатие на кнопку «отправить» в форме – отсылку её на сервер.
* Зажатие кнопки мыши над текстом и её движение в таком состоянии – инициирует его выделение.

Если мы обрабатываем событие в JavaScript, то зачастую такое действие браузера нам не нужно. К счастью, его можно отменить.

Есть два способа отменить действие браузера:

* Основной способ – это воспользоваться объектом event. Для отмены действия браузера существует стандартный метод `event.preventDefault()`.
* Если же обработчик назначен через `on<событие>` (не через `addEventListener`), то также можно вернуть `false` из обработчика.

~~~
<a href="/" onclick="return false">Нажми здесь</a>
или
<a href="/" onclick="event.preventDefault()">здесь</a>
~~~

> Обычно значение, которое возвращает обработчик события, игнорируется. Единственное исключение – это return false из обработчика, назначенного через `on<событие>`. В других случаях return не нужен, он никак не обрабатывается.

##### Опция «passive» для обработчика

Необязательная опция `passive: true` для `addEventListener` сигнализирует браузеру, что обработчик не собирается выполнять `preventDefault()`.

Почему это может быть полезно?

Есть некоторые события, как touchmove на мобильных устройствах (когда пользователь перемещает палец по экрану), которое по умолчанию начинает прокрутку, но мы можем отменить это действие, используя `preventDefault()` в обработчике.

Поэтому, когда браузер обнаружит такое событие, он должен для начала запустить все обработчики и после, если `preventDefault` не вызывается нигде, он может начать прокрутку. Это может вызвать ненужные задержки в пользовательском интерфейсе.

Опция `passive: true` сообщает браузеру, что обработчик не собирается отменять прокрутку. Тогда браузер начинает её немедленно, обеспечивая максимально плавный интерфейс, параллельно обрабатывая событие.

Для некоторых браузеров (Firefox, Chrome) опция passive по умолчанию включена в true для таких событий, как `touchstart` и `touchmove`.

##### event.defaultPrevented

Свойство `event.defaultPrevented` установлено в `true`, если действие по умолчанию было предотвращено, и `false`, если нет.

Рассмотрим практическое применение этого свойства:

~~~
<p>Правый клик вызывает меню документа (добавлена проверка event.defaultPrevented)</p>
<button id="elem">Правый клик вызывает меню кнопки и документа</button>

<script>
  elem.oncontextmenu = function(event) {
    alert("Контекстное меню кнопки");
  };

  document.oncontextmenu = function(event) {
    if (event.defaultPrevented) return;

    event.preventDefault(); // также отменяет действие по умолчанию и для дочернего элемента elem
    alert("Контекстное меню документа");
  };
</script>
~~~

Здесь мы проверяем в обработчике `document`, было ли отменено действие по умолчанию? Если да, тогда событие было обработано, и нам не нужно на него реагировать.

### Custom events

Пользовательские события можно создавать двумя способами:

* Использование `Event` конструктора
* Использование `CustomEvent` конструктора

Пользовательские события также можно создавать с помощью `document.createEvent`, но большинство методов, предоставляемых объектом, возвращаемым функцией, устарели.

##### Использование Event конструктора

~~~
const myEvent = new Event("myevent", {
  bubbles: true,
  cancelable: true,
  composed: false,
});
~~~

* Свойство `bubbles` указывает, должно ли событие всплывать. 
* Свойство `cancelable` указывает, должно ли событие быть отменено. Если для пользовательского события `cancelable` установлено на значение `false`, вызов `event.preventDefault()` не будет выполнять никаких действий.
* Свойство `composed` указывает, должно ли событие всплывать из теневой модели DOM (созданной при использовании веб-компонентов) в реальную модель DOM. Если для `bubbles` задано значение `false`, значение этого свойства не будет иметь значения, поскольку вы явно указываете событию, что событие всплывать не будет. Однако, если вы хотите отправить пользовательское событие в веб-компонент и прослушать его в родительском элементе в реальном DOM, тогда для свойства `composed` необходимо установить значение `true`.

##### Использование CustomEvent конструктора

При создании событий с помощью конструктора Event мы были ограничены тем, что не можем передавать данные через событие слушателю. Здесь любые данные, которые необходимо передать слушателю, можно передать в свойстве `detail`, которое создается при инициализации события.

~~~
const button = document.querySelector(".button");
const body = document.querySelector(".body");

const catFound = new CustomEvent("animalfound", {
  detail: {
    name: "cat",
  },
  bubbles: true,
  cancelable: true,
});

body.addEventListener("animalfound", () => console.log("bubbles"));

button.addEventListener("animalfound", (e) => {
  e.preventDefault();
  console.log(e.detail.name);
});

console.log(
  button.dispatchEvent(catFound)
    ? "Event was not cancelled"
    : "Event was cancelled"
);

// cat
// bubbles
// Event was cancelled
~~~

После создания событий вы должны иметь возможность отправлять их. События могут быть отправлены любому объекту, который расширяет `EventTarget`, и они включают все элементы HTML, документ, окно и т. д.

Вы можете отправлять пользовательские события следующим образом:

~~~
button.dispatchEvent(catFound);
~~~

### Delegation

Если у нас есть много элементов, события на которых нужно обрабатывать похожим образом, то вместо того, чтобы назначать обработчик каждому, мы ставим один обработчик на их общего предка. Из него можно получить целевой элемент `event.target`, понять на каком именно потомке произошло событие и обработать его.

Наша задача – реализовать подсветку ячейки <td> при клике.

Вместо того, чтобы назначать обработчик onclick для каждой ячейки <td> (их может быть очень много) – мы повесим «единый» обработчик на элемент <table>.

~~~
<table class="table">
  <tr>
    <th>Company</th>
    <th>Contact</th>
    <th>Country</th>
  </tr>
  <tr>
    <td>Alfreds Futterkiste</td>
    <td>Maria Anders</td>
    <td>Germany</td>
  </tr>
  <tr>
    <td>Centro comercial Moctezuma</td>
    <td>Francisco Chang</td>
    <td>Mexico</td>
  </tr>
</table>
~~~

~~~
let selectedTd;
const table = document.querySelector('.table');

table.onclick = function(event) {
  let td = event.target.closest('td');
  
  if (!td) return;

  highlight(td);
};

function highlight(td) {
  if (selectedTd) {
    selectedTd.style.color = 'black';
  }
  selectedTd = td;
  selectedTd.style.color = 'red';
}
~~~

Разберём пример:

* Метод `elem.closest(selector)` обходит элемент и его родителей (направляясь к корню документа), пока не найдет узел, соответствующий указанному селектору CSS, иначе `null`.
* Если `event.target` не содержится внутри элемента `<td>`, то вызов вернёт `null`, и ничего не произойдёт.
* А если это так, то подсвечиваем его.

##### Event delegation benefits and drawbacks

Преимущества:

* Упрощает процесс инициализации и экономит память: не нужно вешать много обработчиков.
* Меньше кода: при добавлении и удалении элементов не нужно ставить или снимать обработчики.
* Удобство изменений DOM: можно массово добавлять или удалять элементы путём изменения `innerHTML` и ему подобных.

Недостатки:

* Другим недостатком является то, что делегирование событий может плохо работать с некоторыми событиями, которые не всплывают вверх по дереву DOM, такими как `focus`, `blur` или `load`. Также, низкоуровневые обработчики не должны вызывать `event.stopPropagation()`.
* Делегирование создаёт дополнительную нагрузку на браузер, ведь обработчик запускается, когда событие происходит в любом месте контейнера, не обязательно на элементах, которые нам интересны. Но обычно эта нагрузка настолько пустяковая, что её даже не стоит принимать во внимание.