### call、apply、bind作用和差异？

作用：均是`Function.prototype`上的方法，可用来改变`this`指向;

差异：使用方式不一样

call第一参数是`this`指向，其他参数是传递的值。
```jsx harmony
  fn.call(obj,1,2)
```

注意：
第一参数为null、undefined、不传，this 将会指向全局对象（非严格模式下）

apply同call类似，但第二参数为数组格式。

```jsx harmony
  fn.apply(obj,[1,2])
```

bind 被调用时返回一个新的函数，其中第一个参数为新函数的this，其余参数将作为新函数的参数。

```jsx harmony
const obj = {name:'out'}
function fn(){
  setTimeout(function() {
    // inner
    console.log(this.name)
  }.bind({name:'inner'}),1000)
}
fn()
```
#### 手写call
```jsx harmony
Function.prototype.call2 = function(context, ...args) {
  context = context || window
  const fn = Symbol('fn')
  context[fn] = this
  const result = context[fn](...args);
  delete context[fn]
  return result
}
```
#### 手写apply
```jsx harmony
Function.prototype.apply2 = function(context, args) {
  context = context || window
  const fn = Symbol('fn')
  context[fn] = this
  const result = context[fn](args);
  delete context[fn]
  return result
}
```

```jsx harmony
function fn(){
  console.log(this.name)
}
fn.call2({name: 'call'})
fn.apply2({name: 'apply'})
```

#### 手写bind
在写之前需要知道以下几点
+ new 操作符做了哪些事
+ this 指向
+ 原型和原型链

new主要做了以下几件事
+ 创建一个新对象
+ 绑定原型（prototype）
+ 修改this指向
+ 返回这个新对象

```jsx harmony
function new2(Constructor,...args){
  const temp = {}
  temp.__proto__ = Constructor.prototype
  Constructor.call(temp, ...args)
  return temp
}

function Cat(name) {
  this.name = name
}
Cat.prototype.say = function() {
  console.log(this.name)
}
const c = new2(Cat, 'miao')
console.log(c.say())
```

```jsx harmony
Function.prototype.bind2 = function(context,...args) {
  const fn = this
  function F(...otherArgs){
    // new 调用时，this应为构造函数实例
    return fn.call(new.target ? this : context, ...args, ...otherArgs)
  }
  // new 调用时，绑定该构造函数的属性和方法
  extend(fn, F)
  // 更新fn函数的name和length属性
  Object.defineProperties(F,{
    name: {value: fn.name},
    length: {value: fn.length}
  })
  return F
}
function extend(Parent,Child) {
  function Fn(){}
  Fn.prototype = Parent.prototype
  Child.prototype = new Fn()
  Child.prototype.constructor = Child
}

function Student(name, age) {
    this.name = name;
    this.age = age;
}
Student.prototype.exam = 'math'
const S = Student.bind2({ name: 'ha' }, 'test')
const s1 = new S(22)
console.log(s1.exam)

```