接着[react篇之手写redux(一)](https://github.com/FredaFei/blogs/blob/master/articles/react/react%E7%AF%87%E4%B9%8B%E6%89%8B%E5%86%99redux(%E4%B8%80).md)
文章最后提出的待完善问题，来个个击破吧。

#### 问题一： 修改全局`state`时，每个组件均会多次render

**Bug复现**:

+ 在每个子组件中添加一个`log`
+ 初始渲染时，每个组件都会打印一遍
+ 触发B组件修改`state`，此时其他组件（无论有没有用到全局`state`），均重新渲染了一次

**原因：** App组件中的state作了全局的`state`。

#### 如何精准渲染，组件只在自己的数据变化时`render`

**解决方案**： 将appState独立出去。

调整核心代码如下：

``` jsx
// store.js
export const store = {
  state: {
    user: { name: "test", age: 10 }
  },
  setState(newState) {
    store.state = newState;
  }
};
// App.js
const App = (props) => {
  return (
    <appContext.Provider value={store}>
      <section>
        <div>app</div>
        <A />
        <B />
        <C />
      </section>
    </appContext.Provider>
  );
};
// connect.js
export function connect(component) {
  return (props) => {
    const { state, setState } = store;
    function dispatch(action) {
      setState(reducer(state, action));
    }

    return React.createElement(component, { state, dispatch }, props.children);
  };
}
```

[完整代码-store](https://codesandbox.io/s/shouxiereduxpianzhier-youhuaappcontext-ulqgs)

初始渲染正常，修改`user.name`字段时，组件并不更新。因为能触发组件更新的Api是`setState`，而使用App组件的`setState`又会有重复render的问题。

**解决思路**：

1. 使用 `Wrapper` 的 `setState`
2. `Wrapper` 订阅`store.state`的变化

核心代码如截图所示：

![store-1](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/store-1.png)

![connect-1](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/connect-1.png)

[完整代码-subscribe](https://codesandbox.io/s/shouxiereduxpianzhier-youhuaappcontext-ulqgs)

经测试，以上方案做到了修改全局`user.name`时，被`connect`后的组件，能更新成功。
但是，被`connect`后的组件，没有使用`user.name`，也会被重新执行一次。因此并未做到完全的精准渲染。

**优化方案**:

+ 每个 `Wrapper` 对比一下自己依赖的值是否改变
+ 若没变，则不要执行 `update({})` 

#### 那每个 `Wrapper` 怎么知道自己依赖的值是 `state.user` 还是 `state.score` 呢？

**解决思路**：

+ 可以让 `connect` 接受一个 `selector`，
+ 如果依赖 `user`，就给 `connect` 传一个 `state => ({user: state.user})`；
+ 如果依赖 `score`，就传 `state => ({group: state.group})`

以上`connect`的第一个参数 `selector`其实对应的是`redux`的`connect`的第一参数`mapStateToProps`

[完整代码-mapStateToProps](https://codesandbox.io/s/shouxiereduxpianzhier-mapstatetoprops-mw5qy)

**订阅需要取消订阅**

核心代码如截图所示：

![unsubcribe](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/unsubcribe.png)
![unsubcribe_apply](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/unsubcribe_apply.png)

#### mapDispatchToProps(修改state)

核心代码如截图所示：

![mapDispatchToProps](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/mapDispatchToProps.png)
![mapDispatchToProps_apply](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/mapDispatchToProps_apply.png)

[完整代码-mapDispatchToProps](https://codesandbox.io/s/shouxiereduxpianzhier-mapdispatchtoprops-2ohns)

#### connect(mapStateToProps, mapDispatchToProps)(Component)

将以上代码按如上设计优化，核心代码如截图所示：

![connecters](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/connecters.png)
![connecters_apply](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/connecters_apply.png)

[完整代码-connecters](https://codesandbox.io/s/shouxiereduxpianzhier-connectmapstatetoprops-mapdispatchtopropscomponent-67bcn)

#### createStore
redux风格
```js
    createStore(reducer,initState)
```

核心代码如截图所示：

![createStore](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/createStore.png)
![createStore_apply](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/createStore_apply.png)

[createStore](https://codesandbox.io/s/shouxiereduxpianzhier-createstore-ni1pj)

#### 封装Provider
核心代码如截图所示：

![provider](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/provider.png)
![provider_apply](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-2/provider_apply.png)

[完整代码-provider](https://codesandbox.io/s/shouxiereduxpianzhier-provider-p2oqe)

**待完善之处：**

+ Redux Middleware

有兴趣的可看下一篇[react篇之手写redux(三)](https://github.com/FredaFei/blogs/blob/master/articles/react/react%E7%AF%87%E4%B9%8B%E6%89%8B%E5%86%99redux(%E4%BA%8C).md)






