# 前言
文章篇幅有点长，需要花费大概十分钟的时间阅读。若看懂需求后，可跳过个人分析，直接看后面的代码。
若是喜欢求start下，如有疑问可提出issues讨论

完整的代码地址：
https://github.com/FredaFei/mutile-webpack

    git clone git@github.com:FredaFei/mutile-webpack.git
    
运行命令：

    // 开发环境
    npm start
    // 全部打包在一起
    npm run build [project name] [project name]
    // 每个项目单独打包
    npm run build [project name] [project name] separate
    // 打包所有项目，并每个项目单独打包
    npm run build-all
    
通常我们在用vue开发项目时，用的是vue-cli脚手架搭建的环境（默认是单页面配置）。

## 场景描述
然而在实际的工作中遇到多个单页面项目放一起的时候，比如运营部门的活动，会了方便管理会考虑将所有的活动都放在一个项目下管理，抽取公共的逻辑，再对每个活动单独开发。

**此时需要考虑下发布方案了：**
**方案一**：每次上线（测试或正式环境）都对所有的活动重新打包发布；
**方案二**：指定活动项目名打包发布；

**如何选择？**

若选择方案一，像活动这种**量大、项目小、生命短**的项目，在日积月累中项目会越来越多，打包上线的时间也越来越长，反而不利于管理与维护，属于**甜一阵子苦一辈子**；

若选择方案二，前期需要熟悉webpack配置和node基本模块的知识，在项目的基础搭建上需要花些时间和心力，属于**苦一阵子甜一辈子**。

聪明的读者知道怎么选择了吗？

## 确认需求：

修改配置前（默认打包配置）：

![图片标题](https://leanote.com/api/file/getImage?fileId=5b8909e3ab6441259d001dfe)

修改配置后：

![图片标题](https://leanote.com/api/file/getImage?fileId=5b8909d9ab644123a8001f28)

------

下面我们就来说说怎么实现吧！
## 实现思路：

**在自己搭建打包环境的基础下：**

 1. 在改项目中的`package.json`文件中增加字段```pages: module_1,module_2```，作为需要打包的模块参数；
 2. 新建`webpack.config.js`文件，在里面分别配置基本的公用配置，开发环境的配置，以及打包的配置；
 3. 根据不同的运行环境，与第二步中的基本配置合并；
 4. 获取`package.json`中`pages`值并将它循环遍历，作为每次打包的项目名，结合第三步得到当前打包项目的完整配置；
 5. 在`package.json`文件中配置启动命令```"build": "webpack --env=[生产环境]"```;
 
具体配置信息可参考此仓库代码
https://github.com/lihongbin100/any-page-demo

总结： 每次需要打包什么项目就在`package.json`中配置好```pages```字段就可以了。
优点：配置简单，实现起来也比较容易。
缺点：每次都要手动更改```pages```字段，若需要对所有项目打包，该字段输入将较为冗长。
不过这个缺点是可以通过其他方式来解决，比如给定```"pages"： "all"```来判断是否打包全部，来指定需要打包的项目名。

**在使用vue-cli搭建的基础环境下**

1. 沿用`package.json`中的```pages```字段来指定需要打包的项目名；
2. 根据打包项目配置入口参数和出口参数；
3. 运行打包命令；

以上思路虽看似可行，然而需要遍历需要打包的项目来指定当前所打包的出口文件配置。
默认该项目执行```npm run build```时，`package.json`配置的是`"build": "node build/build.js"`，`build.js`中引用到了`config/index.js`中的`build: {...}`，这个对象里写有打包文件的输出配置。
如此若真遍历时我需要传当前项目名给它来指定输出配置，这可就不好办了，看来不能沿用`package.json`中的```pages```字段来指定需要打包的项目名。

**怎么来指定需要打包项目呢？**
想到node里有一个内置API，process.argv，第一个元素是node，第二个元素是js文件的文件名。接下来的元素则是附加的命令行参数。例如：

    $ node index.js one two three four
    0: node
    1: D:/test/app/index.js
    2: one
    3: two
    4: three
    5: four

可以通过 `npm run build module_1`，其中的`module_1`便是需要打包的项目名，通过process.argv[2]取得此值，将它放在`config/index.js`中的`build: {...}`中，如此就解决了打包出口文件的配置问题。

以上方式只能实现一个一个项目的打包，对多个项目的打包还是没能实现。
又是想了许久。。。最后是在谷歌上面搜索时看到了别人做多项目打包配置的文章用node的子进程，这才得以解决。

## 具体配置来了

在`build/utils.js`文件下新增如下代码：

    const glob = require("glob");
    const baseViewPath = path.resolve(__dirname, "../src/views");
    const packageViews = process.env.MODULE_ENV || '';
    
    /**
     * 读取入口文件和模板文件
     * @param basePath {string}
     * @param type {string}
     * @return {object}
    */
    function getEntryAndTemplate() {
      let ctx = { entries: {}, template: {},moduleList: [] };
      // 对所有项目模块打包
      if (!packageViews) {
        ctx.entries = packageAllFiles(baseViewPath, "js");
        ctx.template = packageAllFiles(baseViewPath, "html");
      }
      //针对指定的项目模块打包
      else {
        ctx.entries[packageViews] = `${baseViewPath}/${packageViews}/main.js`;
        ctx.template[packageViews] = `${baseViewPath}/${packageViews}/template.html`;
      }
      ctx.moduleList = Object.keys(ctx.entries)
      return ctx;
    }
    
    /**
     * 读取指定目錄下的指定文件類型
     * @param basePath {string}
     * @param type {string}
     * @return {object}
     */
    function packageAllFiles(basePath, type) {
      let temp = glob.sync(`${basePath}/\*/\*.${type}`);
      let map = {};
      temp.forEach(pathItem => {
        let arr = pathItem.split("/");
        let _filename = arr[arr.length - 2];
        map[_filename] = pathItem;
      });
      return map;
    }
    const moduleCtx = getEntryAndTemplate();
    exports.moduleCtx = moduleCtx
    
    // 多入口配置
    exports.entries = moduleCtx.entries;

修改`build/webpack.base.conf.js`中的`entry`

    module.exports = {
      context: path.resolve(__dirname, "../"),
      entry: utils.entries,
      ......
    }

在`build/webpack.dev.conf.js`中新增

    function devHtmlPlugin() {
      let { moduleList, template } = utils.moduleCtx
      let tempArr = [];
      for (let item of moduleList){
        let conf = {
          filename: `${item}/index.html`,
          template: template[item],
          chunks: [item],
          inject: true
        };
        tempArr.push(new HtmlWebpackPlugin(conf));
      }
      return tempArr;
    };
    const devWebpackConfig = merge(baseWebpackConfig, {
        ...,
        plugins: [...].concat(devHtmlPlugin())
    })
    
在`config/index.js`中新增和修改

    const moduleName = process.env.MODULE_ENV || '';
    // 入口模板路径
    const htmlTemplate = `./src/views/${moduleName}/index.html`;
    module.exports = {
      dev: {...},
      build: {
        // Template for index.html
        index: path.resolve(__dirname, '../dist', moduleName, 'index.html'),
        htmlTemplate: htmlTemplate,
        // Paths
        assetsRoot: path.resolve(__dirname, '../dist', moduleName),
        assetsSubDirectory: 'static',
        assetsPublicPath: '',
        ...
      }
   
在`build`文件下新建如下两个文件

    module-conf.js、build-all.js
    
`build-all.js`代码如下

    const path = require('path')
    const execFileSync = require('child_process').execFileSync
    const { moduleList } = require("./utils").moduleCtx
    const buildFile = path.join(__dirname, 'build.js')
    
    console.log(moduleList);
    for(let module of moduleList){
      console.log('正在编译：', module)
      execFileSync("node", [buildFile, module, "separate"], {});
    }

`module-conf.js`代码如下

    const chalk = require('chalk')
    const glob = require('glob')
    const moduleBasePath = './src/views'
    const moduleList = []
    const allModule = glob.sync(`${moduleBasePath}/*`) //当前执行node命令时项目目录名  process.cwd()
    allModule.forEach(item=>{
      moduleList.push(item.split('/')[3])
    })
    
    exports.checkedModule = function () {
      const currentModule = process.env.MODULE_ENV
      const repeactModule = moduleList.filter((item,index)=>{
        return moduleList.indexOf(item) !== index
      })
      if(repeactModule.length>0){
        console.log(chalk.red('moduleList 重复了'))
        console.log(chalk.red(repeactModule.join()))
        return false
      }
      let illegal
      let result = true
      for(let item of currentModule.split(',')){
        if(moduleList.indexOf(item) === -1){
          result = false
          illegal = item
          break
        }
      }
      if(result === false){
        console.log(chalk.red('参数错误，允许的参数为：'))
        console.log(chalk.green(moduleList.join()))
        console.log(chalk.yellow('非法参数有：' + illegal))
      }
      return result
    }   
   
在`build/build.js`中新增   

    process.env.NODE_ENV = 'production'
    process.env.MODULE_ENV = process.argv[2];
    process.env.MODE_ENV = process.argv[3];
    
    //打包前检测参数是否非法
    const checkedModule = require('./module-conf').checkedModule
    if(process.env.MODULE_ENV !== ''  && !checkedModule()){
      return false
    }
       
配置package.json

    ...
     "scripts": {
        "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
        "start": "npm run dev",
        "clear": "rm -rf ./dist",
        "build": "npm run clear && node build/build.js",
        "build-all": "npm run clear && node build/build-all.js",
        ...
      },
      ...
    
    
参考文章：

+ [基于vue-cli搭建多模块且各模块独立打包的项目](https://segmentfault.com/a/1190000014571631)



