# ?Installing and compiling

Чтобы начать работать с TypeScript, установим необходимый инструментарий. Установить TypeScript можно двумя способами: через пакетный менеджер или как плагин.

### Установка через `npm`

~~~
npm install -g typescript
~~~

Вполне возможно, что ранее уже был установлен TS. В этом случае его можно обновить до последней версии с помощью команды:

~~~
npm update -g typescript
~~~

Для проверки версии необходимо ввести команду:

~~~
tsc -v
~~~

### Компиляция приложения без `tsconfig.json`

Добавим файл `index.html` с `<script src="app.js"></script>`.

Создадим файл `app.ts`. Это обычный текстовый файл, который имеет расширение `.ts`.

app.ts
~~~
console.log("Hi");
~~~

Сохраним и скомпилируем этот файл. Для этого перейдем к каталогу, где расположен файл `app.ts`. И для компиляции выполним следующую команду:

~~~
tsc app.ts
~~~

После компиляции в каталоге проекта создается файл `app.js`, который будет выглядеть так:

~~~
console.log("Hi");
~~~

### Настройки компиляции

При компиляции файлов TypeScript из командной строки компилятор позволяет установить ряд конфигурационных настроек. Рассмотрим лишь основные из них.

#### Автоматическая перекомпиляция

Опция `--watch`, а также ее сокращенная версия `-w` автоматически перекомпилирует файлы Typescript, если в них были внесены какие-либо изменения. Благодаря чему не надо при каждом малейшем изменении вручную вводить команду в консоль для перекомпиляции.

~~~
tsc -w app.ts
~~~

#### Версия ECMAScript

С помощью параметра `–-target` или его сокращенной версии `–t` можно задать версию стандарта JavaScript, в которую будет компилироваться код TypeScript. Этот параметр может принимать следующие значения: `"ES3"` (по умолчанию), `"ES5"`, `"ES6"` / `"ES2015"`, `"ES7"` / `"ES2016"`, `"ES2017"`, `"ES2018"`, `"ES2019"`, `"ES2020"` или `"ESNext"`:

~~~
tsc app.ts -t ES5
~~~

#### Удаление комментариев

По умолчанию в файлы javascript переходят все комментариии, которыми снабжен код в файлах TS. Удаление комментариев при компиляции осуществляется с помощью параметра `–-removeComments`:

~~~
tsc app.ts --removeComments
~~~

#### Установка каталога

С помощью параметра `--outDir` можно задать папку для хранения скомпилированных файлов `js`:

~~~
tsc --outDir src/ index.ts
~~~

В данном случае скомпилированный файл `app.js` окажется в папке `src`.

#### Объединение файлов

Если у нас несколько файлов TS, то с помощью параметра `--outFile` их можно объединить в один файл `js`:

~~~
tsc --outFile output.js app.ts hello.ts
~~~

Здесь файлы `app.ts` и `hello.ts` скомпилируются в один файл `output.js`.

#### Тип модуля

С помощью параметра `--module`, либо `-m` можно указать тип модуля, который будет использоваться для компиляции. Эта опция может принимать следующие значения: `"None"`, `"CommonJS"` (значение по умолчанию, если задана версия ECMAScript `"ES3"` или `"ES5"`), `"AMD"`, `"System"`, `"UMD"`, `"ES2015"`, `"ES2020"` и `"ESNext"`.

~~~
tsc -m commonjs app.ts
~~~

#### Несколько параметров

Если надо задать несколько параметров, то они и их значения последовательно перечисляются через пробел.

~~~
tsc -t ES5 --outDir js -m commonjs app.ts
~~~

#### Вызов справки

И чтобы посмотреть все доступные параметры и справку по ним, можно воспользоваться параметром `-h`:

~~~
tsc -h
~~~

### Файл конфигурации `tsconfig.json`

С помощью файла `tsconfig.json` можно настроить проект TypeScript.

Для его использования достаточно вручную добавить новый файл с именем `tsconfig.json` в корень проекта.

`tsconfig.json` представляет собой стандартный файл в формате `json`, который содержит ряд секций. Так, секция `"compilerOptions"` настраивает параметры компиляции. Здесь можно указать необходимые параметры и их значения. Параметры называются также, как и в командной строке. И соответственно значения мы им можем передать те же, что передаются в командной строке (или терминале).

Файл `tsconfig.json` используется при компиляции в том случае, если компилятору не передаются названия файлов, которые надо скомпилировать. В этом случае компилятор TypeScript просматривает текущий каталог, ищет в нем файл `tsconfig.json` и затем при компиляции использует те параметры, которые определены в этом файле.

##### Если же компилятору передаются названия файлов, например, `tsc app.ts`, то файл `tsconfig.json` игнорируется.

~~~
{
  // Commented-out options have their default values.
  "include": ["src/**/*"],
  "exclude": ["node_modules/*"],
  // "files": [], // A list of relative or absolute file paths to include.
  // "extends": "", // A string containing a path to another configuration file to inherit from.
  // "references": [], // An array of objects `{"path": "./to/dirOrConfig"}` that specifies projects to reference.
  // "compileOnSave": false, // Signals to the IDE to generate all files for a given tsconfig.json upon saving.

  "compilerOptions": {
    // Main options
    "target": "esnext", // Specify ECMAScript target version: 'es3' (default), 'es5', 'es2015', 'es2016', 'es2017','es2018' or 'esnext'.
    "module": "esnext", // Specify module code generation: 'none', 'commonjs', 'amd', 'system', 'umd', 'es2015', or 'ESNext'.
    "lib": ["esnext", "dom"], // Specify library files to be included in the compilation.
    // "allowJs": false, // Allow javascript files to be compiled.
    // "checkJs": false, // Report errors in .js files.
    // "outFile": "./", // Concatenate and emit output to single file.
    // "outDir": "./", // Redirect output structure to the directory.
    // "rootDir": "./", // Specify the root directory of input files. Use to control the output directory structure with `--outDir`.
    // "project": "", // Compile a project given a valid configuration file.

    // Compilation options
    // "composite": true, // Enable project compilation: https://www.typescriptlang.org/docs/handbook/project-references.html
    // "diagnostics": false, // Show diagnostic information.
    // "incremental": true, // Enable incremental compilation by reading/writing information from prior compilations to a file on disk.
    // "isolatedModules": false, // Transpile each file as a separate module (similar to 'ts.transpileModule').
    // "listEmittedFiles": false, // Print names of generated files part of the compilation.
    // "listFiles": true, // Print names of files part of the compilation.
    // "noErrorTruncation": false, // Do not truncate error messages.
    // "preserveWatchOutput": false, // Keep outdated console output in watch mode instead of clearing the screen.
    // "traceResolution": false, // Enable tracing of the name resolution process.
    // "tsBuildInfoFile": ".tsbuildinfo", // Specify file to store incremental compilation information.

    // Strict typechecking options
    "strict": true, // Enable all strict type-checking options.
    // "noImplicitAny": true, // Raise error on expressions and declarations with an implied 'any' type.
    // "noImplicitThis": true, // Raise error on 'this' expressions with an implied 'any' type.
    // "strictBindCallApply": true, // Enable stricter checking of of the `bind`, `call`, and `apply` methods on functions.
    // "strictFunctionTypes": true, // Disable bivariant parameter checking for function types.
    // "strictNullChecks": true, // In strict null checking mode, the null and undefined values are not in the domain of every type and are only assignable to themselves and any.
    // "strictPropertyInitialization": true, // Ensure non-undefined class properties are initialized in the constructor. This option requires `--strictNullChecks` be enabled in order to take effect.
    // "alwaysStrict": true, // Parse in strict mode and emit "use strict" for each source file.

    // Additional checks
    // "allowUnreachableCode": false, // Do not report errors on unreachable code.
    // "allowUnusedLabels": false, // Do not report errors on unused labels.
    "forceConsistentCasingInFileNames": true, // Disallow inconsistently-cased references to the same file.
    // "noStrictGenericChecks": false, // Disable strict checking of generic signatures in function types.
    "noUnusedLocals": true, // Report errors on unused locals.
    "noUnusedParameters": true, // Report errors on unused parameters.
    "noImplicitReturns": true, // Report error when not all code paths in function return a value.
    "noFallthroughCasesInSwitch": true, // Report errors for fallthrough cases in switch statement.
    // "skipLibCheck": false, // Skip type checking of all declaration files (*.d.ts).
    // "suppressExcessPropertyErrors": false, // Suppress excess property checks for object literals.
    // "suppressImplicitAnyIndexErrors": false, // Suppress noImplicitAny errors for indexing objects lacking index signatures.

    // Module resolution options
    "moduleResolution": "node", // Specify module resolution strategy: 'node' (Node.js) or 'classic' (TypeScript pre-1.6).
    // "baseUrl": "./", // Base directory to resolve non-absolute module names.
    // "paths": {}, // A series of entries which re-map imports to lookup locations relative to the 'baseUrl'.
    // "rootDirs": [], // List of root folders whose combined content represents the structure of the project at runtime.
    // "typeRoots": [], // List of folders to include type definitions from.
    "types": [], // Type declaration files to be included in compilation.
    // "allowSyntheticDefaultImports": false // Allow default imports from modules with no default export. This does not affect code emit, just typechecking.
    // "esModuleInterop": false, // Emit '__importStar' and '__importDefault' helpers for runtime babel ecosystem compatibility and enable '--allowSyntheticDefaultImports' for typesystem compatibility.
    // "maxNodeModuleJsDepth": 0, // The maximum dependency depth to search under node_modules and load JavaScript files. Only applicable with --allowJs.
    // "preserveSymlinks": false, // Do not resolve the real path of symlinks.
    "resolveJsonModule": true // Include modules imported with '.json' extension.

    // Emit options
    // "declaration": false, // Generates corresponding '.d.ts' file.
    // "declarationDir": "./", // Output directory for generated declaration files.
    // "declarationMap": false, // Generates a sourcemap for each corresponding '.d.ts' file.
    // "emitBOM": false, // Emit a UTF-8 Byte Order Mark (BOM) in the beginning of output files.
    // "emitDeclarationOnly": false, // Only emit ‘.d.ts’ declaration files.
    // "importHelpers": false, // Import emit helpers from 'tslib'.
    // "newLine": "LF", // Use the specified end of line sequence to be used when emitting files: "crlf" (windows) or "lf" (unix).”
    // "noEmit": true, // Do not emit outputs.
    // "noEmitHelpers": false, // Do not generate custom helper functions like __extends in compiled output.
    // "noEmitOnError": false, // Do not emit outputs if any errors were reported.
    // "noImplicitUseStrict": false, // Do not emit "use strict" directives in module output.
    // "noResolve": false, // Do not add triple-slash references or module import targets to the list of compiled files.
    // "preserveConstEnums": false, // Do not erase const enum declarations in generated code.
    // "removeComments": false, // Remove all comments except copy-right header comments beginning with //!
    // "experimentalDecorators": true, // Enables experimental support for ES7 decorators.
    // "emitDecoratorMetadata": true, // Enables experimental support for emitting type metadata for decorators.

    // Source map options
    // "sourceMap": false, // Generates corresponding '.map' file.
    // "sourceRoot": "", // Specify the location where debugger should locate TypeScript files instead of source locations.
    // "mapRoot": "", // Specify the location where debugger should locate map files instead of generated locations.
    // "inlineSourceMap": true, // Emit a single file with source maps instead of having a separate file.
    // "inlineSources": true, // Emit the source alongside the sourcemaps within a single file; requires '--inlineSourceMap' or '--sourceMap' to be set.

    // JSX options
    // "jsx": "preserve", // Specify JSX code generation: 'preserve', 'react-native', or 'react'.
    // "jsxFactory": "React.createElement", // Specify the JSX factory function to use when targeting react JSX emit, e.g. 'React.createElement' or 'h'.

    // Other options
    // "allowUmdGlobalAccess": true, // Allow accessing UMD globals from modules.
    // "charset": "utf8", // The character set of the input files.
    // "downlevelIteration": false, // Provide full support for iterables in 'for-of', spread, and destructuring when targeting 'ES5' or 'ES3'.
    // "disableSizeLimit": false, // Disable size limitation on JavaScript project.
    // "keyofStringsOnly": false, // Resolve 'keyof' to string valued property names only (no numbers or symbols).
    // "noLib": false, // Do not include the default library file (lib.d.ts).
    // "pretty": true, // Stylize errors and messages using color and context.
  }
}
~~~

В настоящее время опция `compileOnSave` файла `tsconfig.json` игнорируется VS Code.

Эта опция и `ts-node`, которые можно встретить в root секции конфига, не являются официальными и используются IDE для своих целей.
