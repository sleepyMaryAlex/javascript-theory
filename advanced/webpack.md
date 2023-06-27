# ?Webpack

_Webpack_ — это сборщик модулей. Он анализирует модули приложения, создает граф зависимостей, затем собирает модули в правильном порядке в один или более бандл (bundle), на который может ссылаться файл `index.html`.

~~~
App.js ->       |
Dashboard.js -> | Bundler | -> bundle.js
About.js ->     |
~~~

### Какие проблемы решает вебпак?

Обычно, при создании приложения на JavaScript, код разделяется на несколько частей (модулей). Затем в файле `index.html` необходимо указать ссылку на каждый скрипт.

~~~
<body>
  // ...
  <script src='src/admin.js'></script>
  <script src='src/dashboard.js'></script>
  <script src='src/api.js'></script>
  <script src='src/auth.js'></script>
  <script src='src/rickastley.js'></script>
</body>
~~~

Это не только утомительно, но и подвержено ошибкам. Важно не только не забыть про какой-нибудь скрипт, но и расположить их в правильном порядке.

~~~
<body>
  // ...
  <script src='dist/bundle.js'></script>
</body>
~~~

Сбор модулей является лишь одним из аспектов работы вебпака. При необходимости можно заставить вебпак осуществить некоторые преобразования модулей перед их добавлением в бандл. Например, преобразование SASS/LESS в обычный CSS, или современного JavaScript в ES5 для старых браузеров.

### Установка вебпака

После инициализации проекта с помощью `npm`, для работы вебпака нужно установить два пакета — `webpack` и `webpack-cli`.

~~~
npm i webpack webpack-cli -D
~~~

### webpack.config.js

После установки указанных пакетов, вебпак нужно настроить. Для этого создается файл `webpack.config.js`, который экспортирует объект. Этот объект содержит настройки вебпака.

Основной задачей вебпака является анализ модулей, их опциональное преобразование и интеллектуальное объединение в один или более бандл, поэтому вебпаку нужно знать три вещи:

1. Точка входа приложения.
2. Преобразования, которые необходимо выполнить.
3. Место, в которое следует поместить сформированный бандл.

### Точка входа

Сколько бы модулей не содержало приложение, всегда имеется единственная точка входа. Этот модуль включает в себя остальные. Обычно, таким файлом является `index.js`. Это может выглядеть так:

~~~
index.js
  imports about.js
  imports dashboard.js
    imports graph.js
    imports auth.js
      imports api.js
~~~

Если мы сообщим вебпаку путь до этого файла, он использует его для создания графа зависимостей приложения. Для этого необходимо добавить свойство `entry` в настройки вебпака со значением пути к главному файлу. Например:

~~~
module.exports = {
  entry: './app/index.js'
}
~~~

### Преобразования с помощью лоадеров (loaders)

После добавления точки входа, нужно сообщить вебпаку о преобразованиях, которые необходимо выполнить перед генерацией бандла. Для этого используются лоадеры.

По умолчанию при создании графика зависимостей на основе операторов `import`/`require()` вебпак способен обрабатывать только JavaScript и JSON-файлы.

Едва ли в своем приложении вы решитесь ограничиться JS и JSON-файлами, скорее всего, вам также потребуются стили, SVG, изображения и т.д. Вот где нужны лоадеры. Основной задачей лоадеров, как следует из их названия, является предоставление вебпаку возможности работать не только с JS и JSON-файлами.

Первым делом нужно установить лоадер. Если мы хотим загружать SVG, с помощью `npm` устанавливаем `svg-loader`.

~~~
npm i svg-inline-loader -D 
~~~

Далее добавляем его в настройки вебпака. Все лоадеры включаются в массив объектов `module.rules`.

Информация о лоадере состоит из двух частей. Первая — тип обрабатываемых файлов (`.svg` в нашем случае). Вторая — лоадер, используемый для обработки данного типа файлов (`svg-inline-loader` в нашем случае).

~~~
module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' }
    ]
  }
}
~~~

Теперь мы можем импортировать SVG-файлы. Но что насчет наших CSS-файлов? Для стилей используется `css-loader`.

~~~
npm i css-loader -D 
~~~

И для того, чтобы наши стили работали корректно, нужно добавить еще один лоадер. Благодаря `css-loader` мы можем импортировать CSS-файлы. Но это не означает, что они будут включены в DOM. Мы хотим не только импортировать такие файлы, но и поместить их в тег `<style>`, чтобы они применялись к элементам DOM. Для этого нужен `style-loader`.

~~~
npm i style-loader -D 
~~~

~~~
module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' },
      { test: /\.css$/, use: [ 'style-loader', 'css-loader' ] }
    ]
  }
}
~~~

Поскольку для обработки CSS-файлов используется два лоадера, значением свойства `use` является массив. Также обратите внимание на порядок следования лоадеров, сначала `style-loader`, затем `css-loader`. Это важно. Вебпак будет применять их в обратном порядке. Сначала он использует `css-loader` для импорта `'./styles.css'`, затем `style-loader` для внедрения стилей в DOM.

Лоадеры могут использоваться не только для импорта файлов, но и для их преобразования.

`babel-loader` — позволяет писать код на современном JS, но исполнять его даже в старых браузерах. `babel-loader` преобразует "новый" JS и JSX в "старый" JS. Т.е. итоговый файл `bundle.js` может отличаться с `babel-loader` и без него.

~~~
npm i babel-loader -D 
~~~

~~~
module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' },
      { test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
      { test: /\.(js)$/, use: 'babel-loader' }
    ]
  }
}
~~~

Существуют лоадеры почти для любого типа файлов.

### Точка выхода

Теперь вебпак знает о точке входа и лоадерах. Следующим шагом является указание директории для бандла. Для этого нужно добавить свойство `output` в настройки вебпака.

~~~
const path = require('path')

module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' },
      { test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
      { test: /\.(js)$/, use: 'babel-loader' }
    ]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  }
}
~~~

Метод `path.resolve()` используется для преобразования последовательности сегментов пути в абсолютный путь.

`__dirname` - это переменная среды, которая сообщает вам абсолютный путь к каталогу, содержащему исполняемый в данный момент файл.

Здесь мы создаем новый каталог `dist`.

Весь процесс выглядит примерно так:

1. Вебпак получает точку входа, находящуюся в `./app/index.js`.
2. Он анализирует операторы `import`/`require` и создает граф зависимостей.
3. Вебпак начинает собирать бандл, преобразовывая код с помощью соответствующих лоадеров.
4. Он собирает бандл и помещает его в `dist/bundle.js`.

### Плагины (plugins)

Мы рассмотрели, как использовать лоадеры для обработки отдельных файлов перед или в процессе генерации бандла. В отличие от лоадеров, плагины позволяют выполнять задачи после сборки бандла. Эти задачи могут касаться как самого бандла, так и другого кода. Можно думать о плагинах как о более мощных, менее ограниченных лоадерах.

#### HtmlWebpackPlugin

Основной задачей вебпака является генерация бандла, на который можно сослаться в `index.html`.

`HtmlWebpackPlugin` создает `index.html` в директории с бандлом и автоматически добавляет в него ссылку на бандл.

`HtmlWebpackPlugin` создаст новый файл `index.html` в директории `dist` и добавит в него ссылку на бандл.

Как обычно, сначала его нужно установить.

~~~
npm i html-webpack-plugin -D 
~~~

Создаем экземпляр `HtmlWebpackPlugin` в массиве `plugins`.

~~~
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' },
      { test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
      { test: /\.(js)$/, use: 'babel-loader' }
    ]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlWebpackPlugin()
  ]
}
~~~

#### CopyPlugin

`CopyPlugin` копирует отдельные файлы или целые каталоги, которые уже существуют, в каталог сборки.

Установка:

~~~
npm i copy-webpack-plugin -D
~~~

~~~
const CopyPlugin = require("copy-webpack-plugin");

module.exports = {
  // ...
  plugins: [
    new CopyPlugin({
      patterns: [
        {
          from: path.resolve(__dirname, "assets"),
          to: path.resolve(__dirname, "dist"),
        },
      ],
    }),
  ],
  // ...
};
~~~

### resolve

`resolve` определяет порядок разрешения модулей. Значением `resolve` является объект.

По умолчанию вебпак не будет разрешать файлы `.jsx` при импорте без расширения. Чтобы настроить его для включения файлов JSX, настройте, как показано ниже:

~~~
module.exports = {
  // ...
  resolve: {
    extensions: [
      '.js',
      '.jsx'
    ]
  },
  // ...
};
~~~

Если несколько файлов имеют одно и то же имя, но разные расширения, `webpack` разрешит файл с расширением, указанным первым в массиве, и пропустит остальные.

Именно это позволяет не указывать расширение при импорте:

~~~
import { animal } from "./animal";
~~~

Использование вышеописанного `resolve.extensions` приведет к переопределению массива по умолчанию.

### Режим (mode)

В настройках вебпака можно установить mode в значение `development` или `production` в зависимости от среды разработки.

После установки `mode` в значение `production` вебпак автоматически присваивает `process.env.NODE_ENV` значение `production`. Это также минифицирует код и удаляет предупреждения.

Давайте рассмотрим, как переключаться между режимами.

При сборке для продакшна, мы хотим все оптимизировать, насколько это возможно. В случае с режимом разработки верно обратное.

Для переключения между режимами необходимо создать два скрипта в `package.json`.

1. `npm run build` будет собирать продакшн-бандл.
2. `npm run start` будет запускать сервер для разработки и следить за изменениями файлов.

Немного изменим `package.json`:

~~~
"scripts": {
  "build": "SET NODE_ENV='production' && webpack",
  "start": "webpack-dev-server"
},
~~~

Теперь в настроках вебпака мы можем менять значение `mode` в зависимости от `process.env.NODE_ENV`.

~~~
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './app/index.js',
  module: {
    rules: [
      { test: /\.svg$/, use: 'svg-inline-loader' },
      { test: /\.css$/, use: [ 'style-loader', 'css-loader' ] },
      { test: /\.(js)$/, use: 'babel-loader' }
    ]
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlWebpackPlugin()
  ],
  mode: process.env.NODE_ENV === 'production' ? 'production' : 'development'
}
~~~

Для сборки готового бандла для нашего приложения мы просто запускаем `npm run build` в терминале. В директории `dist` создаются файлы `index.html` и `bundle.js`.

### Сервер для разработки

Когда речь идет о разработке приложения принципиально важное значение имеет скорость. Мы не хотим перезапускать вебпак и ждать новую сборку при каждом изменении. Вот где нам пригодится пакет `webpack-dev-server`.

Как следует из названия, это вебпак-сервер для разработки. Вместо создания директории `dist`, он хранит данные в памяти и обрабатывает их на локальном сервере. Более того, он поддерживает живую перезагрузку. Это означает, что при любом изменении `webpack-dev-server` пересоберет файлы и перезапустит браузер.

~~~
npm i webpack-dev-server -D 
~~~

### Пример

index.html
~~~
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title></title>
</head>

<body>
</body>

</html>
~~~

index.js
~~~
import "./style.css";
import cat from "./assets/cat.svg";
import { animal } from "./animal";

console.log(animal.name);

const user = { firstName: "John" };
const userExtended = { ...user, lastName: "Doe" };
console.log(userExtended);

const main = `<h1>Hi</h1>`;
document.body.insertAdjacentHTML("afterbegin", main);

function component() {
  const element = document.createElement("div");
  element.innerHTML = cat;
  return element;
}

document.body.appendChild(component());
~~~

style.css
~~~
h1 {
  color: orange;
}

svg {
  width: 100px;
}
~~~

animal.js
~~~
export const animal = {
  name: "Cat",
};
~~~

package.json
~~~
{
  "dependencies": {
    "debug": "^4.3.4",
    "node-static": "^0.7.11",
    "platform": "^1.3.6"
  },
  "scripts": {
    "build": "SET NODE_ENV='production' && webpack",
    "start": "webpack-dev-server"
  },
  "name": "javascript",
  "version": "1.0.0",
  "main": "index.js",
  "author": "",
  "license": "ISC",
  "description": "",
  "devDependencies": {
    "@types/node": "^20.2.5",
    "babel-loader": "^9.1.2",
    "copy-webpack-plugin": "^11.0.0",
    "css-loader": "^6.8.1",
    "html-webpack-plugin": "^5.5.3",
    "style-loader": "^3.3.3",
    "svg-inline-loader": "^0.8.2",
    "webpack": "^5.88.0",
    "webpack-cli": "^5.1.4",
    "webpack-dev-server": "^4.15.1"
  }
}
~~~

webpack.config.js
~~~
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: "./index.js",
  module: {
    rules: [
      { test: /\.svg$/, use: "svg-inline-loader" },
      { test: /\.css$/, use: ["style-loader", "css-loader"] },
      { test: /\.(js)$/, use: "babel-loader" },
    ],
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "bundle.js",
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: "My app",
    }),
  ],
  mode: process.env.NODE_ENV === "production" ? "production" : "development",
};
~~~

В `html-webpack-plugin` можно поредать `title`, который является заголовком для сгенерированного HTML-документа. По умолчанию `Webpack App`.

dist/index.html
~~~
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My app</title>
  <meta name="viewport" content="width=device-width, initial-scale=1"><script defer src="bundle.js"></script></head>
  <body>
  </body>
</html>
~~~
