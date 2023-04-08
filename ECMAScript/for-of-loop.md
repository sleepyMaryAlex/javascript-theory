# ?for..of loop

Цикл `for...of` появился в стандарте ES6. Предназначен он для перебора итерируемых объектов, т.е. объектов, в которых реализован метод `Symbol.iterator`. Этот метод ещё называют итератором. Именно его и использует цикл `for...of` для перебора объектов. Итератор должен возвращать всего один метод `next()`. Этот метод в свою очередь тоже должен возвращать объект, состоящий из 2 свойств: `value` и `done`. Ключ `done` - булевый. Он определяет есть ли ещё значения в последовательности (`false` - да, `true` - нет).

~~~
const array = {
  values: ["gray", "black", "white", "red", "green"],
  [Symbol.iterator]() {
    const values = this.values;
    let i = 0;
    return {
      next() {
        const done = i >= values.length;
        const value = done ? undefined : values[i++];
        return { value, done };
      },
    };
  },
};

for (let values of array) {
  console.log(values); // gray black white red green
}
~~~

Метод `Symbol.iterator` есть у `String`, `Array`, `Map`, `Set`, `arguments`, `NodeList` и других объектов. `Object` не является итерируемым.

Для `for..of` обход происходит в соответствии с тем, какой порядок определён в итерируемом объекте.

В большинстве задач, использующих цикл, предпочтительнее `for..of`. Иногда его бывает недостаточно, и требуется ручное управление обходом. В таких случаях можно возвращаться к использованию `for`. Например, когда нужно идти не по каждому элементу массива, а через один или обходить массив в обратном порядке.