### promisify是什么？
把带有callback的函数，变成重新使用promise来实现的一种技术方案。

#### 为什么要使用promisify？
promise的到来，解决了回调地狱，让代码可以链式调用。代码如下：

```js
    // 原始回调代码
    a(()=>{
        return b(()=>{
            return c(()=>{
                return d()
            })
        })
    })
    // promise 链式代码
    a().then(b)
        .then(c)
        .then(d)
        .catch(function(err){
            console.log(err);
        });

```
然而，在项目中并非所有的回调、异步操作都实现了promise，所以创造了promisify方案。让使用者既享用了promise，又不需要对原始的回调、异步操作的实现做额外的处理。

#### 原始使用方式
```js
    const fs = require('fs');
    
    fs.readFile('./test.js', function(error,data){
        if(error){return error}
        console.log(data);
        // doSomething...
    });
```

#### promisify的使用方式
```js
    // promisify 形式
    const fs = require('fs');
    const Promise = require('bluebird');
    const readFileAsync = Promise.promisify(fs.readFile);
    
    readFileAsync('./test.js').then(function(data){
        console.log(data);
        // doSomething...
    }).catch(error=>console.log(error));

    // await 风格
    try{
        const data = await readFileAsync('./test.js')
        // doSomething...
    }catch (error) {
      console.log(error)
    }
```

#### 满足什么条件可以使用promisify？

+ 该方法须含回调函数
+ 回调函数须执行
+ 回调函数参数形式为first-error风格。即第一参数表示error，第二参数表示成功返回的结果

API设计上和node风格一致

#### promisify实现原理
```
    function promisify(original) {
        return function (...args) {
            return new Promise((resolve, reject) => {
                args.push(function callback(err, ...values) {
                    if (err) {return reject(err);}
                    return resolve(...values)
                });
                // args: arg1,arg2,...,(error,data)=>void
                original.call(this, ...args);
            });
        };
    }
    function fn(a,b,cb){
        setTimeout(()=>{
            let error = ''
            if(typeof b !== 'nubmer'){error = 'b is not number'}
            const data = a+b
            cb(error, data)
        },2000)
    }
    var fn2 = promisify(fn)
    fn2(4,'2').then(value=>{
        console.log(value)
    },error=>{
        console.log(error)
    })
    //运行结果: b is not number
```

#### promisifyAll实现原理

```js
    function promisifyAll(obj) {
        Object.keys(obj).forEach(key => { // es5将对象转化成数组的方法
            if (typeof obj[key] === 'function') {
                obj[key + 'Async'] = promisify(obj[key])
            }
        })
    }
    // 使用例子
    const fs = require('fs');
    promisifyAll(fs); 
    fs.readFileAsync('1.txt', 'utf8').then(function (data) {
        console.log(data);
    });

```

参考阅读

+ [bluebird](https://github.com/petkaantonov/bluebird)
+ [nodejs promisify 实现](https://set.sh/post/200820-how-promisify-works)