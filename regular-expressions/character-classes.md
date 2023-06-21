# ?Character classes

Рассмотрим практическую задачу – у нас есть номер телефона вида `"+7(903)-123-45-67"`, и нам нужно превратить его в строку только из чисел: `79031234567`.

_Символьный класс_ – это специальное обозначение, которое соответствует любому символу из определённого набора.

Для начала давайте рассмотрим класс «цифра». Он обозначается как `\d` и в регулярном выражении соответствует «любой одной цифре».

Добавим флаг `g`, чтобы найти все цифры:

~~~
const str = "+7(903)-123-45-67";

const regexp = /\d/g;

console.log(str.match(regexp)); // ['7', '9', '0', '3', '1', '2', '3', '4', '5', '6', '7']

// и можно сделать из них уже чисто цифровой номер телефона
console.log(str.match(regexp).join("")); // 79031234567
~~~

Есть и другие символьные классы.

Наиболее используемые:

* `\d` («d» от английского «digit» означает «цифра»). Цифра: символ от `0` до `9`.
* `\s` («s»: от английского «space» – «пробел»). Пробельные символы: включает в себя символ пробела, табуляции `\t`, перевода строки `\n` и некоторые другие редкие пробельные символы, обозначаемые как `\v`, `\f` и `\r`.
* `\w` («w»: от английского «word» – «слово»). Символ «слова», а точнее – буква латинского алфавита или цифра или подчёркивание `_`. Нелатинские буквы не являются частью класса `\w`, то есть буква русского алфавита не подходит.

Для примера, `\d\s\w` обозначает «цифру», за которой идёт пробельный символ, а затем символ слова, например `1 a`.

##### Регулярное выражение может содержать как обычные символы, так и символьные классы.

Например, `CSS\d` соответствует строке `CSS` с цифрой после неё:

~~~
const str = "Есть ли стандарт CSS4?";
const regexp = /CSS\d/;
console.log(str.match(regexp)); // ['CSS4', index: 17, input: 'Есть ли стандарт CSS4?', groups: undefined]
~~~

Также мы можем использовать несколько символьных классов:

~~~
console.log("I love HTML5!".match(/\s\w\w\w\w\d/)); // [' HTML5', index: 6, input: 'I love HTML5!', groups: undefined]
~~~

### Обратные символьные классы

Для каждого символьного класса существует «обратный класс», обозначаемый той же буквой, но в верхнем регистре.

«Обратный» означает, что он соответствует всем другим символам, например:

* `\D`. Не цифра: любой символ, кроме `\d`, например буква.
* `\S`. Не пробел: любой символ, кроме `\s`, например буква.
* `\W`. Любой символ, кроме `\w`, то есть не буквы из латиницы, не знак подчёркивания и не цифра. В частности, русские буквы принадлежат этому классу.

~~~
const str = "+7(903)-123-45-67";

console.log(str.replace(/\D/g, "")); // 79031234567
~~~

### Точка – это любой символ

Точка `.` – это специальный символьный класс, который соответствует «любому символу, кроме новой строки».

~~~
const regexp = /CS.4/;

console.log("CSS4".match(regexp)); // ['CSS4', index: 0, input: 'CSS4', groups: undefined]
console.log("CS-4".match(regexp)); // ['CS-4', index: 0, input: 'CS-4', groups: undefined]
console.log("CS 4".match(regexp)); // ['CS 4', index: 0, input: 'CS 4', groups: undefined]
console.log("CS4".match(/CS.4/)); // null, нет совпадений потому что нет символа для точки
~~~

Точка не соответствует символу новой строки `\n`.

~~~
console.log("A\nB".match(/A.B/)); // null (нет совпадения)
~~~

### Точка как буквально любой символ, с флагом `«s»`

Во многих ситуациях точкой мы хотим обозначить действительно «любой символ», включая перевод строки.

Как раз для этого нужен флаг `s`. Если регулярное выражение имеет его, то точка `.` соответствует буквально любому символу:

~~~
console.log("A\nB".match(/A.B/s));
/*
['A
B', index: 0, input: 'A
B', groups: undefined]
*/
~~~

##### Пробел – это символ. Такой же важный, как любой другой.

Нельзя просто добавить или удалить пробелы из регулярного выражения, и ожидать, что оно будет также работать.

Другими словами, в регулярном выражении все символы имеют значение, даже пробелы.

~~~
console.log("1 - 5".match(/\d-\d/)); // null, нет совпадения!

console.log("1 - 5".match(/\d - \d/)); // ['1 - 5', index: 0, input: '1 - 5', groups: undefined]
// или можно использовать класс \s:
console.log("1 - 5".match(/\d\s-\s\d/)); // ['1 - 5', index: 0, input: '1 - 5', groups: undefined]
~~~