## Webpack 核心原理

#### Webpack 是什么？

> webpack is a static module bundler for modern JavaScript applications

处理JS应用程序的【静态模块打包器】

#### Webpack 核心概念？

+ Entry，打包入口
+ Output，打包完成后的输出地址
+ Loaders，处理各种类型文件，如`less-loader`
+ Plugins，扩展打包功能，如`html-webpack-plugin`

#### Webpack 工作流程？

+ 初始化配置（webpack.config.js）；
+ 递归分析模块、构建其依赖；
    + 对模块位置进行解析，开始构建某个模块；
    + 执行对应的loader，将loader解析后的模块进行编译，生成AST树；
    + 遍历AST，遇到`require`表达式时，收集依赖；
+ 所有依赖构建完成，执行plugin；
+ 输出到指定的目录，打包结束。

#### 实现一个简易的 webpack

##### 准备工作

初始化项目，目录结构如下：

```
tiny-webpack
├── dist
│   ├── bundle.js
│   ├── index.html
├── build // webpack配置目录
│   ├── webpack.config.js
├── lib  // 手写webpack源码目录
│   ├── compier.js
├── src 
│   ├── index.html
│   ├── index.js
├── index.js
├── package.json
```

依赖安装

```
npm install babel-core babel-preset-env @babel/parser @babel/traverse
```

+ @babel/parser：用于将源码生成AST
+ @babel/traverse：对AST节点进行递归遍历
+ babel-core、babel-preset-env：将获得的ES6的AST转化成ES5

小试牛刀：ES6转化成ES5

``` js
// compiler.js
const fs = require('fs');
const path = require('path');
const parser = require('@babel/parser');
const { transformFromAst } = require('babel-core');

function writeFileRecursive(outputDir, outputFilename, buffer, callback) {
    fs.mkdir(outputDir, { recursive: true }, (err) => {
        if (err) return callback(err);
        fs.writeFile(outputFilename, buffer, function (err) {
            if (err) return callback(err);
            return callback(null);
        });
    });
};

module.exports = function run(options) {
    const { path: outputDir, filename } = options.output;
    const outputFilename = path.join(outputDir, filename);

    const source = fs.readFileSync(options.entry, 'utf-8');
    const parsedAst = parser.parse(source, {
        sourceType: 'module', //表示要解析的是ES模块
    });
    // 将获得的ES6的AST转化成ES5
    const { code } = transformFromAst(parsedAst, null, {
        presets: [ 'env' ],
    });

    writeFileRecursive(outputDir, outputFilename, code, error => {
        console.log(error);
    });
};
```

``` js
// index.js
const run = require('./lib/compiler');
const webpackConfig = require('./build/webpack.config');

run(webpackConfig);
```

``` js
// webpack.config.js
const path = require('path');

module.exports = {
    entry: path.join(__dirname, '../src/index.js'),
    output: {
        path: path.join(__dirname, '../dist'),
        filename: 'bundle.js'
    }
};
```

以上约40行代码，实现了如下功能：

+ 获取指定入口的代码；
+ 将其从ES6转译为ES5；
+ 最后输出到指定目录。

##### 手写Webpack

代码如下：

```js
// parser.js
const fs = require('fs');
const path = require('path');
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const { transformFromAst } = require('babel-core');

module.exports = {
    getAST(path) {
        const source = fs.readFileSync(path, 'utf-8');
        return parser.parse(source, {
            sourceType: 'module', //表示要解析的是ES模块
        });
    },
    getDependencies(ast) {
        const dependencies = [];
        traverse(ast, {
            ImportDeclaration({ node }) {
                dependencies.push(node.source.value);
            }
        });
        return dependencies;
    },
    transform(ast) {
        // 将获得的ES6的AST转化成ES5
        const { code } = transformFromAst(ast, null, {
            presets: [ 'env' ],
        });
        return code;
    },
};
```

```js
// compiler.js
const fs = require('fs');
const path = require('path');
const { getAST, getDependencies, transform } = require('./parser');

function writeFileRecursive(outputDir, outputFilename, buffer, callback) {
    fs.mkdir(outputDir, { recursive: true }, (err) => {
        if (err) return callback(err);
        fs.writeFile(outputFilename, buffer, function (err) {
            if (err) return callback(err);
            return callback(null);
        });
    });
};

/**
 * 解决Linux与Windows下路径分隔符问题
 * */
function formatPathSep(string) {
    return string.split(path.sep).join('/');
}

module.exports = class Compiler {
    constructor(options) {
        const { entry, output } = options;
        this.entry = formatPathSep(entry);
        this.output = output;
        this.modules = [];
    }

    run() {
        this.buildDependencies();
        this.emitFiles();
    };

    buildModule(filename, isEntry) {
        const _path = isEntry ? filename : path.join(process.cwd(), './src', filename);
        const ast = getAST(formatPathSep(_path));
        return {
            filename: formatPathSep(filename),
            dependencies: getDependencies(ast),
            transformCode: transform(ast)
        };
    }

    buildDependencies() {
        const entryModule = this.buildModule(this.entry, true);
        this.modules.push(entryModule);
        this.modules.map((_module) => {
            _module.dependencies.map((dependency) => {
                this.modules.push(this.buildModule(dependency));
            });
        });
    }

    bundle() {
        let result = '';
        this.modules.map((_module) => {
            result += `'${_module.filename}' : function(require, module, exports) {${_module.transformCode}},`;
        });
        return `
            (function(modules) {
              function require(fileName) {
                const fn = modules[fileName];
                const module = { exports: {}};
                fn(require, module, module.exports)
                return module.exports
              }
              require('${this.entry}')
            })({${result}})
        `;
    }

    emitFiles() {
        const { path: outputDir, filename } = this.output;
        const outputPath = path.join(outputDir, filename);
        writeFileRecursive(outputDir, outputPath, this.bundle(), error => {
            console.log(error);
        });
    }
};

```

`buildDependencies`函数功能是收集模块之间的依赖关系，解析后表示如下：

```js
  [
    {
        filename: '/src/index.js',
        dependencies: [ './util/add.js', './util/multiply.js' ],
        transformCode: '"use strict";...'
    },
    {
        filename: './util/add.js',
        dependencies: [],
        transformCode: '"use strict";...'
    },
    // ...
]
```

`require`时，根据`filename`来执行对应的代码。

`bundle`函数功能是模拟`Webpack`文件打包机制。

### `Webpack`文件打包机制

经Webpack打包后的文件，其格式形式如下

```js
  (function (modules) {
    // ...
    function __webpack_require__(id) {
        // ...
        modules[id].call(module.exports, module, module.exports, __webpack_require__)
        // ...
        return module.exports
    }

    __webpack_require__(0)
})([
    (function (require, module, __webpack_require__) { // ... }),    
        (function (require, module, __webpack_require__) { // ... }),
            // ...
        ]);
```

以上代码作用如下：

+ 将打包文件以匿名理解执行函数的格式输出；
+ `modules` 是一个数组，每一项是一个模块初始化函数；
+ `__webpack_require__`用来加载模块，返回`module.exports`
+ `__webpack_require__(0)`开始调用

运行`npm run build`，`bundle.js`代码如下

```js
(function (modules) {
    function require(fileName) {
        const fn = modules[fileName];
        const module = { exports: {} };
        fn(require, module, module.exports);
        return module.exports;
    }

    require('D:/tiny-webpack/src/index.js');
})({
    'D:/tiny-webpack/src/index.js': function (require, module, exports) {
        'use strict';

        var _add = require('./util/add.js');

        var _multiply = require('./util/multiply.js');

        var a = (0, _add.add)(1, 4);
        var b = (0, _multiply.multiply)(2, 4);
        document.body.innerHTML = '<div>add: ' + a + '</div><div>multiply: ' + b + '</div>';
    }, './util/add.js': function (require, module, exports) {
        'use strict';

        Object.defineProperty(exports, '__esModule', {
            value: true
        });

        var add = exports.add = function add(x, y) {
            return x + y;
        };
    }, './util/multiply.js': function (require, module, exports) {
        'use strict';

        Object.defineProperty(exports, '__esModule', {
            value: true
        });
        exports.multiply = multiply;

        function multiply(x, y) {
            return x * y;
        }
    },
});

```
以上是整个手写 Webpack 打包实现的全部内容。

[完整代码](https://github.com/FredaFei/tiny-webpack)

#### 待完善之处：

+ 手写 loader
+ 手写 plugin
+ 其他
