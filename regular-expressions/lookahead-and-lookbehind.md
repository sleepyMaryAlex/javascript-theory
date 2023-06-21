# ?Lookahead and lookbehind

В некоторых случаях нам нужно найти соответствия шаблону, но только те, за которыми или перед которыми следует другой шаблон.

Для этого в регулярных выражениях есть специальный синтаксис: опережающая (lookahead) и ретроспективная (lookbehind) проверка.

В качестве первого примера найдём стоимость из строки `1 индейка стоит 30€`. То есть, найдём число, после которого есть знак валюты `€`.

### Опережающая проверка

Синтаксис опережающей проверки: `X(?=Y)`.

Он означает: найди `X` при условии, что за ним следует `Y`. Вместо `X` и `Y` здесь может быть любой шаблон.

Для целого числа, за которым идёт знак `€`, шаблон регулярного выражения будет `\d+(?=€)`:

~~~
const str = "1 индейка стоит 30€";

console.log(str.match(/\d+(?=€)/)); // ['30', index: 16, input: '1 индейка стоит 30€', groups: undefined]
// число 1 проигнорировано, так как за ним НЕ следует €
~~~

Обратим внимание, что проверка – это именно проверка, содержимое скобок `(?=...)` не включается в результат `30`.

При поиске `X(?=Y)` движок регулярных выражений, найдя `X`, проверяет есть ли после него `Y`. Если это не так, то игнорирует совпадение и продолжает поиск дальше.

Возможны и более сложные проверки, например `X(?=Y)(?=Z)` означает, что мы ищем `X` при условии, что за ним идёт и `Y` и `Z`.

Такое возможно только при условии, что шаблоны `Y` и `Z` не являются взаимно исключающими.

Например, `\d+(?=\s)(?=.*30)` ищет `\d+` при условии, что за ним идёт пробел, и где-то впереди есть `30`:

~~~
const str = "1 индейка стоит 30€";

console.log(str.match(/\d+(?=\s)(?=.*30)/)); // ['1', index: 0, input: '1 индейка стоит 30€', groups: undefined]
~~~

В нашей строке это как раз число `1`.

### Негативная опережающая проверка

Допустим, нам нужно узнать из этой же строки количество индеек, то есть число `\d+`, за которым НЕ следует знак `€`.

Для этой задачи мы можем применить негативную опережающую проверку.

Синтаксис: `X(?!Y)`

Он означает: найди такой `X`, за которым НЕ следует `Y`.

~~~
const str = "2 индейки стоят 60€";

console.log(str.match(/\d+(?!€)/)); // ['2', index: 0, input: '2 индейки стоят 60€', groups: undefined]
// (в этот раз проигнорирована цена)
~~~

### Ретроспективная проверка

Опережающие проверки позволяют задавать условия на то, что «идёт после».

Ретроспективная проверка выполняет такую же функцию, но с просмотром назад. Другими словами, она находит соответствие шаблону, только если перед ним есть что-то заранее определённое.

Синтаксис:

* Позитивная ретроспективная проверка: `(?<=Y)X`, ищет совпадение с `X` при условии, что перед ним ЕСТЬ `Y`.
* Негативная ретроспективная проверка: `(?<!Y)X`, ищет совпадение с `X` при условии, что перед ним НЕТ `Y`.

Чтобы протестировать ретроспективную проверку, давайте поменяем валюту на доллары США. Знак доллара обычно ставится перед суммой денег, поэтому для того чтобы найти `$30`, мы используем `(?<=\$)\d+` – число, перед которым идёт `$`:

~~~
const str = "1 индейка стоит $30";

// знак доллара экранируем \$, так как это специальный символ
console.log(str.match(/(?<=\$)\d+/)); // ['30', index: 17, input: '1 индейка стоит $30', groups: undefined]
// одинокое число игнорируется
~~~

Если нам необходимо найти количество индеек – число, перед которым не идёт `$`, мы можем использовать негативную ретроспективную проверку `(?<!\$)\d+`:

~~~
const str = "2 индейки стоят $60";

console.log(str.match(/(?<!\$)\d+/)); // ['2', index: 0, input: '2 индейки стоят $60', groups: undefined]
// проигнорировалась цена
~~~

### Скобочные группы

Как правило, то что находится внутри скобок, задающих опережающую и ретроспективную проверку, не включается в результат совпадения.

Например, в шаблоне `\d+(?=€)` знак `€` не будет включён в результат. Это логично, ведь мы ищем число `\d+`, а `(?=€)` – это всего лишь проверка, что за ним идёт знак `€`.

Но в некоторых ситуациях нам может быть интересно захватить и то, что в проверке. Для этого нужно обернуть это в дополнительные скобки.

В следующем примере знак валюты `(€|kr)` будет включён в результат вместе с суммой:

~~~
const str = "1 индейка стоит 30€";
const regexp = /\d+(?=(€|kr))/;
// добавлены дополнительные скобки вокруг €|kr

console.log(str.match(regexp)); // ['30', '€', index: 16, input: '1 индейка стоит 30€', groups: undefined]
~~~

То же самое можно применить к ретроспективной проверке:

~~~
const str = "1 индейка стоит $30";
const regexp = /(?<=(\$|£))\d+/;

console.log(str.match(regexp)); // ['30', '$', index: 17, input: '1 индейка стоит $30', groups: undefined]
~~~