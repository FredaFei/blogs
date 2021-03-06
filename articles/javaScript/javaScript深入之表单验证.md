在一个项目时，表单不可忽略，最常见如登录注册。表单多起来后，验证也变得繁琐。常见的验证像手机、密码、邮箱、验证码等等，不想每次遇见表单都重写一遍验证。

##### 解决方案如下

+ 社区里有很多表单验证的项目，找一个star最多，持续维护中的；
    + 优点：便捷
    + 缺点：额外引入依赖，可能存在未知风险不能及时处理
+ 自己写，支持常见的验证规则，还可以自定义验证规则；
    + 优点：代码可控，随时迭代
    + 缺点：对开发者要求高，有一定的维护成本
    
#### API设计

参考优秀的表单验证库，取其精华

+ [async-validator](https://github.com/yiminghe/async-validator) Validate form asynchronous，Element UI 和 ant design 用的就是这款
+ [validatorjs/validate.js](https://github.com/validatorjs/validator.js) A library of string validators and sanitizers.
+ [rickharrison/validate.js](https://github.com/rickharrison/validate.js) A lightweight JavaScript form validation library


综上得出自定义封装的表单验证库格式如下：

```javascript

class Validator {
  // ...
  validate(data,rules){}
}

const v1 = new Validator()
const errors = v1.validate(data, rules)

```

```javascript
  // data 格式如下
  const data = { phone: '', password: '' }
  // rules 格式如下，其中 pattern 支持默认的验证规则（手机，邮箱等）和自定义验证规则
  const rules = [
    { key: 'phone', require: true, pattern: 'phone' },
    { key: 'password', require: true, pattern: /^(?![^a-zA-Z]+$)(?!\D+$).{8,16}$/ }
  ]
  // errors 格式如下
  { phone: ['不能为空', '格式不正确'], password: ['不能为空','密码格式不正确'] }
```

业务场景：有一个input输入框，现在需要对它做三步业务验证（同步验证），这三步的验证提示信息均不一样。
要满足一个这样的功能，通过*自定义验证器*可以办到

class Validator {
  // 自定义验证器
  static add(name, fn) {
     Validator.prototype[name] = fn
  }
  // ...
  validate(data,rules){}
}

```javascript
    const data = {
      orderId: '123'
    }
    let rules = [{ key: 'orderId', required: true, threeValidate: true }]
    function threeValidate(value){
      if (!/\d/.test(value)) {
        return '必须要有数字'
      }
      if (!/[xy]/.test(value)) {
        return '必须要有x或者y'
      }
      if (!/\_$/.test(value)) {
        return '必须以下划线结尾'
      }
    }
    // 自定义实例的验证器
    v1.threeValidate = threeValidate;
    const v1 = new Validator()
    const errors = v1.validate(data, rules)
    //全局添加自定义验证器
    threeValidate.add('threeValidate', threeValidate);
    const v2 = new Validator()
    const errors = v2.validate(data, rules)
```

以上便能满足基本的开发，完整代码如下：

```
class Validator {
  constructor() {}

  static add(name, fn) {
    Validator.prototype[name] = fn
  }

  validate(data, rules) {
    let errors = {}
    rules.forEach(ruleItem => {
      let value = data[ruleItem.key]
      if (ruleItem.required) {
        let errorText = this.required(value)
        if (errorText) {
          ensureArray(errors, ruleItem.key)
          errors[ruleItem.key].push(errorText)
          return
        }
      }
      let validators = Object.keys(ruleItem).filter(
        key => key !== 'key' && key !== 'required'
      )
      validators.forEach(validator => {
        if (validator) {
          let errorText = this[validator](value, ruleItem[validator])
          if (errorText) {
            ensureArray(errors, ruleItem.key)
            errors[ruleItem.key].push(errorText)
          }
        } else {
          throw `不存在的验证器${validator}`
        }
      })
    })
    return errors
  }

  isEmpty(errors) {
    return Object.keys(errors).length <= 0
  }

  required(value) {
    if (!value && value !== 0) {
      return '必填'
    }
  }

  pattern(value, pattern) {
    // 可根据需要自定义格式
    if (pattern === 'email') {
      pattern = /^.+@.+$/
    }
    if (pattern === 'phone') {
      pattern = /^1\d{10}$/
    }
    if (pattern === 'idCard') {
      pattern = /(^[1-9]\d{14}$)|(^[1-9]\d{16}([0-9]|X)$)/
    }
    if (pattern === 'idBank') {
      pattern = /^[1-9]\d{14,20}$/
    }
    if (pattern.test(value) === false) {
      return '格式不正确'
    }
    
  }

  minLength(value, minLength) {
    if (value.length < minLength) {
      return `长度不能小于${minLength}位`
    }
  }

  maxLength(value, maxLength) {
    if (value.length > maxLength) {
      return `长度不能大于${maxLength}位`
    }
  }
}

function ensureArray(obj, key) {
  if (Object.prototype.toString.call(obj[key]) !== '[object Array]') {
    obj[key] = []
  }
}


```

[code](https://github.com/FredaFei/amazing-ui/blob/master/src/validate.js)

+ 优点
  + 组合使用方便，不论项目是用Vue/React/jQ/原生js
  + 可做[单元测试](https://github.com/FredaFei/amazing-ui/blob/master/tests/unit/validate.spec.js)，迭代无惧；

不足之处，不支持异步校验。若要实现异步验证提供两种方案

+ 使用 callback，参考 [async-validator](https://github.com/yiminghe/async-validator)
+ 所有验证全部转为Promise
