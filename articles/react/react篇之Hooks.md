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

##### useReducer
##### useLayoutEffect
##### useMemo

#### 什么时候使用 React Hooks