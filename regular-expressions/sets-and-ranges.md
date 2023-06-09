# ?Sets and ranges

Несколько символов или символьных классов в квадратных скобках `[...]` означают «искать любой символ из заданных».

### Наборы

Для примера, `[eao]` означает любой из 3-х символов: `'a'`, `'e'` или `'o'`.

Наборы могут использоваться в регулярных выражениях вместе с обычными символами, например:

~~~
console.log("Cat mat".match(/[cm]at/gi)); // ['Cat', 'mat']
~~~

### Диапазоны

Ещё квадратные скобки могут содержать диапазоны символов.

К примеру, `[a-z]` соответствует символу в диапазоне от `a` до `z`, или `[0-5]` – цифра от `0` до `5`.

В приведённом ниже примере мы ищем `"x"`, за которым следуют две цифры или буквы от `A` до `F`:

~~~
console.log("Exception 0xAF".match(/x[0-9A-F][0-9A-F]/g)); // ['xAF']
~~~

Здесь в `[0-9A-F]` сразу два диапазона: ищется символ, который либо цифра от `0` до `9`, либо буква от `A` до `F`.

Также мы можем использовать символьные классы внутри `[...]`.

Например:

* `\d` – то же самое, что и `[0-9]`,
* `\w` – то же самое, что и `[a-zA-Z0-9_]`,
* `\s` – то же самое, что и `[\t\n\v\f\r ]`, плюс несколько редких пробельных символов Юникода.

### Пример: многоязычный аналог `\w`

Так как символьный класс `\w` является всего лишь сокращением для `[a-zA-Z0-9_]`, он не найдёт китайские иероглифы, кириллические буквы и т.п.

Давайте сделаем более универсальный шаблон, который ищет символы, используемые в словах, для любого языка. Это очень легко с Юникод-свойствами: `[\p{Alpha}\p{M}\p{Nd}\p{Pc}\p{Join_C}]`.

~~~
const regexp = /[\p{Alpha}\p{M}\p{Nd}\p{Pc}\p{Join_C}]/gu;

const str = `Hi 你好 12`;

// найдены все буквы и цифры
console.log(str.match(regexp)); // ['H', 'i', '你', '好', '1', '2']
~~~

### Исключающие диапазоны

Помимо обычных диапазонов, есть «исключающие» диапазоны, которые выглядят как `[^...]`.

Они обозначаются символом каретки `^` в начале диапазона и соответствуют любому символу за исключением заданных.

~~~
console.log("alice15@gmail.com".match(/[^\d\sA-Z]/gi)); // ['@', '.']
~~~

### Экранирование внутри `[...]`

В квадратных скобках большинство специальных символов можно использовать без экранирования:

~~~
// Нет необходимости в экранировании
const regexp = /[-().^+]/g;

console.log("1 + 2 - 3".match(regexp)); // ['+', '-']
~~~

...Впрочем, если вы решите экранировать «на всякий случай», то не будет никакого вреда:

~~~
// Экранирование всех возможных символов
const regexp = /[\-\(\)\.\^\+]/g;

console.log("1 + 2 - 3".match(regexp)); // ['+', '-']
~~~

### Наборы и флаг `«u»`

Если в наборе есть суррогатные пары, для корректной работы обязательно нужен флаг `u`.

Например, давайте попробуем найти шаблон `[𝒳𝒴]` в строке `𝒳`:

~~~
console.log("𝒳".match(/[𝒳𝒴]/)); // ['�', index: 0, input: '𝒳', groups: undefined]
// (поиск был произведён неправильно, вернулась только половина символа)
~~~

Результат неверный, потому что по умолчанию регулярные выражения «не знают» о существовании суррогатных пар.

Если добавить флаг `u`, то всё будет в порядке:

~~~
console.log("𝒳".match(/[𝒳𝒴]/u)); // ['𝒳', index: 0, input: '𝒳', groups: undefined]
~~~
