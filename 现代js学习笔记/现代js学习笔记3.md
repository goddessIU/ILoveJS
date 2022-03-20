# 现代JavaScript教程学习笔记
这是我学习现代JavaScript教程的笔记，他们的内容确实好，有一种相见恨晚的感觉，当时要是初学js能遇到就好了。。。 没有抄袭，只是笔记，并尝试自己稍微讲一下，如果涉及侵权，会立刻删除
## 对象属性配置
### 属性标志和属性描述符
对象中的属性不仅仅是一个值，它还有属性标志，
所以属性有value writable enumerable configurable四个特性

得到属性描述符
```js
let user = {
    name: 'john'
}
console.log(Object.getOwnPropertyDescriptor(user, 'name'))
```

定义
```js
let user = {
    name: 'john'
}
console.log(Object.getOwnPropertyDescriptor(user, 'age', {
    value: 22,
    writable: true
}))
```

writable: 是否可改
enumerable： 是否可枚举
configurable： 是否可配置，不可配置则不能删除，特性也不能修改但此时writable可由true改为false



##### 配置多个属性
```js
let user = {}
Object.defineProperties(user, {
    name: {
        value: '2',
        writable: true
    },
    age: {
        value: 22,
        writable: true
    }
})
console.log(Object.getOwnPropertyDescriptors(user))
```

其他方法
```js
let user = {}
Object.preventExtensions(user) //禁止添加
Object.seal(user) //禁止添加删除，现有都为configurable:false
Object.freeze(user) //禁止添加更改删除，现有configurable：false,writable: false
Object.isExtensible(user) //这三个是检测上面的
Object.isSealed(user)
Object.isFrozen(user)
```


### getter  setter
之前的是数据属性，现在是访问器属性
```js
let obj = {
    username: 'ww',
    get fullName() {
        return this.username
    },
    set fullName(value) {
        this.username = 'a' + value
    }
}
console.log(obj.fullName)
obj.fullName = 'k'
console.log(obj.fullName)
```
访问器属性有四个特性
get
set
configurable
enumerable

## 原型与继承
js中的继承不同于java，其继承基于原型链，感觉挺灵活的，也挺绕的
[[Prototype]]是对象的原型
我们一般用__proto__操作，__proto__是它的set get
```js
let animal = {}
let rabbit = {}
rabbit.__proto__ = animal
```
当我们使用一个属性，就会顺着原型链往上找，在哪里找到就在哪里停下
如果一个函数用的this， this与原型链无关，与调用者有关
```js
let animal = {
    eats: true,
    isEating() {
        return this.eats
    }
}

let rabbit = {
    eats: false,
    __proto__: animal
}

console.log(rabbit.isEating())
```

##### 获取属性
Object.keys(obj)返回自己的属性
for in 则返回所有，包括原型
如果我们想循环只有自己的， obj.hasOwnProperty(property)可以判断
```js
let animal = {
    eats: true
  };
  
  let rabbit = {
    jumps: true,
    __proto__: animal
  };

  for (let prop in rabbit) {
      let isOwn = rabbit.hasOwnProperty(prop)

      if (isOwn) {
          console.log(prop)
      } else {
          console.log('*', prop)
      }
  }

```

### F.prototype
new F()， 构造函数有prototype属性，new F()之后，会让新创建的对象的[[Prototype]]为构造函数的prototype(需要F.prototype是一个对象)

```js
let animal = {
    eats: true
}

function Rabbit() {

}

Rabbit.prototype = animal

let rabbit = new Rabbit()
console.log(rabbit.eats)
```

F.prototype属性有一个constructor属性指向F，所以两个家伙感觉是相互指，所以新的对象也可以使用constructor这个属性
```js
function Rabbit() {}
console.log(new Rabbit.constructor())
```

```js
let obj = {}
alert(obj) 
```
显然obj用了toString()，但是哪来的？它的[[Prototype]]是Object， 所以用了上面的方法

##### 借用原型
```js
let obj = {
    0: 'hello',
    1: 'a',
    length: 2
}
obj.join = Array.prototype.join
console.log(obj.join(','))
```
也可以直接obj.prototype = Array.prototype，但是只能继承一个原型

##### 与原型有关的方法
```js
Object.create()
第一个参数是原型，第二个是属性描述符

let obj = Object.create(null)
一个plain对象，什么都没有继承，原型是null
```

```js
Object.getPrototypeOf(obj)//得到prototype
Object.setPrototypeOf(obj, proto) //设置prototype
```

## 类
```js
class MyClass {
    constructor(name) {
        this.name = name
    }
    
    sayHi() {
        console.log(this.name)
    }
}

let user = new MyClass('a')
user.sayHi()
```
当new的时候，调用constructor，构造对象

class本质上是一个Function， 它是个constructor，它的prototype中放置了类的方法如sayHi

##### 类表达式
```js
let User = class MyClass {
    
}
let user = new User()
```

##### getter/setter
```js
class User {
    constructor(name) {
        this.name = name
    }

    get name() {
        return this._name
    }

    set name(val) {
        this._name = val
    }
}
```


##### class字段
```js
class User {
    age = 20;
    constructor(name) {
        this.name = name
    }

    get name() {
        return this._name
    }

    set name(val) {
        this._name = val
    }
}
```
类字段不在prototype上，而是在每个对象上

### 类继承
```js
class Animal {
    constructor() {

    }

    sayHi() {
        console.log('###')
    }
}

class Cat extends Animal {
    constructor() {
        super()
    }

    sayHi() {
        super.sayHi()
        console.log('***')
    }
}


let cat = new Cat()
cat.sayHi()
```

constructor中要调用super


本质上就是两个构造函数，将它们的prototype用原型链串起来
在js中，还可以extends 函数

我们在new的时候，创建一个空对象，并赋值给this
但是继承用的父类的constructor，它期望父类完成相关工作，所以必须要有super

##### 重写类字段
```js
class Animal {
  name = 'animal';

  constructor() {
    alert(this.name); // (*)
  }
}

class Rabbit extends Animal {
  name = 'rabbit';
}

new Animal(); // animal
new Rabbit(); // animal
```
父类的类字段在super前创建，子类在之后所以会如此

##### [[HomeObject]]
我们可以简单尝试实现一下类
```js
let animal = {
    name: 'Animal',
    eat() {
        console.log(this.name)
    }
}

let rabbit = {
    name: 'Rabbit',
    __proto__: animal,
    eat() {
        this.__proto__.eat.call(this)
    }
}

rabbit.eat()
```

但是，如果再来一层继承？
```js
let animal = {
  name: "Animal",
  eat() {
    alert(`${this.name} eats.`);
  }
};

let rabbit = {
  __proto__: animal,
  eat() {
    // ...bounce around rabbit-style and call parent (animal) method
    this.__proto__.eat.call(this); // (*)
  }
};

let longEar = {
  __proto__: rabbit,
  eat() {
    // ...do something with long ears and call parent (rabbit) method
    this.__proto__.eat.call(this); // (**)
  }
};

longEar.eat();
```
不可以，longEar.eat()实际上执行longEar.__proto__.eat.call(longEar), 就是rabbit.eat.call(longEar)，就是longEar.__proto__eat.call(longEar)，又回来了，循环引用，然后爆栈了


所以使用了[[HomeObject]]
因此，在类的方法中加入了[[HomeObject]]，他是这个对象，因此在super的时候我们就知道该做什么了

### 静态属性和方法
```js
class User {
    static staticMethod() {
        console.log('b')
    }

    static name = 'a'
}

console.log(User.name)
User.staticMethod()
```
静态的可以继承， extends时就实现了，是将两个构造函数用prototype串联起来


### 受保护的字段
这并非js语言级别的，它并没有被实现，只是一种大家的约定
_name这就是保护字段，你不能直接访问它
```js
class CoffeeMachine {
    _waterAmount = 0;
    set waterAmount(value) {
        this._waterAmount = value
    }
    get waterAmount() {
        return this._waterAmount
    }
    constructor(power) {
        this._power = power
    }
}

let coffeeMachine = new CoffeeMachine(100)

coffeeMachine.waterAmount = 10
console.log(coffeeMachine.waterAmount)
```

### 私有属性和方法
私有属性以#为前缀
```js
class CoffeeMachine {
    #waterLimit = 200;

    #fixWaterAmount(value) {
        return this.#waterLimit
    }

    setWaterAmount(value) {
        this.#waterLimit = this.#fixWaterAmount(value)
    }
}

let coffeeMachine = new CoffeeMachine()

```
现在这些#我们无法从外部访问
他们无法直接继承，只能用get/set操作

### 内建类
```js
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }
}

let arr = new PowerArray(1, 2, 5, 10, 50);
alert(arr.isEmpty()); // false

let filteredArr = arr.filter(item => item >= 10);
alert(filteredArr); // 10, 50
alert(filteredArr.isEmpty()); 
```
filter返回的数组是PowerArray，如果想返回Array，可以
```js
class PowerArray extends Array {
  isEmpty() {
    return this.length === 0;
  }

  // 内建方法将使用这个作为 constructor
  static get [Symbol.species]() {
    return Array;
  }
}
```
内建类没有方法继承， 如Object和Date


### instanceof
这个操作符判断对象是否属于某个class， 包括继承
```js
class Animal {
    static [Symbol.hasInstance](obj) {
        if (obj.canEat) return true
    }
}

let obj = { canEat: true }
console.log(obj instanceof Animal)

```
如果类有静态方法Symbol.hasInstance则调用它


如果没有，就判断是否在原型链上

##### 相关方法
objA.isPrototypeOf(objB)判断是否在原型链上，可以Class.prototype.isPrototypeOf(obj)

##### Symbol.toStringTag
```js
let user = {
    [Symbol.toStringTag]: 'user'
}
```
这个属性定义对象的toString行为
很多内部对象都有


### Mixin模式
因为通过原型链，只能单继承，所以需要mixin帮助
```js
let sayHiMixin = {
    sayHi() {
      alert(`Hello ${this.name}`);
    },
    sayBye() {
      alert(`Bye ${this.name}`);
    }
  };
  
  // 用法：
  class User {
    constructor(name) {
      this.name = name;
    }
  }
Object.assign(User.prototype, sayHiMixin)
new User('d').sayHi()
```

mixin的对象再继承
```js
let sayMixin = {
  say(phrase) {
    alert(phrase);
  }
};

let sayHiMixin = {
  __proto__: sayMixin, // (或者，我们可以在这儿使用 Object.setPrototypeOf 来设置原型)

  sayHi() {
    // 调用父类方法
    super.say(`Hello ${this.name}`); // (*)
  },
  sayBye() {
    super.say(`Bye ${this.name}`); // (*)
  }
};

class User {
  constructor(name) {
    this.name = name;
  }
}

// 拷贝方法
Object.assign(User.prototype, sayHiMixin);

// 现在 User 可以打招呼了
new User("Dude").sayHi(); // Hello Dude!
```
sayHiMixin中的super，是通过mixin原型查找方法而不是class，因为super中的[[HomeObject]]就是mixin原型

方法sayHi中的[[HomeObject]]内部引用的是sayHiMixin， 当super在[[HomeObject]].[[Prototype]]中寻找父方法时，搜索的是sayHiMixin.[[Prototype]]



他们可以这样用

转换可以返回任意原始类型

这是我用vue3实现的饿了么，如果您感兴趣，希望您能看看，谢谢 https://github.com/goddessIU/vue3-eleme


项目预览地址
https://goddessiu.github.io/