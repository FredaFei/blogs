#### 在处理表单数据时，React 有两种处理方式。一种是受控组件，另一种是非受控组件。

受控组件：表单数据由 React 组件负责处理
+ 每个表单元素的值(value)都有一个对应的state和事件监听
+ 触发事件时，变更对应的state

代码示例：

```
import React, { useState } from "react";

export default function App() {
  const [value, setValue] = useState("init");

  const onChange = e => {
    setValue(e.target.value);
  };
  return (
    <div className="App">
      <input value={value} onChange={onChange} />
    </div>
  );
}

```


非受控组件：其表单数据由 DOM 元素本身处理
+ DOM 中的 value，可使用ref获取表单值
+ 希望 React 有初始值，但保留后续更新不受控制时，使用 defaultValue 属性

代码示例：

```
import React, { useState, useRef } from "react";

export default function App() {
  const [initValue] = useState("initValue");

  const inputRef = useRef(null);
  const onBlur = () => {
    console.log(inputRef.current.value);
  };
  const onChange = e => {
    console.log(e.target.value);
  };
  return (
    <div className="App">
      <input defaultValue={initValue} onBlur={onBlur} ref={inputRef} />
      <input defaultValue="defaultValue" onChange={onChange} />
    </div>
  );
}


```