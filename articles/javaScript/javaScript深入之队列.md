思路：
+ 使用队列的特点，先进先出，支持插队 
+ 当前队列执行完后，再call下一个执行

具体代码如下

```js
    class _LazyMan {
      constructor(message) {
        this.queue = [];
        this.sayHi(message);
        // 保证在当前执行栈的最后执行
        setTimeout(() => {this.next();}, 0);
      }
    
      next() {
        const fn = this.queue.shift();
        if (fn && typeof fn === 'function') {fn();}
      };
    
      sayHi(message) {
        this.queue.push(() => {
          console.log(`Hi,${ message }`);
          this.next();
        });
        return this;
      };
    
      eat(message) {
        this.queue.push(() => {
          console.log(message);
          this.next();
        });
        return this;
      };
    
      sleep(time) {
        this.queue.push(() => {
          setTimeout(() => {
            console.log(`我已经睡了${ time }s`);
            this.next();
          }, time * 1000);
        });
        return this;
      };
    
      sleepFirst(time) {
        this.queue.unshift(() => {
          setTimeout(() => {
            console.log(`我先睡${ time }s`);
            this.next();
          }, time * 1000);
        });
        return this;
      };
    }
    
    function LazyMan(name) {
      return new _LazyMan(name);
    }
    
    LazyMan('ha');
    LazyMan('ha').sleep(2).eat('dinner');
    LazyMan('ha').eat('dinner').sleepFirst(2).eat('supper');

```