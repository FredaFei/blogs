接着[react篇之手写redux(一)]()
文章最后提出的待完善问题，来个个击破吧。

#### Redux 中的Middleware是什么？
>Middleware is the suggested way to extend Redux with custom functionality. Middleware lets you wrap the store's dispatch method for fun and profit. The key feature of middleware is that it is composable. Multiple middleware can be combined together, where each middleware requires no knowledge of what comes before or after it in the chain.

以上为官方描述，总结出Redux 的 `Middleware`（中间件）特点：

+ 自定义，扩展Redux功能。
    + 代表异步操作的中间件有
        + `redux-thunk`
        + `redux-promise`
+ 可组合，多个中间件可组合，互不影响。

**常见需求场景**
+ 获取用户数据
+ 再将其更新到全局`state`，伪代码`dispatch({type:'update', payload:...})`

**解决方案：**

+ 函数Action（同步）
+ Promise Action（异步）

下面分别阐述这两种方案的实现

#### 函数Action（同步）

功能流程，伪代码：
``` jsx
const fetchUser = (dispatch)=>{
    fetch('/user').then(response=>{
        dispatch({type:'getUser', payload:response.data})
    }
}
const getUser = ()=>{
    fetchUser(dispatch)
})
<button onClick={getUser}>获取用户信息</button>
``` 

参考`redux`中的函数`action`的API设计是`dispatch(fetchUser)`。

这种风格更易于阅读、理解。那就把上面的修改成与官方`redux`一致，其转换核心如下：

```js
    // 缓存真实的dispatch 
    const prevDispatch = dispatch;
    // 重写dispatch
    const dispatch = (fn) => {
        // 使用缓存的dispatch
        fn(prevDispatch);
    };
```
核心代码如截图所示：

![actionFunction](https://leanote.com/api/file/getImage?fileId=606d81f3ab64417265000383)

![actionFunction_apply_connect](https://leanote.com/api/file/getImage?fileId=606d821bab64417465000310)

![actionFunction_apply_component](https://leanote.com/api/file/getImage?fileId=606d8263ab64417465000313)

[完整代码-action](https://codesandbox.io/s/shouxiereduxpianzhisan-function-action-xz232)

#### 支持Promise Action（异步）

核心代码如截图所示：

![asyncAction](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/asyncAction.png)
![asyncAction_apply_component](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/asyncAction_apply_component.png)

[完整代码-async action](https://codesandbox.io/s/shouxiereduxpianzhisan-async-action-klvb2)

redux-thunk源码的核心代码如截图所示：

![redux-thunk-源码](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/redux-thunk.png)

[redux-thunk-源码](https://github.com/reduxjs/redux-thunk/blob/master/src/index.js)

以上是便是手写redux系列的中间件实现。至此整个redux手写系列已完成啦~~