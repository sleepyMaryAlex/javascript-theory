# ?Object methods

~~~
const countryCanada = {
  countryName: "Canada",
  capital: "Ottawa",
  region: "North America",
}
~~~

```console.log(Object.keys(countryCanada)); // ['countryName', 'capital', 'region']```
Возвращает массив ключей объекта

```console.log(Object.values(countryCanada)); // ['Canada', 'Ottawa', 'North America']```
Возвращает массив перечисляемых значений объекта

```console.log(Object.entries(countryCanada)); // [ ["countryName", "Canada"], ["capital", "Ottawa"], ["region", "North America"] ];```
Возвращает массив всех пар ключ-значение объекта, не перечисляет свойства из цепочки прототипов

```console.log(Object.fromEntries(arr));```
Преобразует массив пар ключ-значение (как после Object.entries) в объект

```console.log(Object.assign({}, countryCanada)); // { countryName: 'Canada', capital: 'Ottawa', region: 'North America' }```
Чтобы сделать копию объекта или объединить сколько угодно объектов
Еще один способ сделать копию объекта: {...countryCanada}

```console.log(Object.create(countryCanada, { p: { value: 42 } })); // { p: 42 }```
Cоздает и возвращает новый объект с заданным прототипом и свойствами

```console.log(countryCanada.hasOwnProperty("capital")); // true```
Проверяет, владеет ли объект искомым свойством, а не наследует его

```console.log(Object.is(countryCanada, countryCanada)); // true, так как это ссылки на один и тот же объект```

Похож на ===, но с 2 отличиями. Считает +0 и -0 неравными, null и null равными

```console.log(Object.freeze(countryCanada));```
Возвращает замороженный объект. Предотвращает добавление новых свойств к объекту, удаление старых и изменение существующих свойств или значения их атрибутов перечисляемости, настраиваемости и записываемости. Дочерние объекты объекта, могут быть изменены, если только они не заморожены отдельно.

```console.log(Object.isFrozen(countryCanada)); // true```
Определяет, был ли объект заморожен

```console.log(Object.seal(countryCanada));```
Возвращает запечатанный объект. Запрещает добавление новых свойств к объекту и удаление старых, но значения существующих свойств всё ещё могут изменяться. Все текущие свойства делает configurable: false. Дочерние объекты объекта, могут быть изменены, если только они не запечатаны отдельно.

```console.log(Object.isSealed(countryCanada));```
Проверяет, является ли объект запечатанным.

~~~
Object.defineProperties(countryCanada, {
  population: {
    value: 13000,
    writable: true
  },
  language: {
    value: 'eng'
  }
});
~~~

defineProperties определяет новые или изменяет существующие свойства объекта

```console.log(Object.getOwnPropertyDescriptor(countryCanada, 'capital')) // { value: 'Ottawa', writable: true, enumerable: true, configurable: true }```
Возвращает дескриптор свойства для собственного свойства

```console.log(Object.getOwnPropertyDescriptors(countryCanada));```
Возвращает все собственные дескрипторы свойств данного объекта

```console.log(Object.getOwnPropertyNames(countryCanada)); // ['countryName', 'capital', 'region']```
Возвращает массив строк, соответствующих перечисляемым и неперечисляемым свойствам. В этом отличие от Object.keys

```console.log(Object.getOwnPropertySymbols(countryCanada)); // []```
Возвращает массив всех символьных свойств

```const obj = Object.create(countryCanada)```
```console.log(countryCanada.isPrototypeOf(obj)); // true```

Проверяет, существует ли объект в цепочке прототипов другого объекта.

```const obj = Object.create(countryCanada)```
```console.log(Object.getPrototypeOf(obj) === countryCanada); // true```
Возвращает прототип (то есть, внутреннее свойство ```[[Prototype]]```) указанного объекта.

const obj = {};
```console.log(Object.setPrototypeOf(obj, countryCanada));```
// Устанавливает прототип (т.е. внутреннее ```[[Prototype]]``` свойство) указанного объекта в другой объект

```console.log(Object.hasOwn(countryCanada, 'capital')); // true```
возвращает true, если указанный объект имеет указанное свойство как собственное свойство

```console.log(countryCanada.hasOwnProperty('capital')); // true```
Похоже на hasOwn, но другой синтаксис

```Object.preventExtensions(countryCanada);```
Предотвращает добавление к объекту новых свойств. В отличие от Object.seal позволяет удаление существующих свойств и перенастройку значений атрибутов перечисляемости, настраиваемости и записываемости.

```console.log(Object.isExtensible(countryCanada)); // false```
Определяет, могут ли к объекту добавляться новые свойства.

```console.log(countryCanada.propertyIsEnumerable('capital')); // true```
Возвращает логическое значение, указывающее, является ли указанное свойство перечисляемым и собственным.

```const date = new Date(Date.UTC(2012, 11, 20, 3, 0, 0));```
```console.log(date.toLocaleString('ar-EG'));```
Возвращает строку, представляющую объект, специфичную для локали.

```console.log(10..toString(16)); // a```
```console.log(countryCanada.toString()); // [object Object]```
Возвращает строку, представляющую объект.

~~~
const countryCanada = {
  population: 4
}

countryCanada.valueOf = function() {
  return this.population;
};

console.log(countryCanada + 3); // 7
~~~

valueOf возвращает примитивное значение указанного объекта.

```console.log(countryCanada instanceof Object); // true```
Используется для определения того, является ли переменная экземпляром объекта
