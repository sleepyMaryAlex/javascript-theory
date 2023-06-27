# ?Symbol.iterator usage

_Итерируемые объекты_ – это те объекты, которые реализуют метод `Symbol.iterator` и которые можно перебрать в цикле `for..of`. Например, итерируемым объектом является массив, список DOM-узлов, строки, `Map`, `Set`.

##### Явное использование `Symbol.iterator`:

~~~
const arr = [1, 2, 3];
const iterator = arr[Symbol.iterator]();
while (true) {
  const result = iterator.next();
  if (result.done) {
    break;
  }
  console.log(result.value); // 1, 2, 3
}
~~~

##### `Symbol.iterator` дает возможность сделать итерируемыми любые объекты.

Допустим, у нас есть объект `range`:

~~~
let range = {
  from: 1,
  to: 5
};
~~~

Мы хотим на выходе 1, 2, 3, 4, 5

Чтобы сделать `range` итерируемым, нам нужно добавить в объект метод `Symbol.iterator`.

~~~
range[Symbol.iterator] = function() {
  return {
    current: this.from,
    last: this.to,
    next() {
      if (this.current <= this.last) {
        return { done: false, value: this.current++ };
      } else {
        return { done: true };
      }
    }
  };
};

for (let num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}
~~~

* Когда цикл `for..of` запускается, он вызывает `Symbol.iterator` один раз. `Symbol.iterator` должен вернуть объект с методом next.
* Когда `for..of` хочет получить следующее значение, он вызывает метод `next()` этого объекта.

##### Как написать свой итератор:

~~~
const iterator = {
  [Symbol.iterator](n = 3) {
    let i = 0;
    return {
      next() {
        if (i < n) {
          return { done: false, value: i++ }
        } else {
          return { done: true, value: undefined }
        }
      }
    }
  }
}

for(let k of iterator) {
  console.log(k); // 0 1 2
}
~~~
