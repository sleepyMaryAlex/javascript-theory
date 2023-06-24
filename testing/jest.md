# ?Jest

_Jest_, возможно, является самой популярной средой тестирования JavaScript, используемой и поддерживаемой Facebook.

Jest очень простой. Он не требует дополнительных настроек, легкий в понимании и применении, а так же имеет довольно хорошую документацию.

Основные преимущества JEST:

* Совместимость с NodeJS, React, Angular, VueJS и другими проектами на основе Babel.
* Стандартный синтаксис с поддержкой документации.
* Очень быстрый и высокопроизводительный.

Когда использовать JEST:

* Unit Testing
* Integration Testing
* End to End Testing

### Установка

Для установки Jest в ваш проект выполните:

~~~
npm install --save-dev jest
~~~

Если вы используете `yarn`:

~~~
yarn add --dev jest
~~~

После установки можете обновить секцию `scripts` вашего `package.json`:

~~~
"scripts" : {
  "test": "jest"
}
~~~

### Пример

Создадим файл `first.test.js` и напишем наш первый тест:

~~~
test("My first test", () => {
  expect(Math.max(1, 5, 10)).toBe(10);
});
~~~

И запустим наши тесты с помощью `npm run test`. После запуска мы увидим отчет о прохождении тестов.

Функция `test` используется для создания нового теста. Она может принимать три аргумента.

1. Строка с названием теста, его Jest отобразит в отчете. 
2. Функция, которая содержит логику нашего теста.
3. Таймаут. Он является не обязательным, а его значение по умолчанию составляет 5 секунд. Задаётся в миллисекундах. Этот параметр необходим когда мы работаем с асинхронным кодом и возвращаем из функции теста промис. Он указывает как долго Jest должен ждать разрешения промиса. По истечению этого времени, если промис не был разрешен — Jest будет считать тест не пройденным. 

Также вместо `test()` можно использовать `it()`. Разницы между такими вызовами нету. `it()` это просто алиас на функцию `test()`.

Внутри функции теста мы сначала вызываем `expect()`. Ему мы передаем значение, которое хотим проверить. В нашем случае, это результат вызова `Math.max(1, 5, 10)`.

`expect()` возвращает объект «обертку», у которой есть ряд методов для сопоставления полученного значения с ожидаемым. Один из таких методов мы и использовали — `toBe()`.

Давайте разберем основные из этих методов.

### Основные методы

* `toBe()` — подходит, если нам надо сравнивать примитивные значения или является ли переданное значение ссылкой на тот же объект, что указан как ожидаемое значение. Сравниваются значения при помощи `Object.is()`. В отличие от `===` это дает возможность отличать `0` от `-0`, проверить равенство `NaN` c `NaN`.
* `toEqual()` — подойдёт, если нам необходимо сравнить структуру более сложных типов. Он сравнит все поля переданного объекта с ожидаемым. Проверит каждый элемент массива. И сделает это рекурсивно по всей вложенности.

~~~
test("toEqual with objects", () => {
  expect({ foo: "foo", subObject: { baz: "baz" } }).toEqual({
    foo: "foo",
    subObject: { baz: "baz" },
  }); // Ок
  expect({ foo: "foo", subObject: { num: 0 } }).toEqual({
    foo: "foo",
    subObject: { baz: "baz" },
  }); // А вот так ошибка.
});
~~~

~~~
test("toEqual with arrays", () => {
  expect([11, 19, 5]).toEqual([11, 19, 5]); // Ок
  expect([11, 19, 5]).toEqual([11, 19]); // Ошибка
});
~~~

* `toContain()` — проверят содержит массив или итерируемый объект значение. Для сравнения используется оператор `===`.

~~~
test("toContain with arrays and iterable objects", () => {
  const arr = ["apple", "orange", "banana"];
  expect(arr).toContain("banana");
  expect(new Set(arr)).toContain("banana");
  expect("apple, orange, banana").toContain("banana");
});
~~~

* `toContainEqual()` — проверяет или содержит массив элемент с ожидаемой структурой.

~~~
test("toContainEqual with arrays", () => {
  expect([{ a: 1 }, { b: 2 }]).toContainEqual({ a: 1 });
});
~~~

* `toHaveLength()` — проверяет, что свойство `length` у объекта соответствует ожидаемому.

~~~
test("toHaveLength with objects", () => {
  expect([1, 2, 3, 4]).toHaveLength(4);
  expect("foo").toHaveLength(3);
  expect({ length: 1 }).toHaveLength(1);
});
~~~

* `toBeNull()` — проверяет на равенство с `null`.
* `toBeUndefined()` — проверяет на равенство с `undefined`.
* `toBeDefined()` — противоположность `toBeUndefined`. Проверяет, что значение `!== undefined`.
* `toBeTruthy()` — проверяет, что в булевом контексте значение соответствует `true`. Тоесть любые значения кроме `false`, `null`, `undefined`, `0`, `NaN` и пустых строк.
* `toBeFalsy()` — противоположность `toBeTruthy()`. Проверяет, что в булевом контексте значение соответствует `false`.
* `toBeGreaterThan()` и `toBeGreaterThanOrEqual()` — первый метод проверяет, что переданное числовое значение больше, чем ожидаемое `>`, второй проверяет, что число больше или равно ожидаемому `>=`.
* `toBeLessThan()` и `toBeLessThanOrEqual()` — противоположность `toBeGreaterThan()` и `toBeGreaterThanOrEqual()`.
* `toBeCloseTo()` — удобно использовать для чисел с плавающей запятой, когда вам не важна точность и вы не хотите, чтобы тест зависел от незначительной разницы в дроби. Вторым аргументом можно передать до какого знака после запятой необходима точность при сравнении.

~~~
test("toBeCloseTo with numbers", () => {
  const num = 0.1 + 0.2; // 0.30000000000000004
  expect(num).toBeCloseTo(0.3);
  expect(Math.PI).toBeCloseTo(3.14, 2);
});
~~~

* `toMatch()` — проверяет соответствие строки регулярному выражению.

~~~
test("toMatch with strings", () => {
  expect("Banana").toMatch(/Ba/);
});
~~~

* `toThrow()` — используется в случаях, когда надо проверить исключение. Можно проверить как сам факт ошибки, так и проверить на выброс исключения определенного класса, либо по сообщению ошибки, либо по соответствию сообщения регулярному выражению.

~~~
test("toThrow", () => {
  function funcWithError() {
    throw new Error("Some error");
  }

  expect(funcWithError).toThrow();
  expect(funcWithError).toThrow(Error);
  expect(funcWithError).toThrow("Some error");
  expect(funcWithError).toThrow(/Some/);
});
~~~

* `not` — это свойство позволяет сделать проверки на неравенство. Оно предоставляет объект, который имеет все методы перечисленные выше, но работать они будут наоборот.

~~~
test("not", () => {
  expect(true).not.toBe(false);
  expect({ foo: "bar" }).not.toEqual({});

  function funcWithoutError() {}
  expect(funcWithoutError).not.toThrow();
});
~~~

### Примеры простых тестов

Чтобы использовать _Babel_, установите необходимые зависимости:

~~~
npm install --save-dev babel-jest @babel/core @babel/preset-env
~~~

Настройте _Babel_ для вашей текущей версии _Node_, создав файл `babel.config.js` в корне вашего проекта:

~~~
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          node: "current",
        },
      },
    ],
  ],
};
~~~

Давайте напишем пару простых тестов. Для начала создадим простой модуль, который будет содержать несколько методов для работы с окружностями.

circle.js
~~~
export const area = (radius) => Math.PI * radius ** 2;
export const circumference = (radius) => 2 * Math.PI * radius;
~~~

circle.test.js
~~~
import { area, circumference } from "./circle.js";

test("Circle area", () => {
  expect(area(5)).toBeCloseTo(78.54);
  expect(area()).toBeNaN();
});

test("Circumference", () => {
  expect(circumference(11)).toBeCloseTo(69.1, 1);
  expect(circumference()).toBeNaN();
});
~~~

В этих тестах мы проверили результат работы 2-х методов — `area` и `circumference`. При помощи метода `toBeCloseTo` мы сверились с ожидаемым результатом.

В первом случае мы проверили или вычисляемая площадь круга с радиусом `5` приблизительно равна `78.54`, при этом разница с полученым значением (оно составит `78.53981633974483`) не большая и тест будет засчитан.

Во втором мы указали, что нас интересует проверка с точностью до `1` знака после запятой. Также мы вызвали наши методы без аргументов и проверили результат с помощью `toBeNaN`. Поскольку результат их выполнения будет `NaN`, то и тесты будут пройдены успешно.

Разберём ещё один пример. Создадим функцию, которая будет фильтровать массив продуктов по цене:

productFilter.js
~~~
export const byPriceRange = (products, min, max) =>
  products.filter((item) => item.price >= min && item.price <= max);
~~~

product.test.js
~~~
import { byPriceRange } from "./productFilter.js";

const products = [
  { name: "onion", price: 12 },
  { name: "tomato", price: 26 },
  { name: "banana", price: 29 },
  { name: "orange", price: 38 },
];

test("Test product filter by range", () => {
  const from = 15;
  const to = 30;
  const filteredProducts = byPriceRange(products, from, to);

  expect(filteredProducts).toHaveLength(2);
  expect(filteredProducts).toContainEqual({ name: "tomato", price: 26 });
  expect(filteredProducts).toEqual([
    { name: "tomato", price: 26 },
    { name: "banana", price: 29 },
  ]);
  expect(filteredProducts[0].price).toBeGreaterThanOrEqual(from);
  expect(filteredProducts[1].price).toBeLessThanOrEqual(to);
  expect(filteredProducts).not.toContainEqual({ name: "orange", price: 38 });
});
~~~

В этом тесте мы проверям результат работы функии `byRangePrice`.

### Тестирование асинхронного кода

#### Promises

Например, предположим, что `fetchData` возвращает промис, который должен разрешаться в строку `'peanut butter'`.

~~~
export function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("peanut butter"), 1000);
  });
}
~~~

Мы могли бы протестировать это так:

~~~
test("the data is peanut butter", () => {
  return fetchData().then((data) => {
    expect(data).toBe("peanut butter");
  });
});
~~~

#### Async/Await

В качестве альтернативы вы можете использовать `async` и `await` в своих тестах. Чтобы написать асинхронный тест, используйте ключевое слово `async` перед функцией, переданной в `test`. Например, тот же сценарий `fetchData` можно протестировать так:

~~~
export function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("peanut butter"), 1000);
  });
}
~~~

~~~
test('the data is peanut butter', async () => {
  const data = await fetchData();
  expect(data).toBe('peanut butter');
});
~~~

Или:

~~~
export function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject("error"), 1000);
  });
}
~~~

~~~
test("the fetch fails with an error", async () => {
  expect.assertions(1);
  try {
    await fetchData();
  } catch (e) {
    expect(e).toMatch("error");
  }
});
~~~

Не забудьте добавить `expect.assertions`, чтобы убедиться, что вызывается определенное количество утверждений.

#### Callbacks

Предположим, что `fetchData` вместо возврата промиса ожидает обратный вызов, т.е. извлекает некоторые данные и вызывает `callback(null, data)` по завершении. Вы хотите проверить, что возвращенные данные являются строкой `'peanut butter'`.

По умолчанию тесты Jest завершаются, когда достигают конца своего выполнения.

Вместо помещения теста в функцию с пустым аргументом используйте единственный аргумент с именем `done`. Jest будет ждать, пока не будет вызван обратный вызов done перед завершением теста.

~~~
export function fetchData(callback) {
  setTimeout(() => {
    const data = "peanut butter";
    callback(null, data);
  }, 1000);
}
~~~

~~~
test("the data is peanut butter", (done) => {
  function callback(error, data) {
    if (error) {
      done(error);
      return;
    }
    try {
      expect(data).toBe("peanut butter");
      done();
    } catch (error) {
      done(error);
    }
  }

  fetchData(callback);
});
~~~

#### `.resolves/.rejects`

Вы также можете использовать `.resolves` в выражении `expext` и Jest будет ожидать что промис будет выполнен.

~~~
export function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("peanut butter"), 1000);
  });
}
~~~

~~~
test("the data is peanut butter", () => {
  return expect(fetchData()).resolves.toBe("peanut butter");
});
~~~

Если Вы ожидаете, что промис будет отклонён, используйте `.rejects`. Он работает аналогично `.resolves`.

~~~
export function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => reject("error"), 1000);
  });
}
~~~

~~~
test("the fetch fails with an error", () => {
  return expect(fetchData()).rejects.toMatch("error");
});
~~~

### Подготовка и очистка

Часто при написании тестов вам нужно проделать некоторую работу до того, как запустится тест, и некоторую работу по его завершению. Jest предоставляет вспомогательные функции для этих целей.

* `beforeEach(fn, timeout)` - запускает функцию перед запуском каждого из тестов в этом файле. Если функция возвращает промис, Jest ждет разрешения этого промиса перед запуском теста.

* `afterEach(fn, timeout)` - запускает функцию после завершения каждого из тестов в этом файле. Если функция возвращает промис, Jest ждет разрешения этого промиса, прежде чем продолжить.

* `beforeAll(fn, timeout)` - запускает функцию перед запуском любого из тестов в этом файле. Если функция возвращает промис, Jest ждет разрешения этого промиса, прежде чем запускать тесты.

* `afterAll(fn, timeout)` - запускает функцию после завершения всех тестов в этом файле. Если функция возвращает промис, Jest ждет разрешения этого промиса, прежде чем продолжить.

При желании можно указать `timeout` (в миллисекундах), чтобы указать, как долго ждать до прерывания. `Timeout` по умолчанию составляет 5 секунд.

~~~
beforeAll(() => console.log("1 - beforeAll"));
afterAll(() => console.log("1 - afterAll"));
beforeEach(() => console.log("1 - beforeEach"));
afterEach(() => console.log("1 - afterEach"));
test("", () => console.log("1 - test"));

describe("Scoped / Nested block", () => {
  beforeAll(() => console.log("2 - beforeAll"));
  afterAll(() => console.log("2 - afterAll"));
  beforeEach(() => console.log("2 - beforeEach"));
  afterEach(() => console.log("2 - afterEach"));
  test("", () => console.log("2 - test"));
});

// 1 - beforeAll
// 1 - beforeEach
// 1 - test
// 1 - afterEach
// 2 - beforeAll
// 1 - beforeEach
// 2 - beforeEach
// 2 - test
// 2 - afterEach
// 1 - afterEach
// 2 - afterAll
// 1 - afterAll
~~~

Хуки `before*` и `after*` верхнего уровня применяются к каждому тесту в файле. Хуки, объявленные внутри блока `describe`, применяются только к тестам внутри блока `describe`.

`describe(name, fn)` группирует связанные по логике тесты в один блок.

### Mock-функции

Мок-функции позволяют тестировать связи между кодом, удаляя фактическую реализацию функции.

Давайте представим, что мы тестируем реализацию функции `forEach`, которая выполняет обратный вызов для каждого элемента предоставленного массива.

~~~
export function forEach(items, callback) {
  for (let index = 0; index < items.length; index++) {
    callback(items[index]);
  }
}
~~~

Чтобы протестировать эту функцию, мы можем использовать мок-функцию, и посмотреть на состояние мока чтобы убедиться, что функция была вызвана как ожидалось.

~~~
const mockCallback = jest.fn((x) => 42 + x);

test("forEach mock function", () => {
  forEach([0, 1], mockCallback);

  // The mock function was called twice
  expect(mockCallback.mock.calls).toHaveLength(2);

  // The first argument of the first call to the function was 0
  expect(mockCallback.mock.calls[0][0]).toBe(0);

  // The first argument of the second call to the function was 1
  expect(mockCallback.mock.calls[1][0]).toBe(1);

  // The return value of the first call to the function was 42
  expect(mockCallback.mock.results[0].value).toBe(42);
});
~~~

У всех мок-функций есть особое свойство `.mock`, где хранятся данные о том как функция была вызвана и что она вернула.

### Тестирование при помощи снимков

_Тестирование моментальных снимков_ — это автоматизированная процедура тестирования, посредством которой фиксируется текущее состояние пользовательского интерфейса приложения и сравнивается с ранее сохраненным моментальным снимком. Это помогает обнаруживать неожиданные изменения в выводе и упрощает проверку изменений кода.

#### Тестирование при помощи снимков приложений React

Вот несколько ключевых причин, по которым тестирование моментальных снимков необходимо для тестирования компонентов React в Jest.

* Использование моментальных снимков для имитации изменений пользовательского интерфейса для тестирования. Хотя разработчики прилагают все усилия для реализации изменений именно так, как задумал дизайнер, тестирование моментальных снимков помогает гарантировать, что это произойдет. Визуальное представление кода записывается, а затем сравнивается со снимком экрана нового сценария, чтобы проверить, есть ли разница.
* Использование снэпшот-тестов для определения того, были ли изменения преднамеренными: при использовании Jest для снэпшот-тестирования компонентов React вы можете быстро определить, отображает ли снэпшот результат, который пытается получить разработчик. Если все пройдет гладко, снимки могут сообщить об этом; если нет, они показывают результат, которого разработчик не ожидал.

Настроим Jest в своем проекте:

~~~
npm install react-test-renderer
~~~

Создадим React-приложение, для которого напишем Snapshot-тесты с помощью Jest. 

HelloWorld.js
~~~
function HelloWorld() {
  return (
    <div>
      <h1>Hello World</h1>
    </div>
  );
}

export default HelloWorld;
~~~

App.js
~~~
import HelloWorld from "./HelloWorld";

function App() {
  return <HelloWorld />;
}

export default App;
~~~

HelloWorld.test.js
~~~
import renderer from "react-test-renderer";
import HelloWorld from "./HelloWorld";

test("performs snapshot testing", () => {
  const tree = renderer.create(<HelloWorld />).toJSON();
  expect(tree).toMatchSnapshot();
});
~~~

После того, как вы написали тестовый файл, выполним команду `npm test`.

Снимок сохраняется внутри каталога:

`__snapshots__/HelloWorld.test.js.snap`

~~~
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`performs snapshot testing 1`] = `
<div>
  <h1>
    Hello World
  </h1>
</div>
`;
~~~

Если немного изменить код компонента и запустить тестовую команду, терминал отобразит результат, подчеркнув изменение.

Если изменение является преднамеренным, вы можете обновить снимок, нажав клавишу `«u»` в режиме `–watch`.

Встроенные моментальные снимки ведут себя так же, как внешние моментальные снимки (`.snap` файлы), за исключением того, что значения моментальных снимков автоматически записываются обратно в исходный код. Это означает, что вы можете воспользоваться преимуществами автоматически созданных моментальных снимков, не переключаясь на внешний файл, чтобы убедиться, что записано правильное значение.

Сначала вы пишете тест, вызывающий `.toMatchInlineSnapshot()` без аргументов:

HelloWorld.test.js
~~~
import renderer from "react-test-renderer";
import HelloWorld from "./HelloWorld";

test("renders correctly", () => {
  const tree = renderer.create(<HelloWorld />).toJSON();
  expect(tree).toMatchInlineSnapshot();
});
~~~

В следующий раз, когда вы запустите Jest, дерево будет оцениваться, а снимок будет записан в качестве аргумента `toMatchInlineSnapshot`:

HelloWorld.test.js
~~~
import renderer from "react-test-renderer";
import HelloWorld from "./HelloWorld";

test("renders correctly", () => {
  const tree = renderer.create(<HelloWorld />).toJSON();
  expect(tree).toMatchInlineSnapshot(`
<div>
  <h1>
    Hello World
  </h1>
</div>
`);
});
~~~

Вот и все! Вы можете обновить снимки с помощью ключа `«u»` в режиме `–watch`.

Если появляется ошибка `"Support for the experimental syntax 'jsx' isn't currently enabled"`, то можно создать файл `.babelrc` в корневом каталоге вашего проекта со следующим содержимым:

~~~
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
~~~
