# 现代JavaScript教程学习笔记
这是我学习现代JavaScript教程的笔记，他们的内容确实好，有一种相见恨晚的感觉，当时要是初学js能遇到就好了。。。 没有抄袭，只是笔记，并尝试自己稍微讲一下，如果涉及侵权，会立刻删除
## 原始类型的方法
```js
let t = 'abc'
console.log(t.toUpperCase())//ABC
```
为什么一个原始类型可以调用方法？
在js中有七个原始类型，去掉null undefined,其余五个都有各自的对象包装器，Number String Boolean BigInt Symbol
上面那段代码，发生了这些：
t被包装成一个对象，并且具有我们需要的方法，执行方法返回值，销毁那个对象

## Number
```js
2e3  //2 * 1000
let a = 1_000 // 1000
```
16进制0x: 0xff
8进制: 0o377
2进制: 0b11111111

num.toString()
将num转为字符串，你可以传递你需要的进制
```js
let a = 300
console.log(a.toString(16))
```

### 相关方法
```js
Math.floor()//向下取整
Math.ceil()//up
Math.round()//四舍五入
Math.trunc()//去掉小数部分
Math.random()
Math.max()
Math.pow()
```

```js
isFinite() //是NaN、undefined、null返回false
isNaN()//判断是否为number
```

```js
parseInt()
parseFloat()
toFixed()//返回string
```

### string
有如下方法
```js
let str = 'heyhey'
console.log(str.charAt(0)) //h
str.toUpperCase()
str.toLowerCase()
str.indexOf('hey') // 0, 从哪里开始，没有是-1
str.includes('hey')//true
str.startsWith('hey')//true
str.endsWith('hey')//true
str.slice(0, 3)//hey， 开头和结尾
str.substr(0, 2)//he, 第二个参数是截取几个
str.substring(0, 3)//hey， 开头和结尾
```

### Array
Array有自己的toString
```js
let arr = [1, 2, 3]
String(arr) //1,2,3  
```

##### Array的方法
极其多的方法
```js
let arr = [1, 2, 3]
arr.splice(0, 1, 6)//第一个参数是从哪里开始，第二个参数时删除几个，第三个以后是添加多少内容
console.log(arr)//6, 2, 3


arr.slice(0, 2)//两个参数start, end



console.log(arr.concat([5, 6]))/1, 2, 3, 5, 6
let arrayLike = {
    [Symbol.isConcatSpreadable]: true
}
有这个键的类数组也可以使用此方法


arr.forEach(function(item, index, array) {
    对每个数组内容执行函数
    console.log(item)
})

indexOf()  includes() lastIndexOf()和string一样

arr.find(function(item, index, array) {
    return item === 1
})
如果为true则是返回值

arr.filter(function(item, index, array) {
    //返回所有为true的item组成的数组
})

arr.map(fn) //返回所有结果的数组

arr.sort(function(a, b) {
    //如果值为负数， a 在b前
    //为正数，b在aqian
    //为0，相对位置不变
})

arr.split()
arr.join()

arr.reduce(function(acu, item, index, array) {

}, [initial])
let sum = [1, 2, 3].reduce((acu, item) => {
    return acu + item
}, 0)
console.log(sum)//6
返回值成为下次的acu

console.log(Array.isArray([]))//typeof无法区分{}和[]

以上很多如arr.filter(fun, thisArg)还有第二个参数，为this的值
```

### 迭代
我们的for...of 其实就是对可迭代对象用的
怎么就是可迭代对象？
```js
let obj = {
    from: 1,
    to: 5
}
obj[Symbol.iterator] = function() {
    return {
        current: this.from,
        last: this.to,
        next() {
            if (this.current <= this.last) {
                return {done: false, value:this.current++}
            } else {
                return {
                    done: true
                }
            }
        }
    }
}
for (let i of obj) {
    console.log(i)
}
```
这个obj具有Symbol.iterator键，这个键为一个函数，返回一个对象(迭代器iterator)，对象的next方法是每次迭代调用的，next方法返回{done: true/false, value: }格式对象, 因而obj满足这些，可以迭代

我们也可以显示调用迭代器，
```js
let str = 'hello'
let fun = str[Symbol.iterator]()
while (true) {
    let res = iterator.next()
    if (res.done) break;
    console.log(res.value)
}
```

可迭代对象和类数组有所不同，上面介绍了可迭代对象，类数组是有索引和length属性的对象
Array.from()可将二者转换为数组

## Map Set
```js
new Map()
map.set(key, val)
map.get(key) //没有undefined
map.has(key)
map.delete(key)
map.clear()
map.size

map.keys()
map.values()
map.entries()
可以迭代

map.forEach((value, key, map) => {
    console.log(value, key)
})
```

```js
new Set()
set.add()
set.delete(value)
set.has(value)
set.clear()
set.size
也支持forEach keys values entries()
```

## WeakMap WeakSet
这二者的键必须为对象，如果对象失去引用，二者中有关这个对象的内容就没有了
他们没有keys values entries 方法

## 对象方法
Object.keys(obj)
Object.values(obj)
Object.entries(obj)
Object.fromEntries(array)//返回对象

## Date
```js
new Date()//创建表示当前日期和时间的对象
new Date(milliseconds)

let date = new Date()
date.getFullYear()
date.getMonth()
date.getDate()
date.getHours()
date.getMinutes()
date.getSconds()
date.getMilliseconds()
date.getDay()
date.getTime()//返回时间戳
把如上的get改为set，就是设置
Date对象转换为number，得到时间戳
Date.now()//返回当前时间戳
Date.parse()//从字符串读取时间，返回时间戳
```

## JSON
```js
JSON.stringfy()//跳过函数属性，Symbol类型的键和 值，存储undefined的属性
JSON.parse()
```
字符串和键必须是双引号
两个方法不知一个参数，可以mdn

如果有toJSON,则stringfy会使用它
```js
let obj = {
    num: 23,
    toJSON() {
        return this.num
    }
}
console.log(JSON.stringify(obj))
```

## rest 和spread
```js
function sumAll(...args) {
    for (let arg of args) {
        ...
    }
}
args是一个数组接受了参数
```
arguments是一个类数组对象，获取所有参数

spread:
```js
...arr
展开可迭代对象
```

[...arr]复制数组
{...obj}复制对象

## 全局对象
浏览器中window， node是global， globalThis则是所有环境都支持的
```js
window.alert === alert // true
```
使用polyfills
```js
if (!window.Promise) {
    window.Promise = 
}
```

## 变量作用域，闭包
### 词法环境
每个代码块和全局都有一个词法环境，它包含着环境记录，存储所有局部变量，和对外部词法环境引用
一个函数运行会创建一个新的词法环境
https://zh.javascript.info/closure#ci-fa-huan-jing

## 函数对象 NFE
函数对象的name属性，因为函数就是object
```js
function sayHi() {
    
}
console.log(sayHi.name)
console.log(sayHi.length)//0,参数个数
```

##### NFE： 命名函数表达式
```js
let sayHi = function func(who) {
    if (who) {
        console.log(who)
    } else {
        func('john')
    }
}
它允许函数在内部调用自己，且外部不可见
```

## 装饰器模式和转发
```js
function slow(x) {
    return x
}

function cachingDecorator(func) {
    const cache = new Map()
    return function(x) {
        if (cache.has(x)) {
            return cache.get(x)
        }
        let res = func(x)
        cache.set(x,res)
        return res
    }
}
slow = cachingDecorator(slow)
console.log(slow(1))
```
我们将slow放入一个装饰器，装饰器返回一个函数，这个函数内部处理slow相关功能，第一次看到确实很酷

```js
func(1, 2, 3)
func.call(obj, 1, 2, 3)//this = obj
```

重写上面的包装器
```js
function slow(x) {
    return x
}

function cachingDecorator(func) {
    const cache = new Map()
    return function(x) {
        if (cache.has(x)) {
            return cache.get(x)
        }
        let res = func.call(this, x)
        cache.set(x,res)
        return res
    }
}
slow = cachingDecorator(slow)
console.log(slow(1))
```
这样，我们将this也进行了传递

func.apply和func.call一样，只是第二个参数是类数组对象
```js
let wrapper = function() {
    return func.apply(this, arguments)
}
```

func.bind绑定函数的this,但是不会直接调用

由此，我们可以实现偏函数
```js
function mul(a, b) {
    return a * b
}
let triple = mul.bind(null, 3)
console.log(triple(2))//6
```

## 箭头函数
箭头函数没有this， 它从外部获取
箭头函数不能new，因为它没有this，new需要在过程中创建this
没有arguments

```js
let group = {
    title: 'a',
    nums: [1, 2, 3],
    show() {
        this.nums.forEach(
            num => console.log(this.title + num)
        )
    }
}
group.show()
```

```js
let group = {
    title: 'a',
    nums: [1, 2, 3],
    show() {
        this.nums.forEach(function(num) {
            console.log(this.title + num)
        })
    }
}
group.show()
```
这样会出问题，因为函数内this为undefined




他们可以这样用

转换可以返回任意原始类型

这是我用vue3实现的饿了么，如果您感兴趣，希望您能看看，谢谢 https://github.com/goddessIU/vue3-eleme


项目预览地址
https://goddessiu.github.io/
