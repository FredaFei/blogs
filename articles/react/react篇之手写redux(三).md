### 支持函数Action
需求：先获取用户数据，再将其更新到全局`state`

功能流程，伪代码：
``` jsx
const fetchUser = (dispatch)=>{
    fetch('/user').then(response=>{
        dispatch({type:'getUser',payload:response.data})
    }
}
const getUser = ()=>{
    fetchUser(dispatch)
})
<button onClick={getUser}>获取用户信息</button>
``` 

参考`redux`中的函数`action`的API设计是`dispatch(fetchUser)`，这种风格更易于阅读、理解。
那就把上面的修改成与官方`redux`一致，其转换核心如下：

```js
    const prevDispatch = dispatch;
    const dispatch = (fn) => {
        fn(prevDispatch);
    };
```
核心代码如截图所示：

![actionFunction](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/actionFunction.png)
![actionFunction_apply_connect](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/actionFunction_apply_connect.png)
![actionFunction_apply_component](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/actionFunction_apply_component.png)

[action](https://codesandbox.io/s/shouxiereduxpianzhisan-function-action-xz232)

### 支持Promise Action

同样的需求，使用Promise Action的方式实现。

核心代码如截图所示：

![asyncAction](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/asyncAction.png)
![asyncAction_apply_component](https://github.com/FredaFei/blogs/blob/master/articles/react/images/redux-3/asyncAction_apply_component.png)

[async action](https://codesandbox.io/s/shouxiereduxpianzhisan-async-action-klvb2)

