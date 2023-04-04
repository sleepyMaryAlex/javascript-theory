# ?what is polyfills
_Полифил (полифилл; англ. Polyfill)_ — код, реализующий какую-либо функциональность, которая не поддерживается в некоторых версиях веб-браузеров.

__Array.map polyfill__

~~~
Array.prototype.polyfillMap = function (callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i));
  }
  return result;
};

console.log([1, 2, 3, 4, 5].polyfillMap((num, index) => num + index)); // [1, 3, 5, 7, 9]
~~~

__Array.reduce polyfill__

~~~
Array.prototype.polyfillReduce = function(fn, initial) {
  this.forEach((item) => {
    initial = initial ? fn(initial, item) : item;
  })
  return initial;
}
~~~

__Array.flat polyfill__

~~~
const nestedArray = [[1, 2], [3, [4, 5, [6]]]];

if (!Array.prototype.flat) {
  Array.prototype.flat = function(depth = 1) {
    let result = [];
    if (depth < 1) {
      result = this.slice(0);
    } else {
      this.forEach((value) => {
        Array.isArray(value) ? result.push(...value.flat1(depth - 1)) : result.push(value);
      });
    }
    return result;
  };
}

console.log(nestedArray.flat(3)); // [1, 2, 3, 4, 5, 6]
~~~

__Object.create polyfill__

~~~
if (!Object.create) {
  Object.create = function(proto) {
    function F() {};
    F.prototype = proto;
    return new F();
  }
}
~~~

__Function.bind polyfill__

~~~
if (!Function.prototype.bind) {
  Function.prototype.bind = function(context, ...args) {
    context.func = this;
    return function (...params) {
      return context.func(...args, ...params)
    }
  };
}
~~~

Или с call()

~~~
if (!Function.prototype.bind) {
  Function.prototype.bind = function(context, ...args) {
    const callback = this;
    return function() {
      callback.call(context, ...args);
    };
  };
}
~~~
