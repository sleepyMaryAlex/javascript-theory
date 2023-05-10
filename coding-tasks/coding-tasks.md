# ?Coding tasks

`'hello world'.repeating(3)` => `'hello world hello world hello world'`. How to implement?

~~~
String.prototype.repeating = function (count) {
  const result = [];
  for (let i = 0; i < count; i++) {
    result.push(this);
  }
  return result.join(" ");
};

const str = "hello world";

console.log(str.repeating(3)); // hello world hello world hello world
~~~

***

`myFunc('!', 4 ,-10, 34, 0)` => `'4!-10!34!0'`. How to implement?

~~~
function myFunc(separator, ...args) {
  return args.join(separator);
}

console.log(myFunc('!', 4, -10, 34, 0)); // 4!-10!34!0
~~~

***

`five(plus(seven(minus(three()))))` => `9`. How to implement?

~~~
const five = (func) => (func ? func(5) : 5);
const seven = (func) => (func ? func(7) : 7);
const three = (func) => (func ? func(3) : 3);
const minus = (right) => (left) => left - right;
const plus = (right) => (left) => left + right;

console.log(five(plus(seven(minus(three()))))); // 9
~~~

***

`add(5)(9)(-4)(1)` => `11`. How to implement?

~~~
function add(a) {
  function f(b) {
    a += b;
    return f;
  }
  f.valueOf = function () {
    return a;
  };
  return f;
}

console.log(Number(add(1)(2)(3)(4))); // 10
console.log(Number(add(5)(9)(-4)(1))); // 11
~~~

Or:

~~~
function curry(f) {
  return function(a) {
    return function(b) {
      return function(c) {
        return function(d) {
          return f(a, b, c, d);
        }
      }
    };
  };
}

function sum(a, b, c, d) {
  return a + b + c + d;
}

const add = curry(sum);

console.log(add(1)(2)(3)(4)); // 10
console.log(add(5)(9)(-4)(1)); // 11
~~~

***

`periodOutput(period)` method should output in the console once per every period how much time has passed since the first function call. Example: `periodOutput(100)` => `100(after 100 ms)`, `200(after 100 ms)`, `300(after 100 ms)`, ...

~~~
function periodOutput(period) {
  let time = 0;
  function increaseTime() {
    time += period;
    console.log(time);
  }
  setInterval(increaseTime, period);
}

periodOutput(1000);
~~~

***

`extendedPeriodOutput(period)` method should output in the console once per period how much time has passed since the first function call and then increase the period. Example: `extendedPeriodOutput(100)` => `100(after 100 ms)`, `200(after 200 ms)`, `300(after 300 ms)`

~~~
function extendedPeriodOutput(period) {
  let time = 0;
  function increaseTime() {
    time += period;
    const newPeriod = time + period;
    console.log(time);
    setTimeout(increaseTime, newPeriod);
  }
  setTimeout(increaseTime, period);
}

extendedPeriodOutput(100);
~~~

***

Check objects for identity

~~~
const first = {
  name: "John",
  address: {
    country: "Poland",
    city: "Gdansk",
  },
};

const second = {
  name: "John",
  address: {
    country: "Poland",
    city: "Gdansk",
  },
};

function isEqual(object1, object2) {
  const props1 = Object.getOwnPropertyNames(object1);
  const props2 = Object.getOwnPropertyNames(object2);

  if (props1.length !== props2.length) {
    return false;
  }

  for (let i = 0; i < props1.length; i++) {
    const prop = props1[i];
    const bothAreObjects =
      typeof object1[prop] === "object" && typeof object2[prop] === "object";

    if (
      (!bothAreObjects && object1[prop] !== object2[prop]) ||
      (bothAreObjects && !isEqual(object1[prop], object2[prop]))
    ) {
      return false;
    }
  }

  return true;
}

console.log(isEqual(first, second)); // true
~~~
