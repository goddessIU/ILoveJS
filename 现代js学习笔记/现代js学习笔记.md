# 现代JavaScript教程学习笔记
这是我学习现代JavaScript教程的笔记，他们的内容确实好，有一种相见恨晚的感觉，当时要是初学js能遇到就好了。。。
没有抄袭，只是笔记，并尝试自己稍微讲一下，如果涉及侵权，会立刻删除
## 基础部分
### use strict
```js
'use strict'
//现代模式
```
可以放在脚本开头或者函数体内部

### 交互
alert
显示信息
```js
alert('111')
```
prompt 
```js
const m = prompt('your name?', 'george')
```
第二个参数为默认值， m为用户输入值，若esc取消则返回null

confirm
```js
let a = confirm('yes?')
```
确定为true， 否则为false

### ??
??与||类似
我们认为null/undefined为未定义，??返回第一个已经定义的值
```js
let a = null ?? 'a'
console.log(a)
```

### 数据类型
js共有七种原始类型和一种引用类型
原始类型： 
1. Number
2. BigInt(32n, n结尾代表bigint)
3. Bool
4. String
5. null
6. undefined
7. Symbol
引用类型
8. object

typeof检测类型
```js
console.log(typeof '111')//'string'
```

注意
```js
console.log(typeof null)//object
console.log(typeof function() {})//function
```
Null并非object，只是遗留问题；
function也是object， 但是typeof可以细致区分

### 类型转换
js可以将变量类型进行转换，其中包括显示和隐式
##### 显示转换
```js
Number('123')  //123
Boolean(1) //true
```

##### 隐式转换
如果你自己不体面，js引擎就帮你体面
```js
+'123' //number: 123
'6' / '2' //number: 3
```
虽然是字符串，但是会变为number

js中还有很多奇奇怪怪的转换，但我觉得没人会在实际中搞那种事情吧


## 代码质量
##### 调试
没啥说的，这原文写的不好总结啊
https://zh.javascript.info/debugging-chrome


##### 代码风格
1. 换行
```js
if (true) console.log('***')


if (true) {
    console.log('***')
}
```
相对推荐的两种方式，要么放{}， 要么单行不换行

2. 一行不能过长
```js
let a = `hello thank you, thank you very much` 
```
不如
```js
let a = `hello thank you,
        thank you very much
        `
```


google JavaScript代码风格：https://google.github.io/styleguide/jsguide.html


## 注释
好的代码不需要注释说明他做了什么，避免
```js
//这段代码先买了瓶水，又喝了口水
```
推荐
jsDoc:
```js
/**
 * 返回x 的 二次方
 * 
 * 
 * @param {number} x 输入的值
 * @return {number} x的2次方
 */
function(x) {
    return x ** 2
}
```

##### 自动化测试
https://zh.javascript.info/testing-mocha


##### polyfill和转译器
js不断发展，但是旧的引擎没有内置新的语法，所以我们可以：
1. 转译器： 将一种源码转换为另一种源码
```js
let a = b ?? 'a'
转为
let a = (b != undefined && b !== null) ? b : a
```
2. polyfill
则是原来不具备的东西，把它加进去
假设pow函数不具备
```js
if (!Math.pow) {
    Math.pow = (x, n) => {
        return x ** n
    }
}
```

## object
### object基本内容
object类型是引用类型，我们可以这么构建一个对象
```js
let obj = {
    name: 'george'
}
或者
let obj = new Object()
```

对于对象，它是由键值对形成的，所以操纵这些键尤为重要
```js
delete obj.name //删除
obj.age = 22//添加
obj.name = 'a'//修改
'name' in obj //判断是否存在

for (let k in obj) {
    console.log(obj[k])
}
//遍历
```

操纵key可以通过点号.和中括号[]
```js
obj.name 

obj['name']

let k = 3
obj[k]//中括号可以放变量
```

js中的key都是string类型，无论你怎么搞(Symbol类型除外)
key中整数类型的key按大小排序，其他按创建顺序排序
```js
let obj = {
    45: 1,
    2: 2
}
for (let i  in obj) {
    console.log(obj[i])
}


let obj = {
    name: '2',
    age:'3'
}
for (let i  in obj) {
    console.log(obj[i])
}
```

### 克隆
因为对象是引用类型
```js
let obj = {
    name: '1'
}
```
obj更像是一个名片，它指向一个内存，上面记录着{name: '1'},大概这个感觉吧，其实学过c可能会比较理解一点

所以，有如下：
```js
let user = {}
let a = {}
a === user//false

let user = {}
let b = user
user === b//true
```
比较的是是否来自一个引用

那么如何克隆一个对象?
```js
let user = {
    name: '11'
}
let a = Object.assign({}, user)
console.log(a)
```
最后a为克隆对象，第一个参数也变了，如果键有重复，会出现覆盖现象

如果出现某个属性值为对象，则需要深度克隆, Object.assign()将不起作用
```js
let user = {
    name: '11',
    a: {
        name: '2'
    },
    b: [2,3,5]
}
let a = {}
deepClone(user, a)
console.log(a)
function deepClone(obj, temp) {
    for (let i in obj) {
        if (typeof i === 'object') {
            temp[i] = {}
            deepClone(obj[i], temp[i])
        } else if(typeof i === 'array'){
            temp[i] = []
            deepClone(obj[i], temp[i])
        } else {
            temp[i] = obj[i]
        }
    }
}
```
大概这样，依赖递归，这是个大体思路

## ?.可选链
在代码中，会有这样的问题
```js
let a = user.friends.name
```
如果name不存在， user.friends为undefined,则会报错不能访问undefined的name
我们可以
```js
if (user.friends) {
    a = user.friends.name
}
```
但如果一个更复杂的属性呢？
可选链?.帮我们解决了很多
```js
let a = user?.friends
如果user存在返回user.friends
不存在返回undefined
```
这只对前面的user， 对后面不起到检查作用， 可以在这些情况下使用它
```js
let user = {
    a: {
        name: 'a'
    },
    b() {
        console.log('ccc')
    },
    hi: 3
}
user?.a?.name
user.b?.()//调用函数
user?.['hi']
```
但如果user是个必须存在的值，不建议user后面加?.否则，如果user那里出现问题，我们将无从知道

### this
js是动态的语言，所以它是相对自由的，所以this也很自由，所以就出现各种烦人的this问题
```js
let user = {
    name: '111',
    fun() {
        console.log(this.name)
    }
}
user.fun()//111
```
看样子如同预期，user.fun()打印了user的name

但是
```js
function fun () {
    console.log(this.name)
}
let user = {
    name: 'user'
}
let obj = {
    name: 'obj'
}
user.f = fun
obj.f = fun
user.f()//user
obj.f()//obj
```
可以发现js中的this指向调用者无论是.还是[]

特例：箭头函数没有this,所以箭头函数中的this为它外部函数this指向
```js
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya
```

### Symbol
symbol类型可以如此创建
```js
let id = Symbol('id')
```
第一次学感觉没啥用，后来接触的多点渐渐体会到它的用处了
symbol是唯一的，括号中是该symbol的描述，哪怕描述一样，两个symbol也不等，且symbol不能自动转换为字符串，需要显示转换
```js
let id = Symbol('id')
let id1 = Symbol('id')
console.log(id === id1)//false
```

全局注册表
```js
let sy = Symbol.for('name')//如果存在描述为name的symbol就返回，否则就创建
let m = Symbol.keyFor(sy) //返回name
```
object可以用symbol做键，这样在for...in, Object.keys()等不会访问到，想当与一定程度的隐身；Objcet.assign()则可以获取到
当然，symbol在内置系统中用到很多，如对象是否可迭代，需要判断是否有一个symbol属性，等等

## 垃圾回收
垃圾回收会回收掉不再可达的值，例如
```js
let user = {}
user = null
```
那么user指向的对象就被回收了

```js
let a = {}
let admin = {}
admin = a
a = null//不会清除，因为还有admin引用
```
垃圾清除会找到所有根并标记，然后标记他们的引用，然后再标记引用的引用，最后没标记的清除


## 构造器和new
函数用new，就是构造函数
```js
function User(name) {
    this.name = name
}
let user = new User('aa')
console.log(user)
```
这其中发生了什么
```js
function User(name) {
    //this = {} 创建空对象，并设置this
    this.name = name //添加属性到this
    //return this返回this
}
```

new.target判断是否用了new
```js
function User(name) {
    if (!new.target) {
        return new User(name)
    }
    this.name = name
}
new User('aa')
```
如果构造器return对象，则接受这个对象，覆盖原有this
否则返回this

## object 转换为原始类型
```js
alert(obj)//obj-> string
+obj //obj -> number
```
类型转换有三种辩题，发生在string、number、default三种情况，成为hint
string hint: 
```js
alert(obj)

aObj[obj] = 123 // obj做键
```

number hint:
```js
let num = Number(obj)

let n = +obj
let delta = date1 - date2

let greater = user1 > user2
```

default hint:
```js
let total = obj1 + obj2
if (user == 1) {}
```
根据情况，哪种情况用哪种hint， 不明确一般就是default hint

为了转换，调用obj[Symbol.toPrimitive](hine)--带有symbol键的Symbol.toPrimitive的方法，如果存在
否则，如果hint是string--尝试obj.toString()和obj.valueOf(),无论哪个存在
否则，如果hint是number或default--尝试obj.valueOf()和obj.toString(),无论哪个存在

##### Symbol.toPrimitive
```js
let obj = {
    name: 'a'
}
obj[Symbol.toPrimitive] = function(hint) {
    console.log(hint)
    return hint === 'string' ? 'this.name' : 'e'
}
console.log(+obj) //number NaN
console.log(obj + 500) //number e500
```

##### toString valueOf
对于string hint， 存在toString就用， 不然用valueOf
对于其他hint， valueOf不存在用toString， 否则用valueOf
普通对象拥有这两个方法，toString返回'[object Object]',valueOf返回对象自身
```js
let user = {
    name: 'a',
    money: 100,
    toString() {
        return `${this.name}`
    },
    valueOf() {
        return this.money
    }
}
alert(user)
alert(+user)
```
他们可以这样用

转换可以返回任意原始类型


这是我用vue3实现的饿了么，如果您感兴趣，希望您能看看，谢谢
https://github.com/goddessIU/vue3-eleme


项目预览地址
https://goddessiu.github.io/