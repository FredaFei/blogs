### JS模块化

**什么是模块化？**
>将计算机程序的功能分离成独立的、可相互改变的“模块”（module）的软件设计技术，它使得每个模块都包含着执行预期功能...

[维基百科](https://zh.wikipedia.org/zh-hans/%E6%A8%A1%E5%9D%97%E5%8C%96%E7%BC%96%E7%A8%8B)

**为什么要模块化？**

将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。

**JS有实现该功能吗？**

ES6之前没有，但民间有制定一些模块加载方案，主要有**CommonJS** 和 **AMD**。

+ CommonJS

    + 主要应用于node（服务端） ，模块功能由以下命令构成`module`、`exports`、`require`；
    + 运行时加载(加载结果为一个对象)，无法在编译时做`静态分析`；
    + 重复引入同一模块并不会重复执行，复用之前的缓存；
    + 模块引入位置对输出无影响，模块输出的是一个值的拷贝；
+ AMD

    + 主要应用于浏览器 

ES6时推出了模块的设计思想

+ Modules 语法

    + 模块功能由以下命令构成`export`、`import`；
    + 多次重复执行同一句import语句，只会执行一次，而不会执行多次；
    + 编译时加载，可静态分析；
    + 模块输出的是一个值的引用；
    + ES2020提案 引入import()函数，支持动态加载模块，与Node的`require`类似，但前者为异步加载，后者为同步加载。其使用场景：
        + 按需加载；exp: `btn.onClick = (event)=>{import(f()).then(...);}`
        + 条件加载；exp: `if(condition){import('A').then(...);}`
        + 动态的模块路径；exp: `import(f()).then(...);`


**ES6 模块与 CommonJS 模块的差异**

> 它们有三个重大差异:
  CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
  CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
  CommonJS 模块的require()是同步加载模块，ES6 模块的import命令是异步加载，有一个独立的模块依赖的解析阶段。

**循环加载**

+ CommonJS 模块是加载时执行，即脚本代码在require的时候，就会全部执行。一旦出现某个模块被"循环加载"，就只输出已经执行的部分，还未执行的部分不会输出。
+ ES6 模块是动态引用，例如`import foo from 'foo'`，其引入的那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。

**UMD是什么？**

+ Universal Module Definition - 通用模块定义模式，该模式主要用来解决CommonJS模式和AMD模式代码不能通用的问题，并同时还支持老式的全局变量规范。
+ UMD = CommonJS + AMD

参考阅读：

+[es6 module](https://es6.ruanyifeng.com/#docs/module)