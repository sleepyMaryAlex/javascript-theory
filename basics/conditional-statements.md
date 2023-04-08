# ?Conditional statements

### if, if else
Иногда нам нужно выполнить различные действия в зависимости от условий.

__Инструкция «if»__

Инструкция if(...) вычисляет условие в скобках и, если результат true, то выполняет блок кода. Инструкция if (…) вычисляет выражение в скобках и преобразует результат к логическому типу.
Число 0, пустая строка "", null, undefined и NaN становятся false. Из-за этого их называют «ложными» («falsy») значениями.
Остальные значения становятся true, поэтому их называют «истинными» («truthy»).

__Блок «else»__

Инструкция if может содержать необязательный блок «else» («иначе»). Он выполняется, когда условие ложно.

__Несколько условий: «else if»__

Иногда нужно проверить несколько вариантов условия. 
Пустая инструкция используется, когда инструкция не нужна, хотя синтаксис JavaScript будет предполагать её.
Если three истинно, ничего не произойдёт, four не важна, и функция launchRocket() тоже не запустится:

~~~
if (one) 
  doOne();
else if (two)
  doTwo();
else if (three)
  ; // nothing here
else if (four)
  doFour();
else
  launchRocket();
~~~

Чтобы выполнить несколько инструкций в условии, используйте блочный оператор ({...}) для группирования этих инструкций. В общем, хорошей практикой всегда является использование блочных операторов, особенно в коде, включающем вложенные операторы if.
В случае else if else if выполнится тот блок кода, условие которого истинное и находится выше по коду. Остальные условия будут проигнорированы.

Когда условий становится много, то страдает читаемость кода. В этом случае лучше писать switch, но он подходит не всегда.

### ternary operator

_Условный (тернарный) оператор_ - единственный оператор в JavaScript, принимающий три операнда: условие, за которым следует знак вопроса `(?)`, затем выражение, которое выполняется, если условие истинно, сопровождается двоеточием `(:)`, и, наконец, выражение, которое выполняется, если условие ложно. Он часто используется в качестве укороченного варианта условного оператора if else.

```условие ? выражение1 : выражение2```

Оператор может что-то возвращать:

```const elvisLives = Math.PI > 4 ? "Да" : "Нет";```

А может быть сам по себе:

```age > 18 ? location.assign("continue.html") : stop = true;```

Также возможно выполнять несколько операций на каждое сравнение, разделив их запятыми. При присвоении значения также возможно выполнение более одной операции. В этом случае переменной будет присвоено то значение, которое стоит последним в списке значений, разделённых запятой:

~~~
let stop = false, age = 13;

const result = age > 18 ? (
  console.log("Yes"),
  "You can continue"
) : (
  stop = true,
  console.log(stop),
  "No",
  "You can't continue"
);
console.log(result); // true, you can't continue
~~~

### switch case - examples, where it can be useful

_switch_ операторы могут иметь более чистый синтаксис по сравнению со сложными if else операторами. 

~~~
switch ("oboe") {
  case "trumpet":
    console.log("I play the trumpet");
    break;
  case "flute":
    console.log("I play the flute");
    break;
  case "oboe":
    console.log("I play the oboe");
    break;
  default:
    console.log("I don't play an instrument. Sorry");
    break;
}
~~~

Компьютер выполнит switch оператор и проверит строгое равенство === между case и expression. Если один из случаев соответствует expression, то код внутри этого case предложения будет выполнен. Если ни один из случаев не соответствует выражению, предложение default будет выполнено. Если оператору соответствует несколько случаев, то будет использоваться первый case, который соответствует expression.

Оператор break выйдет из switch при совпадении. Если break отсутствуют, компьютер продолжит выполнение switch, даже если будет найдено совпадение.

Если return операторы присутствуют в switch, то вам не нужен break оператор.

__Что будет если break отсутствует?__

~~~
switch (2) {
  case 1:
    console.log("Number 1 was chosen");
  case 2:
    console.log("Number 2 was chosen");
  case 3:
    console.log("Number 3 was chosen");
  default:
    console.log("No number was chosen");
}
// Number 2 was chosen
// Number 3 was chosen
// No number was chosen
~~~

Если break нет, то выполнение пойдёт ниже по следующим case, при этом остальные проверки игнорируются.

__Где разместить default?__

Стандартное соглашение заключается в том, чтобы поместить default как последнее предложение. Но вы можете поместить его и перед другими случаями. И будет работать как надо.

__Несколько случаев для одной операции__

~~~
const country = "United States";
switch (country) {
  case "France":
  case "Spain":
  case "Ireland":
  case "Poland":
    console.log("This country is in Europe.");
    break;
  default:
    console.log("This country is not in Europe.");
}
~~~

__Операторы области действия блока и switch__

В этом примере будет выдано сообщение об ошибке, поскольку переменная message уже была объявлена:

~~~
const errand = "Going Shopping";
switch (errand) {
  case "Going to the Dentist":
    let message = "I hate going to the dentist";
    console.log(message);
    break;
  case "Going Shopping":
    let message = "I love to shop";
    console.log(message);
    break;
  default:
    console.log("No errands");
    break;
}
~~~

Чтобы избавиться от этого сообщения об ошибке, случаи должны быть заключены в набор фигурных скобок.

~~~
const errand = "Going Shopping";
switch (errand) {
  case "Going to the Dentist": {
    let message = "I hate going to the dentist";
    console.log(message);
    break;
  }
  case "Going Shopping": {
    let message = "I love to shop";
    console.log(message);
    break;
  }
  default: {
    console.log("No errand");
    break;
  }
}
~~~

__Диапазоны switch__

Может возникнуть ситуация, когда вам потребуется оценить диапазон значений в блоке switch, а не одно значение, как в примере выше. Мы можем сделать это, установив наше выражение true и выполнив операцию внутри каждого case оператора:

~~~
const grade = 87;

switch (true) {
  case grade >= 90:
    console.log("A");
    break;
  case grade >= 80:
    console.log("B");
    break;
  case grade >= 70:
    console.log("C");
    break;
  case grade >= 60:
    console.log("D");
    break;
  default:
    console.log("F");
}
// B
~~~
