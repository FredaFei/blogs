### JS异步错误捕获

**为什么要做错误处理？**

代码健壮性，对代码负责就是对自己负责（放假愉快玩耍）。

**哪些代码需要异步错误捕获?**

+ 需要处理异步错误时，ajax请求、promise等

**有哪些错误捕获方案?**

+ ```try{}catch(e) {}```

**一句话描述`try catch`**

**能被`try catch`捕捉到的异常，必须是线程执行已进入`try catch`代码块，且处在`try catch`里面，未执行完时抛出。**

+ 线程执行处于`try catch`之前

    ```JS
        try{
            a.
        }catch(e){
            console.log("error", e);
        }
        // 运行结果：Uncaught SyntaxError: Unexpected token '}'
    ```
  以上代码执行步骤分析如下： 代码在语法检查阶段，因语法异常，线程执行未进入`try catch`

+ 线程执行处于`try catch`之中

    ```JS
        function fn(){a+1}
        try{
            fn()
        }catch(e){
            console.log("error", e);
        }
        // 运行结果：error ReferenceError: a is not defined
    ```
  以上代码执行步骤分析如下： 线程执行已进入`try catch`，函数fn在执行时出现异常，异常catch成功

+ 线程执行处于`try catch`之后
    + 对setTimeout无效，如下例：
    ```js
        try {
            setTimeout(() => {
                throw new Error('async error')
            }, 1000)
        } catch(error) {
            console.log('catch error',error)
        }
        // 运行结果 : Uncaught Error: async error
    ```
    + 对宏任务的回调函数中的错误无效
        ```js
            // 异步任务
            const task = () => {
                setTimeout(() => {
                    throw new Error('async error')
                }, 1000)
            }
            // 主任务
            function main() {
                try {
                    task();
                } catch(error) {
                    console.log('catch error',error)
                }
            }
            main()
            // 运行结果 : Uncaught Error: async error
        ```
      以上代码执行步骤分析如下：
        1. 当前执行栈（main函数），setTimeout加入任务队列的宏任务。
        2. 当前执行栈全部执行完，main函数退出当前执行栈
        3. 取出任务队列的宏任务（setTimeout）入栈(js 主进程)执行
        4. 上下文环境变了，main函数的try/catch无法捕获到task函数的异常

    + 微任务（Promise）的回调
        ```js
            // 返回一个 promise 对象
            const fetch = () => 
                new Promise((reslove) => {
                    reslove();
                })
            }
            function main() {
                try {
                    // 回调函数里抛出错误
                    fetch().then(() => {
                        throw new Error('async error')
                    })
                } catch(e) {
                    console.log('catch error',error)
                }
            }
        
            main()
            // 运行结果 : Uncaught Error: async error
            
        ```
  以上代码执行步骤分析如下：
    1. 当前执行栈（main函数），new Promise立即执行，.then回调加入任务队列的微任务。
    2. 当前执行栈全部执行完，main函数退出当前执行栈
    3. 取出任务队列的微任务（then）入栈(js 主进程)执行
    4. 上下文环境变了，main函数的try/catch无法捕获到task函数的异常

**以上都是回调无法捕获，是否可以认为回调都无法try catch？**

不是，如下例子为证：

```js
    // 定义一个 fn，参数是函数。
    const fn = (fn) => fn()
    
    function main() {
        try {
            // 传入 callback，fn 执行会调用，并抛出错误。
            fn(() => {
                throw new Error('fn error');
            })
        } catch (error) {
            console.log('catch error', error)
        }
    }
    
    main();
    // 运行结果：catch error Error: fn error
```

**为什么以上例子中的函数fn的回调可以被try catch捕获？**
该例子中的回调为同步代码，执行栈的上下文未改变，所有可以被捕获。

**Promise的异常捕获**

+ **new Promise**

```js
    function main1() {
        try {
            new Promise((resolve, reject) => {
                throw new Error('promise1 error')
            })
        } catch (error) {
            console.log('catch error', error);
        }
    }
    
    function main2() {
        try {
            Promise.reject('promise2 error');
        } catch (error) {
            console.log('catch error', error);
        }
    }
    
    main1()
    // 运行结果：Uncaught (in promise) Error: promise1 error
    main2()
    // 运行结果：Uncaught (in promise) promise2 error
```

两次调用里，try catch 均未捕获到error。原来Promise 的异常都是由 reject 和 Promise.prototype.catch 来捕获，不管是同步还是异步。
核心原因是因为 Promise 在执行回调中都用 try catch 包裹起来了，其中所有的异常都被内部捕获到了，并未往上抛异常。代码如下：

```js
    function main1() {
        return new Promise((resolve, reject) => {
            throw new Error('promise1 error')
        })
    }
    
    function main2() {
        return Promise.reject('promise2 error')
    }
    
    main1().catch(error => {console.log(error)})
    // 运行结果：Error: promise1 error
    main2().catch(error => {console.log(error)})
    // 运行结果：promise2 error
```

+ **then**

then 之后的错误，需在回调内部catch，并return出去，传向下一个then。代码如下：

```js
    function main() {
        Promise.resolve(1).then(() => {
            try {
                throw new Error('throw then');
            } catch (e) {
                return e;
            }
        }).then(error => console.log(error));
    }
    
    main()
    // 运行结果： Error: throw then
```

**那异步错误怎么处理?**

+ 用Promise捕获异步错误

```js
    const asyncFn = () => new Promise((reslove, reject) => {
        setTimeout(() => {
            reject('async error');
        })
    });
    // 以下两行代码等价
    asyncFn().catch(error => console.log(error));
    asyncFn().then(null, error => console.log(error));
```

+ 先使用```async await```将异步代码转为同步，再```try{}catch(e) {}```

```js
    async function fn() {
        try {
            const res = await asyncFn()
        } catch (error) {
            //Todo ......
            console.log(error)
        }
    }
```

在实际项目中，以上两种方案里，第二种应用更方便。

但每个函数都要写一次try catch的话，代码重复且难看。可参考node语法的first-error优化下。代码如下：

```js
    const handleTryCatch = fn => async (...args) => {
        try {
            return [ null, await fn(...args) ]
        } catch (error) {
            console.log(error)
            return [ error ]
        }
    }
    const fetch = () => {
        return new Promise((resolve, reject) => {
            const random = parseInt(Math.random() * 10)
            random > 3 ? resolve(random) : reject('random <= 3')
        })
    }
    const main = async () => {
        const [ error, result ] = await handleTryCatch(fetch)()
        if (error) {
            // doSomething...
            return
        }
        // doSomething...
        console.log('result', result)
    }   
```

以上实现了try catch的抽离，但还需支持自定义错误处理。可将错误类型分类，代码如下

```js
    /**
     * @param errorHandle 错误处理函数
     * @param fn 异步处理
     * @return 新的异步函数
     **/
    const handleTryCatch = errorHandle => fn => async (...args) => {
            try {
                return [ null, await fn(...args) ]
            } catch (error) {
                return [ errorHandle(error) ]
            }
        }
    const fetch = () => {
        return new Promise((resolve, reject) => {
            reject(new ValidatedError('密码格式错误', 10012))
        })
    }
    
    // 数据库错误
    class DataBaseError extends Error {
        constructor(message, code) {
            super(message);
            this.errorMessage = message || 'db_error_msg';
            this.code = code || 20010;
        }
    }
    
    // 验证错误
    class ValidatedError extends Error {
        constructor(message, code) {
            super(message);
            this.errorMessage = message || 'validated_error_msg';
            this.code = code || 20010;
        }
    }
    
    // 统一错误
    const errorHandle = error => {
        if (error instanceof ValidatedError || error instanceof DataBaseError) {
            // doSomething
            return error;
        }
        return {
            code: 101,
            errorMessage: 'unKnown'
        };
    }
    
    const usualHandleTryCatch = handleTryCatch(errorHandle);
    
    const main = async () => {
        const [ error, result ] = await usualHandleTryCatch(fetch)()
        if (error) {
            // doSomething
            console.log('error:', error.code, error.errorMessage)
            return
        }
        // doSomething
        console.log('result', result)
    }
    main()
    // 运行结果：error: 10012 密码格式错误
```

以上代码适性广（函数和类均可使用），若项目风格均为类实现，可该为装饰器风格。代码如下：

```js
    const asyncErrorWrapper = (errorHandler) => (target) => {
        const props = Object.getOwnPropertyNames(target.prototype);
        props.forEach((prop) => {
            var value = target.prototype[prop];
            if (Object.prototype.toString.call(value) === '[object AsyncFunction]') {
                target.prototype[prop] = async (...args: any[]) => {
                    try {
                        return await value.apply(this, args);
                    } catch (err) {
                        return errorHandler(err);
                    }
                };
            }
        });
    };

    @asyncErrorWrapper(errorHandle) class Store {
        async getList() {
            return Promise.reject('类装饰：失败了');
        }
    }
    
    const store = new Store();
    
    async function main() {
        const o = await store.getList();
    }
    
    main();
```

参考文章：

+ [JS 异步错误捕获二三事](https://juejin.im/post/6844903830409183239#heading-9)