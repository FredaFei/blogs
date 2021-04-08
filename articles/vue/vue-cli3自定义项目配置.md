vue脚手架工具vue-cli更新到版本3，项目配置隐藏了，目录结构清晰，开发者只需要关心项目的业务开发。
若需要根据项目来自定义配置，vue-cli3提供了一个配置文件`vue.config.js`，相关参数可参考[文档](https://cli.vuejs.org/config/#vue-config-js)。

需求：项目A有App端、H5端，需求原型已定。审核时，有些页面（宣传、协议等页面）长期迭代存在，不同于活动存在时间短。App团队不想开发，
让H5团队的人做一套适应两端。这时H5端为了便于管理，把App中用到的页面和H5项目分目录放在一个环境配置中，H5项目分模块划分页面，而不是整个项目用一个单页面应用。这样只需要维护一个H5项目。目录结构如下：

```
  |──[项目名] 
    |── public                            
    |   |── favicon.ico                 
    |   |── index.html                 
    |── src
    |   |── assets             
    |   |   |── |...           
    |   |── components                  // 公共组件
    |   |── pages                       // 主要视图模块
    |   |   |── app                     // App页面模块
    |   |   |   |── view_1
    |   |   |   |   |── images     
    |   |   |   |   |── |── ...              
    |   |   |   |   |── app.vue                 
    |   |   |   |   |── main.js    
    |   |   |   |── ...             
    |   |   |── modules                         // H5业务模块
    |   |   |   |── view_1
    |   |   |   |   |── images     
    |   |   |   |   |── router                  // 业务模块路由
    |   |   |   |   |── views     
    |   |   |   |   |── |── components         // 业务模块公共组件
    |   |   |   |   |── |── [业务模块1].vue              
    |   |   |   |   |── |── [业务模块1.1].vue              
    |   |   |   |   |── |── [业务模块1.2].vue              
    |   |   |   |   |── |── ...              
    |   |   |   |   |── app.vue                 
    |   |   |   |   |── main.js    
    |   |   |   |── view_2
    |   |   |   |   |── images     
    |   |   |   |   |── router     
    |   |   |   |   |── views     
    |   |   |   |   |── |── components              
    |   |   |   |   |── |── [业务模块1].vue              
    |   |   |   |   |── |── [业务模块1.1].vue              
    |   |   |   |   |── |── [业务模块1.2].vue              
    |   |   |   |   |── |── ...              
    |   |   |   |   |── app.vue                 
    |   |   |   |   |── main.js                
    |   |   |   |── ...  
    |   |—— ...   
    |── ...
```

modules目录下的文件是主要业务模块，在打包上与App分别打包。打包后的dist目录结构如下：

```
  |──[项目名] 
    |── css                            
    |   |── ...  
    |── img                            
    |   |── ...  
    |── js                   
    |   |── ...   
    |── app                   
    |   |── view_1.html             // App页面模块        
    |   |── view_2.html             // App页面模块        
    |   |── ...                    
    |── view_1                     // H5业务模块(view_1)
    |   |── index.html             // view_1模块 index.html 页面        
    |   |── about.html             // view_1模块 about.html 页面     
    |   |── ...                    
    |── view_2                     // H5业务模块(view_2)
    |   |── index.html             // view_2模块 index.html 页面       
    |   |── about.html             // view_2模块 about.html 页面        
    |   |── ...   
    |── view_3.html                // H5业务模块(view_3)view_3.html 页面 
    |   |── ...    
    
```

`vue.config.js`配置如下：

```js
const path = require('path')
const glob = require('glob')
const colors = require('colors-console')
const pages = {}

function getPages() {
    createPageModule('modules/a', false)
    createPageModule('modules/b', false)
    createPageModule('modules/c')
    createPageModule('app')
    return pages
}

function computedDirName(moduleName, hasChild) {
    if (!hasChild) {
        return ''
    }
    let dirName = ''
    let names = moduleName.split('/')
    let len = names.length
    names.length === 0
    if (hasChild) {
        if (len === 0) {
            dirName = moduleName
        } else {
            dirName = names[len - 1]
        }
    }
    return dirName ? `${dirName}/` : ''
}

function createPageModule(moduleName, hasChild = true) {
    const after = hasChild ? `**/main.js` : `main.js`
    glob.sync(`./src/pages/${moduleName}/${after}`).forEach(function(pageUrl) {
        const temp1 = pageUrl.split(`./src/pages/${moduleName}/`)
        const temp2 = moduleName.split('/')
        const pageCode = hasChild ? temp1[1].split('/main.js')[0] : temp2[temp2.length - 1]
        if (pages[pageCode]) {
            console.error(colors('red', `项目名不能重复使用：${pageCode}。\n`))
            process.exit(1)
        }

        // 配置多页面
        pages[pageCode] = {
            entry: pageUrl,
            template: './public/index.html',
            filename: `${computedDirName(moduleName, hasChild)}${pageCode}.html`
        }
    })
}

module.exports = {
    publicPath: process.env.NODE_ENV === 'production' ? '/h5' : '/',
    pages: getPages(),
    css: {
        loaderOptions: {
            sass: {
                sassOptions: {
                    // 添加如下一行后 原本 import '[相对路径]/styles/_var' 可替换为  @import "var";
                    includePaths: [path.join(__dirname, 'src/assets/styles')]  
                }
            }
        }
    }
}
```

以上是在该需求场景下给出的一个解决方案

可参考：

+ [vue-cli3-multiple-page](https://github.com/FredaFei/vue-cli3-multiple-page)