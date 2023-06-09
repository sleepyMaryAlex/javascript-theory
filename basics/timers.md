# ?Timers

### setTimeout
Метод `setTimeout` — это встроенная функция JavaScript, устанавливающая таймер обратного отсчета (в миллисекундах) для выполнения функции обратного вызова по завершении заданного времени.

`const timerId = setTimeout(func|code, [delay], [arg1], [arg2], ...);`

Параметры:

* func|code
Функция или строка кода для выполнения. Обычно это функция. По историческим причинам можно передать и строку кода, но это не рекомендуется.
* delay
Задержка перед запуском в миллисекундах (1000 мс = 1 с). Значение по умолчанию – 0.
* arg1, arg2…
Аргументы, передаваемые в функцию.

~~~
function sayHi(phrase, who) {
  alert( phrase + ', ' + who );
}

setTimeout(sayHi, 1000, "Привет", "Джон"); // Привет, Джон
~~~

Если первый аргумент является строкой, то JavaScript создаст из неё функцию.

Это также будет работать:

`setTimeout("alert('Привет')", 1000);`

Но использование строк не рекомендуется. Вместо этого используйте функции.

> Передавайте функцию, но не запускайте её.

Вызов `setTimeout` возвращает «идентификатор таймера» `timerId`, который можно использовать для отмены дальнейшего выполнения.

~~~
const timerId = setTimeout(...);
clearTimeout(timerId);
~~~

В коде ниже планируем вызов функции и затем отменяем его (просто передумали). В результате ничего не происходит:

~~~
const timerId = setTimeout(() => console.log("ничего не происходит"), 1000);
console.log(timerId); // идентификатор таймера

clearTimeout(timerId);
console.log(timerId); // тот же идентификатор (не принимает значение null после отмены)
~~~

Как мы видим из вывода `console`, в браузере идентификатором таймера является число. В других средах это может быть что-то ещё. Например, Node.js возвращает объект таймера с дополнительными методами.

__setTimeout с нулевой задержкой__: `setTimeout(func, 0)` или просто `setTimeout(func)`.

Это планирует вызов func настолько быстро, насколько это возможно. Но планировщик будет вызывать функцию только после завершения выполнения текущего кода.

Так вызов функции будет запланирован сразу после выполнения текущего кода.

Например, этот код выводит «Привет» и затем сразу «Мир»:

~~~
setTimeout(() => console.log("Мир"));

console.log("Привет");
~~~

В браузере есть ограничение на то, как часто внутренние счётчики могут выполняться. В стандарте HTML5 говорится: «после пяти вложенных таймеров интервал должен составлять не менее четырёх миллисекунд.».

~~~
const start = Date.now();
const times = [];

setTimeout(function run() {
  times.push(Date.now() - start); // запоминаем задержку от предыдущего вызова

  if (start + 100 < Date.now()) {
    console.log(times);
  } else {
    setTimeout(run);
  }
});

// Пример вывода
// [ 1, 1, 1, 1, 9, 15, 20, 24, 30, 35, 40, 45, 50, 55, 59, 64, 70, 75, 80, 85, 90, 95, 100 ]
~~~

Первый таймер запускается сразу (как и указано в спецификации), а затем задержка вступает в игру, и мы видим 9, 15, 20, 24.... Аналогичное происходит при использовании `setInterval` вместо `setTimeout`.

Этого ограничения нет в серверном JavaScript. Там есть и другие способы планирования асинхронных задач. Например, `setImmediate` для Node.js. Так что это ограничение относится только к браузерам.

~~~
const start = Date.now();
const times = [];

setImmediate(function run() {
  times.push(Date.now() - start); // запоминаем задержку от предыдущего вызова

  if (start + 100 < Date.now()) {
    console.log(times);
  } else {
    setImmediate(run);
  }
});

// Пример вывода
// [ 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1... ]
~~~

Как отменить все `setTimeout` за раз? Так как возвращаемое значение `setTimeout` в браузере - это число, то можно перебрать числа в цикле:

~~~
setTimeout(() => console.log("timeout1"), 200);
setTimeout(() => console.log("timeout2"), 300);
setTimeout(() => console.log("timeout3"), 400);
setTimeout(() => console.log("timeout4"), 500);
setTimeout(() => {
  for (let i = 0; i < 100; i++) {
    clearTimeout(i);
  }
}, 300);

// timeout1 timeout2
~~~

### setInterval

Метод `setInterval` используется для повторного выполнения функции по истечении определенного периода времени.

Метод `setInterval` имеет такой же синтаксис как `setTimeout`.

Все аргументы имеют такое же значение. Но отличие этого метода от `setTimeout` в том, что функция запускается не один раз, а периодически через указанный интервал времени.

Чтобы остановить дальнейшее выполнение функции, необходимо вызвать `clearInterval(timerId)`.

Следующий пример выводит сообщение каждые 2 секунды. Через 5 секунд вывод прекращается:

~~~
const timerId = setInterval(() => console.log("tick"), 2000);

// остановить вывод через 5 секунд
setTimeout(() => {
  clearInterval(timerId);
  console.log("stop");
}, 5000);
~~~

> Во время показа alert время тоже идёт. В большинстве браузеров, включая Chrome и Firefox, внутренний счётчик продолжает тикать во время показа alert/confirm/prompt.

### Nested setTimeout vs setInterval

Есть два способа запускать что-то регулярно.

Один из них `setInterval`. Другим является вложенный `setTimeout`. Например:

~~~
let timerId = setInterval(() => console.log('tick'), 2000);

или

let timerId = setTimeout(function tick() {
  console.log("tick");
  timerId = setTimeout(tick, 2000);
}, 2000);
~~~

Метод `setTimeout` выше планирует следующий вызов прямо после окончания текущего.

Вложенный `setTimeout` – более гибкий метод, чем `setInterval`. С его помощью последующий вызов может быть задан по-разному в зависимости от результатов предыдущего.

Например:

~~~
let delay = 5000;

let timerId = setTimeout(function request() {
  // ...отправить запрос...

  if (ошибка запроса из-за перегрузки сервера) {
    // увеличить интервал для следующего запроса
    delay *= 2;
  }
  timerId = setTimeout(request, delay);
}, delay);
~~~

А если функции, которые мы планируем, ресурсоёмкие и требуют времени, то мы можем измерить время, затраченное на выполнение, и спланировать следующий вызов раньше или позже.

Вложенный `setTimeout` позволяет задать задержку между выполнениями более точно, чем `setInterval`.

Сравним два фрагмента кода:

~~~
let i = 1;
setInterval(function () {
  func(i);
}, 100);

и

let i = 1;
setTimeout(function run() {
  func(i);
  setTimeout(run, 100);
}, 100);
~~~

Реальная задержка между вызовами `func` с помощью `setInterval` меньше, чем указано в коде!

Это нормально, потому что время, затраченное на выполнение `func`, использует часть заданного интервала времени.

Вполне возможно, что выполнение `func` будет дольше, чем мы ожидали, и займёт более 100 мс.

В данном случае движок ждёт окончания выполнения `func` и затем проверяет планировщик и, если время истекло, немедленно запускает его снова.

Вложенный `setTimeout` гарантирует фиксированную задержку (здесь 100 мс). Это потому, что новый вызов планируется в конце предыдущего.

### requestAnimationFrame

`window.requestAnimationFrame` указывает браузеру на то, что вы хотите произвести анимацию, и просит его запланировать перерисовку на следующем кадре анимации. В качестве параметра метод получает функцию, которая будет вызвана перед перерисовкой.

> Примечание: Ваш callback метод сам должен вызвать `requestAnimationFrame()` иначе анимация остановится.

Вы должны вызывать этот метод всякий раз, когда готовы обновить анимацию на экране, чтобы запросить планирование анимации. Обычно запросы происходят 60 раз в секунду, но чаще всего совпадают с частотой обновления экрана.

Возвращаемое значение `requestID` — длинное целое, являющееся уникальным идентификатором для записи, содержащей `callback`. Оно не равно нулю, но других предположений о его значении делать не следует. Вы можете передать его в `window.cancelAnimationFrame()` для отмены вызова.

Обратные вызовы, используемые с `requestAnimationFrame`, получают аргумент `timestamp`, высокоточное значение в миллисекундах, представляющее время с момента загрузки страницы. Удобно, что это значение можно затем использовать для управления анимацией, основанной на времени. Это число аналогичное результату `performance.now()`.

В примере ниже элемент анимируется в течение 2 секунд (2000 миллисекунд). Элемент перемещается вправо со скоростью 0,1 пикселя/мс, поэтому его относительное положение (в пикселях CSS) можно рассчитать как функцию времени, прошедшего с начала анимации (в миллисекундах) с помощью 0.1 * elapsed. Конечная позиция элемента находится на 200 пикселей ( 0.1 * 2000) правее его начальной позиции.

~~~
const element = document.getElementById("element");
let start, previousTimeStamp;
let done = false;

function step(timestamp) {

  if (start === undefined) {
    start = timestamp;
  }
  const elapsed = timestamp - start;
    console.log(elapsed)

  if (previousTimeStamp !== timestamp) {
    // Math.min() is used here to make sure the element stops at exactly 200px
    const count = Math.min(0.1 * elapsed, 200);
    element.style.transform = `translateX(${count}px)`;
    if (count === 200) done = true;
  }

  if (elapsed < 2000) {
    // Stop the animation after 2 seconds
    previousTimeStamp = timestamp;
    if (!done) {
      window.requestAnimationFrame(step);
    }
  }
}

window.requestAnimationFrame(step);
~~~

### Difference between setTimeout (setInterval) and requestAnimationFrame

Самым простым способом создания анимаций является использование `requestAnimationFrame` - метода который делает это непринужденно и легко.

Использование этого метода позволяет браузеру справиться с некоторыми затруднительными задачами связанными с анимацией, например такими как управление частотой кадров.

До этого разработчики использовали `setTimeout` и `setInterval`, чтобы создавать анимации. Проблема тут была в том, что для того, чтобы анимации были плавными, браузер зачастую отрисовывал кадры быстрее, чем они могут показаться на экране. Что вело к ненужным вычислениям. Также ещё одной проблемой в использовании `setInterval` или `setTimeout` было то, что эти анимации продолжали работать, даже если страница не находилась в поле видимости пользователя. Кроме этого, интервал между анимацией мог быть непостоянен.

__Почему нужно использовать `requestAnimationFrame`?__

* __Оптимизация браузером__
С `requestAnimationFrame()` браузер определяет, как и когда делать анимацию. Браузер имеет возможность оптимизировать рендеринг этих анимаций, например, сгруппировав все наши анимации в одну перерисовку браузера. Это не только экономит циклы процессора, но и гарантирует, что все остается синхронизированным.

* __Анимации работают, когда их видно__
Используя `requestAnimationFrame` ваши анимации будут работать только тогда, когда вкладка со страницей видима пользователю.

* __Меньшее потребление питания__
Оптимизации, упомянутые в предыдущих двух моментах помогают сократить количество “мусорных процессов”, которые нужно совершить устройству, чтобы создать анимацию и, следовательно, это ведет к бережному энергопотреблению. Это особенно важно для мобильных устройств, которые обычно имеют относительно короткие сроки работы батареи.
