# ?String templates

_Шаблонные строки_ — это ещё один способ создания строк, наравне с одинарными или двойными кавычками.

~~~
const name = "Tom";
let age = 37;
const userInfo = `${name} is ${age} years old`;
console.log(userInfo);      // Tom is 37 years old
~~~

Благодаря шаблонной строке мы избегаем ненужного экранирования символов. Если в строке, ограниченной кавычками одного из этих видов, встречаются кавычки того же вида, их нужно экранировать.

~~~
const single = '"We don\'t make mistakes. We just have happy accidents." - Bob Ross'

const double = "\"We don't make mistakes. We just have happy accidents.\" - Bob Ross"
~~~

Шаблонные литералы, с другой стороны, объявляют с использованием обратных кавычек. Здесь не нужно экранировать одинарные или двойные кавычки:

~~~
const template = `"We don't make mistakes. We just have happy accidents." - Bob Ross`
~~~

Но обратные кавычки в таких строках экранировать необходимо:

~~~
const template = `Template literals use the \` character.`
~~~

В шаблонной строке с помощью синтаксиса `${ }` можно использовать любые выражения JavaScript. Любой нестроковый результат будет приведён к строке.

~~~
const age = 19;
const message = `You can ${age < 21 ? "not" : ""} view this page`;
console.log(message); // You can not view this page
~~~

Создание многострочных строк с использованием шаблонных литералов достаточно просто, в конце очередного фрагмента строки, нажать клавишу Enter, и продолжить вводить следующую строку шаблонного литерала:

~~~
const address = `Homer J. Simpson
742 Evergreen Terrace
Springfield`;
~~~

Создать многострочную строку можно используя символ перевода строки `(\n)`, но это не очень удобно:

~~~
const address =
  "Homer J. Simpson\n" + "742 Evergreen Terrace\n" + "Springfield";
~~~
