通过[react篇之手写redux(一)]()，已实现了redux中的`connect`、`dispatch`、`reducer`功能。在查看组件的更新流程中，发现组件会多次render。

Bug复现:
+ 在每个子组件中添加一个`log`
+ 初始渲染时，每个组件都会打印一遍
+ 触发B组件修改state，此时其他组件（无论有没有用到全局state），均重新渲染了一次

原因： App组件中的state作了全局的state。

确定需求：只更新需要更新的组件
解决方案： 将appState独立出去。

调整代码如下：
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

[代码](https://codesandbox.io/s/shouxiereduxpianzhier-youhuaappcontext-ulqgs?file=/src/App.js)

初始渲染正常，修改user.name字段时，组件并不更新。因为能触发组件更新的Api是`setState`，而使用App组件的`setState`又会有重复render的问题。

解决方案：
+ 使用 `Wrapper` 的 `setState`
+ `Wrapper` 订阅`store.state`的变化

核心代码如截图所示：

![](/redux-2/store-1.png)

![](/redux-2/connect-1.png)

[完整代码](https://codesandbox.io/s/shouxiereduxpianzhier-youhuaappcontext-ulqgs?file=/src/App.js)

以上方案解决了没有被`connect`，没有使用全局store中的state的组件重复render的问题。

但是，被`connect`后的组件，无论是否使用user.name，均会被重新执行一次。

优化方案:

+ 每个 `Wrapper` 对比一下自己依赖的值是否改变了
+ 没变就不要执行 `update({})` 

每个 `Wrapper` 怎么知道自己依赖的值是 `state.user` 还是 `state.score` 呢？
让 `connect` 接受一个 `selector` 即可，如果依赖 `user`，就给 `connect` 传一个 `state => ({user: state.user})`；如果依赖 `score`，就传 `state => ({group: state.group})`

原来 redux 的 `connect` 的第一个参数 `mapStateToProps`是干这个的啊
