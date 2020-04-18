# 图片引用的问题和思考

在最近的一个vue项目中，项目结构如下：

目录结构：

    |──... 
    |── static                            // 静态文件
    |   |── images                 
    |   |   |...                       
    |── src
    |   |── assets
    |   |   |── images                
    |   |   |── |...           
    |   |── components
    |   |── views                         // 主要视图模块
    |   |   |── view_1
    |   |   |   |── components            // 模块内组件
    |   |   |   |── index.vue                 
    |   |   |── view_2
    |   |   |   |── components            // 模块内组件
    |   |   |   |── index.vue                 
    |   |   |── ...                       // 其他视图模块                
    |   | ...   
    |──...
    
图片一部分放在`static`目录下，一部分放在`src/assets`目录下。原项目中图片的引用如下：

    // html中引用
    <img src="~@/assets/images/1.jpg" />
    // css中引用
    background: url('./static/1.png') no-repeat center/100%;
    
打包（打包配置默认vue-cli配置）后的文件目录如下：

    |── index.html
    |── static                            // 静态文件
    |   |── images                 
    |   |   |...    
    |   |── css
    |   |   |...               
    |   |── js         
    |   |── |...
    |   |── 图片 （原static下的图片）
    
 
 **疑惑一**
 
这与我们想象中的不一样，图片不应该是都放在images目录下吗？怎么跑到static目录下了。

**解惑一**

>  原来`static/`目录下的文件并不会被`Webpack`处理：它们会直接被复制到最终的打包目录（默认是`dist/static`）下。必须使用绝对路径引用这些文件，这是通过在`config.js`文件中的`build.assetsPublicPath` 和 `build.assetsSubDirectory`连接来确定的。
而`assets`目录中的文件会被`webpack`处理解析为模块依赖，只支持相对路径形式

有了以上了解后，我想实现这样一个需求，想把项目中的图片集中管理，通过webpack处理下来提高性能。所以图片要放在`assets`目录下。
    
图片经处理后，引用assets图片如下
    
    // html中引用
    <img src="~@/assets/images/1.jpg" />
    // css中引用
    background: url('~@/assets/images/1.png') no-repeat center/100%;
    
若想通过js来引入图片，如下

    data(){
      return {
        imgName: require(`@/assets/images/bg/1.png`)
      }
    }

这样就可以愉快的在你想用的地方使用图片啦~


参考阅读：
[vue-cli 自定义路径别名 assets和static文件夹的区别 ](https://juejin.im/post/59be4d325188257e764c8485)
[vue-loader 生成的css文件中background url()图片路径问题](https://github.com/vuejs/vue-loader/issues/481#issuecomment-306202382)

# 自定义事件传参问题
    
自定义事件要传递多参数时，可参考如下方式    
    
    // child.vue
    <button @click="clickFn">click me</button>
    // js
    clickFn(){
        this.$emit('button:click', 11,22)
    }
    //parent.vue
    <div class="cell">
        <input />
        <i-button @button:click="parentFn(...arguments,33}"></i-button>
    </div>
    // js
    import IButton from 'child.vue'
    ...
    parentFn(beforeNum, ...rest){
        console.log{beforeNum, ...rest}
    }
    parentFn(){
        console.log(...arguments) 
    }
    ...


参考阅读：

[自定义事件传参问题](https://github.com/vuejs/vue/issues/5735)
