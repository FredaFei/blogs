#### 什么是 React Hooks

##### React Hooks 诞生背景
+ function component若是要拥有自己的状态，需要使用class component
+ 组件复杂度增加时，抽象和调试困难，会遇到多个状态嵌套，wrapper地狱
+ 业务逻辑复杂时，复用困难

#### React Hooks 解决的痛点
+ 更容易和适合将组件的UI与状态分离
+ 更函数式，更容易阅读和单测
+ Hooks 可以引用其他 Hooks

#### React Hooks 有哪些功能
##### useState
适用场景：作为组件自身的状态，和class Component 的 state作用一样

使用形式：

```jsx harmony
    const [a,setA] = React.useState(0)
```
例子：
```jsx harmony
    // exp:
    import React, { useState } from "react";

    export default function App() {
      const [a, setA]  = useState(0);
      const add = ()=>{
          setA(a+1)
      }
      return (
        <div>
            a: {a} <button onClick={add}>+1</button>
        </div>
      );
    }
```
useState 接受函数
使用形式：
```jsx harmony
    const [a,setA] = React.useState(()=>{return 0})
```
setA 接受函数
使用形式：
```jsx harmony
    setA(i=>i+1)
```
*注意：*

优先使用这种形式，两者间的差异如下

```jsx harmony
  const add = ()=>{
    setA(a+1)
    setA(a+1) // a 还是等于 1，a=1
    console.log('函数调用')
    setA(i=>i+1)
    setA(i=>i+1) // i等于上一次的基础上加上1后的 a，a=2
  }
```

##### useEffect
适用场景：
+ 作 componentDidMount使用，[]作第二参数
+ 作 componentDidUpdate，可指定依赖)
+ 作 componentDidWillUnmount，通过 return
+ 以上三种可同时存在

使用形式依次如下：

```jsx harmony
    React.useEffect(()=>{ // doSomething }, [])
    React.useEffect(()=>{ // doSomething }, [指定依赖])
    React.useEffect(()=>{ 
      return ()=>{}
    }, [])
    // 如下这种避免使用，表示无论是谁变化都会调用，会造成无限循环调用
    React.useEffect(()=>{ // doSomething })
```
例子：
```jsx harmony
    // exp:
    import React, { useState,useEffect } from "react";

    export default function App() {
      const [a, setA]  = useState(0);
      const add = ()=>{
          setA(a+1)
      }
      useEffect(() => {
        console.log("mounted");
      }, []);

      useEffect(() => {
        console.log(`a 变为: ${a}`);
      }, [a]);
    
      useEffect(() => {
        console.log("will unmounted");
        return () => {
          console.log("卸载");
        };
      }, []);

      return (
        <div>
            a: {a} <button onClick={add}>+1</button>
        </div>
      );
    }
```
*注意：*
多次调用useEffect，依次执行

##### useLayoutEffect
适用场景：
+ useLayoutEffect在浏览器渲染前执行
+ useEffect在浏览器渲染后执行
+ useLayoutEffect优先useEffect执行

使用形式依次如下（与useEffect类似）：

```jsx harmony
    React.useLayoutEffect(()=>{ // doSomething }, [])
    React.useLayoutEffect(()=>{ // doSomething }, [指定依赖])
```
例子：
```jsx harmony
    // exp:
    import React, { useState,useEffect,useLayoutEffect } from "react";

    export default function App() {
      const [a, setA]  = useState(0);
      const add = ()=>{
          setA(a+1)
      }
      useLayoutEffect(() => {
        console.log("useLayoutEffect 1");
      }, []);
      useLayoutEffect(() => {
        console.log("useLayoutEffect watch a");
      }, [a]);
      useEffect(() => {
        console.log("mounted");
      }, []);

      useEffect(() => {
        console.log(`a 变为: ${a}`);
      }, [a]);

      return (
        <div>
          <div id="test">a 变为：{a}</div>
          <button onClick={add}>+1</button>
        </div>
      );
    }
```
*注意：*
+ useLayoutEffect影响UI响应时长
+ 优先使用useEffect

例子：
```jsx harmony
    // exp:
    import React, { useState,useEffect,useLayoutEffect } from "react";

    export default function App() {
      const [a, setA]  = useState(0);
      const add = ()=>{
          setA(a+1)
      }
      // id=test节点内容出现闪烁，从'a 变为：1' ==> 'a 变为：100'
      useEffect(() => {
        console.log("useEffect watch a"); 
        document.getElementById("test").innerText = "a 变为：100";
      }, [a]);

      return (
        <div>
          <div id="test">a 变为：{a}</div>
          <button onClick={add}>+1</button>
        </div>
      );
    }
    export default function App() {
      const [a, setA]  = useState(0);
      const add = ()=>{
          setA(a+1)
      }
      // id=test节点内容没有出现闪烁，内容直接为 'a 变为：100'
      useLayoutEffect(() => {
        console.log("useLayoutEffect watch a");
        document.getElementById("test").innerText = "a 变为：100";
      }, [a]);

      return (
        <div>
          <div id="test">a 变为：{a}</div>
          <button onClick={add}>+1</button>
        </div>
      );
    }
```

##### useRef
适用场景：需要一个值，在组件不断render时保持不变

使用形式：

```jsx harmony
    const count = React.useRef(0)
    // 读取
    count.current //引用取值
```
例子
```jsx harmony
    // exp:
    import React, { useState,useRef,useEffect } from "react";

    export default function App() {
      const [a, setA] = useState(0);
      let count = useRef(a);
    
      const add = () => {
        setA(a + 1);
      };
      return (
        <div>
          <div>a 的初始值: {count.current}</div>
          <div>a 变为：{a}</div>
          <button onClick={add}>+1</button>
        </div>
      );
    }
```
*注意：*

+ useRef变化时，不会自动render
+ 若要实现这个功能，可自行添加
+ 监听ref.current的变化，setXx触发
+ useRef可用于DOM对象，也可用于普通对象

##### forwardRef
适用场景：实现ref的传递（props不包含ref）

使用形式：

```jsx harmony
    const newComponent = React.forwardRef((props,ref)=><ComponentName ref={ref} {...props}/>)
```
例子
```jsx harmony
    // exp:
    import React, { useRef, useEffect } from "react";

    const Child = React.forwardRef((props, ref) => (
      <div ref={ref} {...props}>
        我是 Child
      </div>
    ));
    export default function App() {
      const childRef = useRef(null);
      useEffect(() => {
        console.log("childRef.current");
        console.log(childRef.current);
      }, []);
      return (
        <div>
          <Child ref={childRef} />
        </div>
      );
    }
```

##### useReducer
适用场景：用来践行Redux思想

使用形式：
+ 创建初始initialState
+ 创建所有操作 reducer(state,action)
+ 传给useReducer，[state, dispatch] = useReducer(reducer, initialState);
+ 调用dispatch({type: '操作类型'})

例子：
```jsx harmony
    import React, { useReducer } from "react";
    import "./styles.css";
    
    const initCount = { count: 0 };
    function reducer(state, action) {
      switch (action.type) {
        case "add":
          return { ...state, count: state.count + action.n };
        case "decrease":
          return { ...state, count: state.count + action.n };
        case "double":
          return { ...state, count: state.count * 2 };
        case "square":
          return { ...state, count: state.count * state.count };
        default:
          throw new Error("unknown type");
      }
    }
    export default function App() {
      const [state, dispatch] = useReducer(reducer, initCount);
      const onAdd = () => {
        dispatch({ type: "add", n: 1 });
      };
      const onDecrease = () => {
        dispatch({ type: "decrease", n: -1 });
      };
      const onDouble = () => {
        dispatch({ type: "double", count: 2 });
      };
      const onSquare = () => {
        dispatch({ type: "square" });
      };
      return (
        <div>
          <div>Count: {state.count}</div>
    
          <button onClick={onAdd}>count+1</button>
          <button onClick={onDecrease}>count-1</button>
          <button onClick={onDouble}>count*2</button>
          <button onClick={onSquare}>count*count</button>
        </div>
      );
    }

```

##### useContext
适用场景：全局变量（全局上下文）

使用形式：
+ 创建全局上下文，Context = createContext(initial)
+ 使用<Context.Provider> 圈定作用域
+ 在作用域内使用useContext(Context)共享全局上下文

例子：
```jsx harmony
    import React, { useContext, createContext, useState } from "react";
    
    const Context = createContext(null);
    export default function App() {
      const [x, setX] = useState(0);
      return (
        <Context.Provider value={{ x, setX }}>
          <Parent />
        </Context.Provider>
      );
    }
    
    function Child() {
      const { x, setX } = useContext(Context);
      const onClick = () => {
        setX(x + 1);
      };
      return (
        <div>
          child x is: {x} <button onClick={onClick}>child x+1</button>
        </div>
      );
    }
    
    function Parent() {
      const { x, setX } = useContext(Context);
      const onClick = () => {
        setX(x - 1);
      };
      return (
        <div>
          parent x is: {x} <button onClick={onClick}>parent x-1</button>
          <Child />
        </div>
      );
    }
```

*注意：*
+ useContext不是响应式的
+ 数据自顶向下传递
+ 在一个模块内将Context改变，另一个模块不会感知

###### 如何代替Redux
实现步骤
1. 将所有数据放在store对象
2. 将所有操作放在reducer中
3. 将建一个 Context
4. 创建一个数据的读写API
5. 将第四步的内容放到第三步的Context
6. 使用<Context.Provider>将Context提供给所有组件
7. 各个组件使用useContext(Context)获取读写API

[code](https://codesandbox.io/s/react-hooks-usecontext-o2mjz?file=/src/index.js)

##### useMemo
例子：

```jsx harmony
    import React, { useState, useEffect } from "react";

    function Child(props) {
      console.log("Child render");
      console.log("大量数据");
      return <div>我是 Child y: {props.y}</div>;
    }
    
    export default function App() {
      const [x, setX] = useState(0);
      const [y, setY] = useState(0);
      useEffect(() => {
        console.log("App render");
      }, [x]);
    
      const addX = () => {
        setX(x + 1);
      };
      const addY = () => {
        setY(y + 1);
      };
      return (
        <div>
          <div>我是 Parent x: {x}</div>
          <Child y={y} />
          <button onClick={addX}>x+1</button>
          <button onClick={addY}>y+1</button>
        </div>
      );
    }
```
结果：当点击x+1时，x改变，App组件更新渲染，但是Child组件也跟着重新渲染了，而此时y并未改变。

需求：若子组件中的 props 不变，其子组件不必更新。实现该需求有如下方案：

*方案一 React.memo*
使用场景
+ React 默认有多余的render
+ 若子组件中的 props 不变，可不必再次执行一个函数组件

例子：
```jsx harmony
    import React, { useState, useEffect } from "react";

    function Child(props) {
      console.log("Child render");
      console.log("大量数据");
      return <div>我是 Child y: {props.y}</div>;
    }
    // 修改部分
    const ChildMemo = React.memo(Child);
    export default function App() {
      const [x, setX] = useState(0);
      const [y, setY] = useState(0);
      useEffect(() => {
        console.log("App render");
      }, [x]);
    
      const addX = () => {
        setX(x + 1);
      };
      const addY = () => {
        setY(y + 1);
      };
      return (
        <div>
          <div>我是 Parent x: {x}</div>
          // 修改部分
          <ChildMemo y={y} />
          <button onClick={addX}>x+1</button>
          <button onClick={addY}>y+1</button>
        </div>
      );
    }
```

但是，当给Child添加一个监听函数后，又不满足需求了。代码如下

```jsx harmony
    import React, { useState, useEffect } from "react";

    function Child(props) {
      console.log("Child render");
      console.log("大量数据");
      // 修改部分
      return <div onClick={props.onChildClick}>我是 Child y: {props.y}</div>;
    }
    const ChildMemo = React.memo(Child);
    export default function App() {
      const [x, setX] = useState(0);
      const [y, setY] = useState(0);
      useEffect(() => {
        console.log("App render");
      }, [x]);
    
      const addX = () => {
        setX(x + 1);
      };
      const addY = () => {
        setY(y + 1);
      };
      const onClick = () => {};
      return (
        <div>
          <div>我是 Parent x: {x}</div>
          // 修改部分
          <ChildMemo y={y} onChildClick={onClick}/>
          <button onClick={addX}>x+1</button>
          <button onClick={addY}>y+1</button>
        </div>
      );
    }
```

原因：当点击x+1时，x改变，触发App更新，生成新的onClick函数

*方案二 useMemo*
适用场景：
+ 当依赖变化时，重新计算出新的value
+ 当依赖不变时，复用之前的value

使用形式依次如下：

```jsx harmony
    React.useMemo(() => Y, [指定依赖])
```
例子：
```jsx harmony
    import React, { useState, useEffect } from "react";

    function Child(props) {
      console.log("Child render");
      console.log("大量数据");
      return <div onClick={props.onChildClick}>我是 Child y: {props.y}</div>;
    }
    const ChildMemo = React.memo(Child);
    export default function App() {
      const [x, setX] = useState(0);
      const [y, setY] = useState(0);
      useEffect(() => {
        console.log("App render");
      }, [x]);
    
      const addX = () => {
        setX(x + 1);
      };
      const addY = () => {
        setY(y + 1);
      };
      // 修改部分
      const onClick = useMemo(() => {
          return () => {
            console.log(y);
          };
        }, [y]);
      return (
        <div>
          <div>我是 Parent x: {x}</div>
          <ChildMemo y={y} onChildClick={onClick}/>
          <button onClick={addX}>x+1</button>
          <button onClick={addY}>y+1</button>
        </div>
      );
    }
```
*注意：*
+ 若Y是一个函数，则useMemo的值为一个返回函数的函数，改为 React.useMemo(() => x=>{console.log(x)}, [指定依赖])
+ 可用 useCallback优化
+ useCallback(x=>{console.log(x)},[指定依赖]) = useMemo(() => x=>{console.log(x)}, [指定依赖])

##### 自定义 Hook

[code](https://codesandbox.io/s/custom-hooks-urghy)

#### 什么时候使用 React Hooks
+ 更喜欢写Function Component
+ 更细致的抽取公共逻辑，不想使用HOC等