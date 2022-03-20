# 现代JavaScript教程学习笔记
这是我学习现代JavaScript教程的笔记，他们的内容确实好，有一种相见恨晚的感觉，当时要是初学js能遇到就好了。。。 没有抄袭，只是笔记，并尝试自己稍微讲一下，如果涉及侵权，会立刻删除
## try catch
```js
try {

} catch(err) {
  
}
```

对于setTimeout
```js
setTimeout(function() {
  try {

  } catch(err) {
    
  }
}, 1000)
```

##### Error对象
有name message stack属性

throw可以跑出错误对象，此刻中断try，立刻执行catch

```js
try {
    return 1
} catch {

} finally {

}
```
如果return， finally在return前执行

## 自定义Error
```js
class AError extends Error {

} 
```
```js
try {

} catch(err) {
    if (err instanceof AError) {

    } else {
        throw err
    }
}
```

##### 创建MyError
```js
class MyError extends Error {
  constructor(message) {
    super(message)
    this.name = this.constructor.name
  }
}

class ValidationError extends MyError {}

class PropertyRequiredError extends ValidationError {
  constructor(property) {
    super('no property: ' + property)
    this.property = property
  }
}

console.log(new PropertyRequiredError('filed').name)
```

##### 包装异常
我们没必要把所有异常讨论清除，所以可以用ReadError封装，把错误保存其中并输出
```js
class ReadError extends Error {
  constructor(message, cause) {
    super(message);
    this.cause = cause;
    this.name = 'ReadError';
  }
}

class ValidationError extends Error { /*...*/ }
class PropertyRequiredError extends ValidationError { /* ... */ }

function validateUser(user) {
  if (!user.age) {
    throw new PropertyRequiredError("age");
  }

  if (!user.name) {
    throw new PropertyRequiredError("name");
  }
}

function readUser(json) {
  let user
  try {
    user = JSON.parse(json)
  } catch(err) {
    if (err instanceof SyntaxError) {
      throw new ReadError('syntax error', err)
    } else {
      throw err
    }
  }

  try {
    validateUser(user)
  } catch (err) {
    if (err instanceof ValidationError) {
      throw new ReadError('validationError', err)
    } else {
      throw err
    }
  }
}

try {
  readUser('')
} catch(e) {
  if (e instanceof ReadError) {
    console.log(e)
    console.log(e.cause)
  } else {
    throw e
  }
}
```

## Promise
### promise
为了处理回调地域，引入了Promise
Promise对象有state和result属性，最初state为pending，resolve后未'fulfilled',reject后未'rejected'
result最初undefined，resolve后未resolve的value， reject后未reject 的error

```js
new Promise(function (resolve, reject) {
  resolve('done') //resolve
  console.log('888')//照常运行
    reject('')//将忽略
}).then(function (res) {
  console.log(res)//res 为resolve值
  return Promise.resolve('aaa')
}).finally(function () {
  console.log('kkk')
}).then(function (res) {
  console.log(res)//为之前未处理的
})
```

promise链
```js
new Promise(function(resolve, reject) {
  setTimeout(() => resolve(1), 1000)
}).then(function(result) {
  return new Promise(function(resolve, reject) {
    setTimeout(() => resolve(2), 1000)
  })
}).then(function(result) {
  console.log(result)
})
```

Thenable对象
```js
class Thenable {
  constructor(num) {
    this.num = num
  }

  then(resolve, reject) {
    setTimeout(() => resolve(this.num), 1000)
  }
}

new Promise(resolve => resolve(1))
  .then(result => {
    return new Thenable(result)
  })
  .then(result => console.log(result))
```
具有.then方法的对象，可以被当做promise处理
then中handler返回就是这种对象

如果throw error,等于是reject了，直接取catch处理，且当前函数内容不执行了

### 相关方法
Promise.all()全部promise都resolve，将结果数组组成新promise返回，顺序和之前一致
```js
Promise.all([
  new Promise(res => setTimeout(() => res(1), 1000)),
  new Promise(res => setTimeout(() => res(2), 2000))
]).then(res => {
  console.log(res)
})
```

Promise.allSettled对于结果reject也起作用，其他和Promise.all一样

Promise.race 只等待第一个 settled 的 promise 并获取其结果（或 error）

Promise.any只等待第一个 fulfilled 的 promise，并将这个 fulfilled 的 promise 返回。如果给出的 promise 都 rejected，那么则返回 rejected 的 promise 和 AggregateError 错误类型的 error 实例—— 一个特殊的 error 对象，在其 errors 属性中存储着所有 promise error。

Promise.resolve/Promise.reject返回resolve或reject的promise，就是将值进行封装


##### promisification
promisification指一个接受回调的函数转换为一个返回 promise 的函数。
```js
function promisify(f) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      function callback(err,res) {
        if (err) {
          reject(err)
        } else {
          resolve(res)
        }
      }
      args.push(callback)
      f.call(this, args)
    }) 
  }
}

let loadScriptPromise = promisify(loadScript)
loadScriptPromise(...).then(...)
```

### async await
```js
async function f() {
  let res = await new Thenable(1) //await 对Thenable对象也可以使用
  return 1
}
f().then(res => console.log(res))
```
async返回一个promise

## Generator
Generator可以按需一个接一个的返回多个值，可以与iterable配合使用
```js
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
  yield 4;
}

let generator = generateSequence()
console.log(generator.next())
console.log(generator.next())
console.log(generator.next())
console.log(generator.next())


for (let val of generator) {
    console.log(val)
}
```

使用Generator迭代
```js
let range = {
  from: 1,
  to: 5,
  *[Symbol.iterator]() {
    for (let val = this.from; val <= this.to; val++) {
      yield val
    }
  }
}
console.log([...range])
```

##### 双向yield
```js
function* gen() {
  let result = yield '2 + 2 = ?'
  console.log(result)
}

let generator = gen()

let question = generator.next().value
generator.next(4)
```
先是next到第一个yield取出值，然后再next传入4到result，再到下一个yield

##### generator.throw
```js
function* gen() {
  try {
    let res = yield '2 + 2'
    console.log(res)
  } catch(e) {
    console.log(e)
  }
}

let generator = gen()

let question = generator.next().value
generator.throw(new Error('aaa'))
```
错误被穿进去了， 如果内部没有处理错误，就会出来
```js
function* generate() {
  let result = yield "2 + 2 = ?"; // 这行出现 error
}

let generator = generate();

let question = generator.next().value;

try {
  generator.throw(new Error("The answer is not found in my database"));
} catch(e) {
  alert(e); // 显示这个 error
}
```


##### generator.return 
它直接完成执行，后面的就都结束了
```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

let generator = gen()

generator.next()
generator.return('foo')
generator.next()
```

### 异步Generator
异步可迭代对象
使用Symbol.asyncIterator,
next()返回一个promise，可以通过async实现
使用for await ()
```js
let range = {
  from: 1,
  to: 5,
  [Symbol.asyncIterator]() {
    return {
      current: this.from,
      last: this.to,
      async next() {
        await new Promise(resolve => setTimeout(resolve, 1000))
        if (this.current <= this.last) {
          return { done: false, value: this.current++}
        } else {
          return { done: true }
        }
      }
    }
  }
};

(async () => {
  for await (let val of range) {
    console.log(val)
  }
})()
```

异步Generator
```js
async function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, 1000))
    yield i
  }
}
```

由异步generator实现异步可迭代对象
```js
let range = {
  from:1,
  to: 5,
  async *[Symbol.asyncIterator]() {
    for (let val= this.from; val <= this.to; val++) {
      await new Promise(resolve => setTimeout(resolve, 1000))
      yield val
    }
  }
}


```

## 模块
仅在第一次导入被解析，模块执行一次，在导入间共享

import.meta对象包含关于当前模块的信息

模块要在
```html
<script type="module" src="">
</script>
```
中使用， 下载外部模块不会阻止html处理，它们会与其他资源并行加载，等html加载完成然后运行； 保持脚本相对顺序

```js
import * as say from '...';
say.sayHi()
say.sayBye()
```

```js
export { sayHi as hi }
export { sayBye as default }
```

##### 重新导出
```js
export { sayHi } from ''
```

##### 动态导入
import '' 返回promise
```js
let a = await import('')
```

不能根据条件或者在运行时导入
```js
if () {
    import ...;//Error
}
```

## Proxy Reflect
```js
let proxy = new Proxy(target, handler)
```
hanlder是捕捉器，通过丰富它来让proxy变得强大， 例如get
```js
let numbers = [0, 1, 2];

numbers = new Proxy(numbers, {
  get(target, prop) {
      if (prop in target) {
          return target[prop]
      } else {
          return 0
      }
  }
});
```

Reflect是一个内建对象，可以简化proxy的创建， 对于每个可以被proxy捕获的内部方法，在Reflect中都有一个对应的方法，其名称和参数与Proxy捕捉器相同
例如
```js
let user = {
    name: 'john'
}

user = new Proxy(user, {
    get(target, prop, receiver) {
        return Reflect.get(target, prop, receiver)
        //或者
        return Reflect.get(...arguments)
    }


})

```

撤销Proxy
```js
let object = {
  data: "Valuable data"
};

let {proxy, revoke} = Proxy.revocable(object, {})

console.log(proxy.data)

revoke()

console.log(proxy.data)
```

## Reference Type
```js
let user = {
  name: "John",
  hi() { alert(this.name); },
  bye() { alert("Bye"); }
};

user.hi(); // 正常运行

// 现在让我们基于 name 来选择调用 user.hi 或 user.bye
(user.name == "John" ? user.hi : user.bye)(); // Error!
```
为什么？
为确保user.hi()调用正常运行，js中'.'返回的不是一个函数，而是一个特殊的Reference Type的值
他是一个三个值的组合(base, name, strict)
base是对象名, name是属性名, strict在严格模式下为true
user.hi是一个Reference Type值， 为(user, 'hi', true),当()被Reference Type上调用时， 它们会接收到关于对象和对象的方法的完整信息，设置this(为user)
但是如果
```js
let hi = user.hi
hi()
```
例如此类的赋值等操作，会将Reference Type作为一个整体丢弃掉，而是用user.hi(一个函数)的值并继续传递，所以任何后序操作都失去this，只有obj.method() obj['method']()   才能被正确传递

## BigInt
```js
let bigint = 1111111111111111111111111111n;
let aBig = BigInt('2222222222222222222222222222222')
let big = BigInt(10) // 10n
```
无限长

```js
console.log(1n + 2n) 
console.log(5n / 2n)
```
结果也是BigInt

```js
console.log(1n + 2)//不可以，不能混着用

let bigint = 1n;
let number = 2;

// 将 number 转换为 bigint
alert(bigint + BigInt(number)); // 3

// 将 bigint 转换为 number
alert(Number(bigint) + number); // 3


console.log(1 == 1n) // true
console.log(1 === 1n) // false
console.log(2n > 1) // true
```


他们可以这样用

转换可以返回任意原始类型

这是我用vue3实现的饿了么，如果您感兴趣，希望您能看看，谢谢 https://github.com/goddessIU/vue3-eleme


项目预览地址
https://goddessiu.github.io/