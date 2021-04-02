## react 父子组件的状态管理
经典的redux
从react原生提供的状态管理说起，`Context.Provider`，redux正是基于这一点而封装的。下面用一个例子来窥探下redux的实现思路。

如图所示，有一个App组件，它下面有三个子组件，分别是A、B、C。

代码如下：

```jsx
// A.jsx
const A = (props) => {
    return <section>A</section>
};
// B.jsx
const A = (props) => {
    return <section>B</section>
};
// C.jsx
const A = (props) => {
    return <section>C</section>
};
// App.jsx
const App = (props) => {
    return <section>
        <div>root</div>
        <A/>
        <B/>
        <C/>
    </section>
};
```



需求：
+ App组件需要把它的state共享给它的子组件使用；
+ 子组件可以修改共享的state；
+ 共享的state变更，则所有使用state的组件会更新。


使用react [Context.Provider实现](https://codesandbox.io/s/react-reduxyuanlishixianjichupian-x0cmp?file=/src/App.js)

![redux-1](/images/redux-1/redux-1.png)
如图所示，子组件对state的修改思路是：
+ 创建一个新的state；
+ 使用新的state替换旧的state；

对第一步的优化：创建新state的实现单独抽离出来，代码如截图所示：
![reducer](/images/redux-1/reducer.png)

第二步的实现，主要代码在27~33行。假设再对其他字段修改也是一样的处理。即与第27~33行代码模式一样，那是否可以封装下？

封装代码如截图所示：
![updateState](/images/redux-1/updateState.png)

以上优化后的[完整代码](https://codesandbox.io/s/shouxiereduxpianzhiyi-r631j?file=/src/App.js:660-668)

但以上代码中的updateState函数有个问题，它不能脱离组件，因参数`appState`、`setAppState`只有在组件中的`Context`里才可以获取到。有什么办法可以解决呢？

解决方案：
+ 不要把`appState`、`setAppState`放在`Context`里；
+ 将updateState放在组件中，组件可以获取到`Context`；

下面以第二种方案展开实现，核心代码如截图所示：

![createWrapper](/images/redux-1/createWrapper.png)

![createWrapper_apply](/images/redux-1/createWrapper_apply.png)

[代码如下](https://codesandbox.io/s/shouxiereduxpianzhiyi-gaojiezujian-xmen0?file=/src/App.js)

结合`redux`的API:

+ createNewState，规范state创建 => reducer
+ createWrapper， 高阶组件 => connect
+ updateState，指定动作细节 => dispatch

整理以上链接中的代码命名后，[最终代码](https://codesandbox.io/s/shouxiereduxpianzhiyi-forked-xmen0?file=/src/App.js)

至此初步实现了redux的`reducer`、`connect`、`dispatch`三个API的基础功能。

待完善之处：
+ 修改全局`state`时，每个组件均会多次render
+ 丰富`Connect`的功能
    + mapStateToProps
    + mapDispatchToProps
    
+ 封装Provider、createStore

有兴趣的可看下一篇[react篇之手写redux(二)]()

