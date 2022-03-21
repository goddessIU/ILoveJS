# 现代JavaScript教程学习笔记
## Document
1. DOM
文档对象模型简称DOM，将所有页面内容表示为可以修改的对象
document对象是页面的主要“入口点”，可以使用它更改或创建页面上的任何内容
```js
document.body.style.background = 'red'
```

2. BOM
浏览器对象模型， 表示由浏览器提供的用于处理文档之外所有内容的其他对象，如navigator location

通常有道四种节点类型：
1.document： DOM的入口点
2.元素节点： HTML标签
3.文本节点： 包含文本
4.注释


### 遍历DOM
<html> = document.documentElement
<body> = document.body
<head> = document.head
```js
ele.childNodes //所有子节点， 一个类数组的可迭代对象，是一个dom集合
ele.firstChild //第一个
ele.lastChild //最后一个

ele.nextSibling //下一个兄弟节点
ele.previousSibling //上一个兄弟节点
ele.parentNode //父节点
```
如果只是想获得元素节点可以
```js
children//元素子节点
firstElementChild, lastElementChild
previousElementSibling, nextElementSibling
parentElement
```

对于表格
```js
<table>支持一下属性
table.rows // <tr>元素集合
table.caption/tHead/tFoot 引用元素<caption> <thead> <tfoot>
table.tBodies //<tbody>元素的集合

<thead> <tfoot> <tbody>提供了rows属性

<tr>:
tr.cells 给定<tr>中<td>  <th>的集合
tr.sectionRowIndex 给定tr在封闭的<thead>/<tbody>/<tfoot>中的位置
tr.rowIndex  在表格中tr的编号

<td> <th>:
td.cellIndex 在tr中编号
```

### 获取元素
```js
document.getElementById 用id获取元素
querySelectorAll(css) //返回匹配的所有元素
querySelector
elem.matches(css) //检查elem是否与给定的css选择器匹配
elem.closest(css) //查找与css选择器匹配的最近的祖先
elem.getElementsBy*
```


### 节点属性
不同DOM节点可能有不同的属性，这与他们之间的继承有关
EventTarget为最高级，node继承自eventTarget， element text
comment 继承node， htmlElement继承element，HTMLINputElement， HTMLBodyElement HTMLAnchorElement继承HTMLElement


EventTarget <- Node <- Element <- htmlElement <-HTMLBodyElement HTMLAnchorElement

EventTarget:主要处理事件相关属性
node:一个抽象类，dom节点的基础如parentNode
Element：DOM元素的基本类， 提供了元素级别的导航如children， querySelector
HTMLElement： html元素的基类

##### nodeType属性
元素节点 elem.nodeType === 1
文本节点 elem.nodeType === 3
document: elem.nodeType === 9


##### nodeName tagName
```js
document.body.nodeName //BODY
document.body.tagName //BODY
```
tagName仅适用于element节点
nodeName为任意Node定义


##### innerHTML:内容
```js
document.body.innerHTML = 'aaa'
div.innerHTML = '<em>aaa</em>' //可以插入html
```

##### outerHTML:元素完整HTML
outerHTML 属性包含了元素的完整 HTML。就像 innerHTML 加上元素本身一样。

写入outerHTML不会改变你获取的元素，而是在DOM中替换他
```html
<div>Hello, world!</div>

<script>
  let div = document.querySelector('div');

  // 使用 <p>...</p> 替换 div.outerHTML
  div.outerHTML = '<p>A new element</p>'; // (*)

  // 蛤！'div' 还是原来那样！
  alert(div.outerHTML); // <div>Hello, world!</div> (**)
</script>
```
div被移除，插入新的内容，div仍拥有旧的值

非元素节点如文本，用data获取内容

##### textContent
返回元素的纯文本，不带有tag标签

##### hidden属性
```html
<div hidden>With the attribute "hidden"</div>

<div id="elem">JavaScript assigned the property "hidden"</div>

<script>
  elem.hidden = true;
</script>
```

##### 对于更多属性
console.dir(elem) 输出元素并读取其属性

### 特性和属性
特性： attribute 属性：property
<body id="page">特性
body.id = "page" 属性


##### DOM属性
DOM节点是常规的JavaScript对象
```js
document.body.myData = {
    name: 'aaa'
}

Element.prototype.sayHi = function() {}
```

### HTML特性
标准的特性成为对应的DOM属性，非标准的特性则不会

访问特性的方法
```js
elem.hasAttribute(name)
elem.getAttribute(name)
elem.setAttribute(name, value)
elem.removeAttribute(name)
```
特性不分大小写

当特性被修改对应属性也会自动更新，通常，反之亦然

input.checked是boolean， style特性是字符串类型，但style是对象

##### 非标准的特性dataset
所有以data-开头的特性被程序员使用，可在dataset属性中使用
```html
<body data-about="Ele">
    </body>
    <script>
        console.log(document.body.dataset.about)
    </script>
```

### 修改文档
创建一个元素
document.createElement(tag) 
```js
let div = document.createElement('div')
let textNode = document.createTextNode('Here')
```

插入方法
```js
let div = document.createElement('div')
document.body.append(div)
```

```js
node.append(node or string) //node末尾插入节点或字符串
node.prepend() //node开头
node.before() // node前面
node.after() // node后面
node.replaceWith() //node替换
```
让内容中所有标签和其他东西都作为html代码插入
```js
elem.insertAdjacentHTML(where, html)
where: 
beforebegin //html插入到elem前
afterbegin //html插入到elem开头
beforeend //elem结尾
afterend //elem后面
```

节点移除
node.remove()
如果我们要将一个元素移动到另一个地方，无需将其从原来的位置删除
```html
<div id="first">First</div>
<div id="second">Second</div>

所有插入方法自动从旧位置删除节点
<script>
  // 无需调用 remove
  second.after(first)
</script>
```

克隆节点
elem.cloneNode(true)深刻隆
false是浅克隆

### DocumentFragment
是一个特殊的DOM节点，用作来传递节点列表的包装器，我们可以向其附加节点，但是当我们将其插入某个位置，则会插入内容
```html
<body data-about="Ele">
        <ul id="ul"></ul>
    </body>
    <script>
      function getListContent() {
          let fragment = new DocumentFragment()
          for (let i = 1; i <= 3; i++) {
              let li = document.createElement('li')
              li.append(i)
              fragment.append(li)
          }
          return fragment
      }
      ul.append(getListContent())
    </script>
```

老式方法
parentElem.appendChild(node)
parentElem.insertBefore(node, nextSibling)
parentElem.replaceChild(node, oldChild)
parentElem.removeChild(node)

document.write(html)调用只在页面加载时工作
如果稍后调用它，现有文档内容将被擦除，它在加载完成阶段是不可用的

当浏览器正在读取传入html时调用document.write方法来写入一些东西，浏览器会像本来在html文本中那样使用它，因为它 不涉及dom修改，直接写到页面文本中

### 样式和类
```html
类<div class=""></div>
样式<div style=""></div>
```

修改css
```js
let top = ;
let left = ;
elem.style.left = left
elem.style.top = top
```

elem.className对应class特性
elem.classList属性，是一个对象具有add/remove/toggle方法用来操作class
```js
document.body.classList.add('article')
```
elem.classList.toggle(class)
elem.classList.add/remove(class)
elem.classList.contains(class)

elem.style仅对style特性起作用，没有css级联，通过getComputedStyle
```js
getComputedStyle(element, [pseudo])
element: 需要被读取样式值的元素
pseudo: 伪元素， 如::before
```

```html
<head>
  <style> body { color: red; margin: 5px } </style>
</head>
<body>

  <script>
    let computedStyle = getComputedStyle(document.body);

    // 现在我们可以读取它的 margin 和 color 了

    alert( computedStyle.marginTop ); // 5px
    alert( computedStyle.color ); // rgb(255, 0, 0)
  </script>

</body>
```

### window大小和滚动
```js
document.documentElemen.clientHeight/clientWidth//窗口的宽高
window.innerWidth/innerHeight//计算了滚动条

document.documentElement.scrollWidth/scrollHeight//理论上文档的完整大小， 但是出于实际要如下方式获得
let scrollHeight = Math.max(
    document.body.scrollHeight, document.documentElement.scrollHeight,
    document.body.offsetHeight, document.documentElement.offsetHeight,
    document.body.clientHeight, document.documentElement.clientHeight,
)


document.documentElement.scrollLeft/scrollTop //当前滚动状态，也可以
window.pageXOffset/window.pageYOffset


更改滚动：
document.documentElement.scrollTop/scrollLeft //对页面进行操作
window.scrollBy(x, y) //相对坐标
window.scrollTo(pageX, pageY) //绝对坐标
elem.scrollIntoView(top) //将滚动页面使elem可见，top=true页面滚动elem出现窗口顶部， false是底部

document.body.style.overflow = 'hidden' //不可滚动
```


### 元素大小和滚动
offsetParent, offsetLeft/Top
offsetParent: 是最接近的祖先（css定位absolte relative fixed 或 td th table 或 body)
offsetLeft/offsetTop 相对于offsetParent左上角的x/y坐标

offsetWidth/Height元素的完整大小， content text + padding + border

clientTop/Left 左边框上边框宽度

clientWidth/Height ： content width + padding，不包括滚动条

scrollWidth/Height  包括滚动的部分，整个宽高

scrollLeft/scrollTop 是元素的隐藏滚动部分的width/height

不从css中获取height width，因为box-sizing会使得无法达到预期，或者得出auto等值


### 坐标
相对于窗口： 类似fixed, 从窗口的顶部、左侧计算clientX clientY
相对于文档， 如absolute， 从文档顶部左侧计算 pageX pageY

elem.getBoundingClientRect() 返回最小矩形的窗口坐标
返回值的属性有x/y, 矩形原点相对于窗口的X/Y坐标
width/height
top/bottom  顶部、底部矩形边缘的Y坐标
left/right  左右矩形边缘的X坐标
是窗口坐标


document.elementFromPoint(x, y) 返回在窗口坐标(x,y )处嵌套最多的元素

文档坐标， 从文档左上角开始计算不是窗口
pageY = clientY + 垂直部分滚出
pageX = clientX + 水平滚出


## 浏览器事件

```js
鼠标事件:
click
contextmenu //鼠标右键点击一个元素
mouseover/mouseout 
mousedown/mouseup
mousemove

键盘事件:
keydown /keyup 

表单元素事件:
submit
focus

document事件
DOMContentLoaded //HTML加载和处理均完成，DOM被完全构建完成

CSS事件:
transitionend //当一个CSS动画完成
```

因此，我们可以添加处理程序
on<event>特性
```html
<input value="a" onclick="alert(this, 'Click')" type="button">
this是处理的元素


button.onclick = function(event) {

}
button.onclick = null//移除
```

分配多个处理程序，使用
```js
ele.addEventListener(event, handler, options)
options: once 为true， 触发后会被删除监听器
capture： 捕获与冒泡
passive: true，不会调用preventDefault()

removeEventListener
```

事件对象的属性
```js
elem.onclick = function(event) {
    event.type //click
    event.currentTarget //处理事件的元素
    event.clientX //针对窗口的相对坐标
    event.clientY
}
```

如果一个 对象有handleEvent方法，也可以作为处理程序
```js
let obj = {
          handleEvent(event) {
              console.log(event.type, event.currentTarget)
          }
      }
elem.addEventListener('click', obj)
```

### 冒泡和捕获
```html
<form onclick="alert('form')">FORM
  <div onclick="alert('div')">DIV
    <p onclick="alert('p')">P</p>
  </div>
</form>
```
点击p， 会p一直往上传， p  div form body document，就是冒泡


event.target 为引发事件的目标元素
this(currentTarget)是当前元素

停止冒泡
event.stopPropagation()

```html
<body onclick="alert(`the bubbling doesn't reach here`)">
  <button onclick="event.stopPropagation()">Click me</button>
</body>
```

stopPropagation停止向上移动，但是当前元素的其他处理程序会执行，stopImmediatePropagation可以停止冒泡，并停止执行程序

##### dOM事件三个阶段
1.捕获阶段，由window一层层到目标元素
2.目标阶段，到达目标元素
3.冒泡阶段，由目标元素向上
捕获阶段和冒泡阶段都在目标阶段触发，即目标阶段会发生捕获和冒泡

### 事件委托
不把事件安排在内部元素，而是安排在父元素，统一处理

### 浏览器默认行为
点击链接会跳转就是默认行为

有时候我们不需要这些默认行为， 怎么阻止？

1. event.preventDefault()
2. on<event>返回false(其他返回值没有任何意义)

```html
<a href="/" onclick="return false"></a>

<a href="/" onclick="event.preventDefault()"></a>
```
但有时候事件相互关联，一个事件禁止了，其他事件可能也受影响
```html
<input value="Focus works" onfocus="this.value=''">
<input onmousedown="return false" onfocus="this.value=''" value="Click me">
```
第二个因为mousedown禁了，所以focus也无法使用

在addEventListener中选项passive：true， 也可以起到作用

如果默认行为被禁，event.defaultPrevented为true


### 创建自定义事件
```js
let event = new Event(type, options)
let event = new Event('click')
options: {
    bubbles: true/false 是否冒泡
    cancelable: true/false true阻止默认行为
}
```

```js
let event = new Event('click')
elem.dispatchEvent(event) //派发事件
```

对于UI事件如UIEvent  FocusEvent等，创建这样的事件，使用UI事件例如new MouseEvent('click')

```js
let event = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    clientX: 100,
    clientY: 100
})
console.log(event.clientX) //100
```

自定义事件应该使用new CustomEvent， 他的第二个参数允许添加属性detail，用于描述自定义信息


##### event.preventDefault()
对于新的，自定义的事件，绝对没有默认的浏览器行为，但是分派（dispatch）此类事件的代码可能有自己的计划，触发该事件之后应该做什么。
调用event.preventDefault()，处理程序发出一个信号，指出这些行为被取消
这样dispatchEvent()返回false
```html
<pre id="rabbit">
  |\   /|
   \|_|/
   /. .\
  =\_Y_/=
   {>o<}
</pre>
<button onclick="hide()">Hide()</button>

<script>
  function hide() {
    let event = new CustomEvent("hide", {
      cancelable: true // 没有这个标志，preventDefault 将不起作用
    });
    if (!rabbit.dispatchEvent(event)) {
      alert('The action was prevented by a handler');
    } else {
      rabbit.hidden = true;
    }
  }

  rabbit.addEventListener('hide', function(event) {
    if (confirm("Call preventDefault?")) {
      event.preventDefault();
    }
  });
</script>
```

事件是同步的，在一个事件内部发生一个事件，会立刻执行，然后结束后立刻执行外部事件

## UI事件
鼠标事件顺序
mousedown -> mouseup -> click -> dbclick
mousedonw -> mouseup -> contextmenu

event.button === 0//左键
                 1 //中键
                 2 //右键


event.shiftKey: shift
event.altKey: Alt
event.ctrlKey: Ctrl
```js
button.onclick = function(event) {
    if (event.altKey && event.shiftKey) {
        console.log('**')
    }
}
//鼠标点击+alt+shift
```

event.clientX/clientY/pageX/pageY 可以获得鼠标坐标

dbclick可能会导致选择文本，可以阻止其默认行为
阻止oncopy行为禁止复制


### 移动鼠标事件
mouseover/mouseout  移入移除鼠标
event.target  移到的元素/离开的元素
event.relatedTarget  之前的那个元素

mousemove 鼠标移动，如果太快，中途很多元素会被略过

mouseover和mouseout可以在父子元素之间发生

mouseenter mouseleave 不区分父子元素，即父到子不会产生影响，且不会冒泡


### 鼠标拖放事件
dragstart dragend事件是原生拖拽事件，但是具有一定局限性

需要自己实现拖拽算法
先进行mousedown，然后mousemove更改position为absolute不断调整left/top， 最后mouseup
同时禁止dragstart默认行为，以免与内置拖拽事件冲突


### 指针事件
pointerdown  mousedown
pointerup    mouseup
pointermove  mousemove
pointerover  mouseover
pointerout   mouseout
pointerenter mouseenter
pointerleave  mouseleave
pointercancel 
gotpointercapture
lostpointercapture


指针事件具备和鼠标完全相同的属性，clentX/Y target，以及
pointerId : 触发当前事件指针的唯一标识符
pointerType : 指针的设备类型， 如mouse pen
isPrimary: 当指针为首要指针(多点触控时按下的第一根手指) 时为true

多点触控，第一个手指触摸，pointerdown时间触发，isPrimary=true,并且被指派pointerId
后序isPrimary=false


pointercancel
指针交互被中断时触发，中断原因可能如下： 指针设备硬件在物理层面被禁用，设备旋蒸，浏览器打算自己处理交互

我们可以阻止浏览器的默认行为来防止pointercancel触发，
```js
1.阻止原生的拖放操作发生
ball.ondragstart = () => false
2. 对于触屏设备
css中设置 #ball {touch-action: none}
```


##### 指针捕获
elem.setPointerCapture(pointerId) 将pointerId绑定到elem，调用之后，所有具有相同pointerId的指针事件都将elem作为目标

当pointerup pointercancel事件出现，绑定会被自动地移除
当elem被从文档中移除后， 绑定会自动移除
当elem.releasePointerCaputure(pointerId)被调用


### 键盘事件
keydown/keyup
event.code/ event.key
event.key获取字符， event.code获取物理按键代码
如shift+z:  event.key=Z   event.code=keyZ
z: event.key=z   event.code=keyZ

默认行为在键盘可能各不相同， 如屏幕出现一个字符，字符被删除(delete) 滚动页面(pagedown)...阻止keydown默认行为可以取消大多数行为，但OS中特殊按键无法通过js解决

### 滚动
scroll事件
不能在onscroll监听器中使用event.preventDefault()阻止滚动，它会发生在滚动之后出发。可以在pageUp pageDown的keydown事件上添加阻止滚动


### 表单
文档中表单是特殊集合document.forms的成员
```html
<form name="my">
  <input name="one" value="1">
  <input name="two" value="2">
</form>
<script>
  let form = document.forms.my //获取<form name="my">
  let elem = form.elements.one //获取<input name="one"> 
</script>
```

```html
<form>
  <input type="radio" name="age" value="10">
  <input type="radio" name="age" value="20">
</form>

<script>
  let form = document.fomrs[0]
  let ageElems = form.elements.age
</script>
```

```html
<body>
  <form id="form">
    <fieldset name="userFields">
      <legend>info</legend>
      <input name="login" type="text">
    </fieldset>
  </form>

  <script>
    form.elements.login//<input name="login">
    let fieldset = form.elements.userFields//HTMLFieldSetElemnt
    fieldsset.elements.login === form.elements.login //true
  </script>
</body>
```

反向引用： element.form
```html
<form id="form">
  <input type="text" name="login">
</form>

<script>
  leg login = form.login

  login.form //HTMLFormElement
</script>
```

##### 表单控件
input.value input.checked   textarea.value来访问他们的value

<select> 有三个重要属性：
select.options  <option>子元素集合
select.value   当前所有option的value
select.selectedIndex   当前所选择option的编号

为select设置value的不同方式:
将对应option元素的 option.selected设置为true
将select.value设置为对应的value
将select.selectedIndex设置为对应option编号
```html
<select id="select">
  <option value="apple">Apple</option>
  <option value="pear">Pear</option>
  <option value="banana">Banana</option>
</select>

<script>
  select.options[2].selcted = true
  select.selectedIndex = 2
  select.value = 'Banana'
</script>
```

##### new Option
option = new Option(text, value, defaultSelected, selected)
text: option的文本
value: option的value
defaultSelected: 如果为true，那么selected HTML特性就会被创建
selected： 如果为true， 那么就会被选中
```js
let option = new Option('text', 'value')

let option2 = new Option('text', 'value', true, true)
```
option.selected是否被选择
option.index其所属编号
option.text其文本内容

### 聚焦
focus/blur事件
elem.focus()  elem.blur()
```js
input.onblur = function() {
  if (!this.value.includes('@')) {
    this.classList.add('error')
    input.focus()
  } else [
    this.classList.remove('error')
  ]
}
```

##### tabindex
tab可以不断切换焦点，有些如div等不能被聚焦，但tabindex特性可以让元素被聚焦
```html
<ul>
  <li tabindex="1">One</li>
  <li tabindex="0">Zero</li>
  <li tabindex="2">Two</li>
  <li tabindex="-1">Minus one</li>
</ul>

<style>
  li { cursor: pointer; }
  :focus { outline: 1px dashed green; }
</style>
```

聚焦顺序 1 -》 2 -》 0 
0会被放到最后， -1不能通过tab聚焦

##### 委托
focus/blur不冒泡，所以要么用捕获实现，
要么用
focusin / focusout 他们与focus blur一样，但是会冒泡


### 事件
change input cut copy paste

change: 元素更改完成触发change
```html
<input type='text' onchange="alert('aaa')">
```

```
<select onchange="alert(this.value)">
  <option value="">Select something</option>
  <option value="1">Option 1</option>
  <option value="2">Option 2</option>
  <option value="3">Option 3</option>
</select>
```

input:
输入值更改
```html
<input type="text" id="input"> output: <span id="result"></span>
<script>
  input.oninput = function() {
    result.innerHTML = input.value
  }
</script>
```

cut copy paste
剪切 复制  粘贴
```html
<input type="text" id="input">
<script>
  input.onpaste = function(e) {
    console.log(e.clipboardData.getData('text/plain'))
    e.preventDefault()
  }
  input.oncut = input.oncopy = function(e) {
    console.log(e.type, document.getSelection())
    e.preventDefault()
  }
</script>
```
禁用了三个功能
document.getSelection()得到被选中的文本

### 表单： 事件和方法提交
submit事件， 有两种方式：
1. 点击<input type="submit"> 或<input type="image">
2. 在input字段中按下enter

```html
<form onsubmit="alert('submit!');return false">
  First: Enter in the input field <input type="text" value="text"><br>
  Second: Click "submit": <input type="submit" value="Submit">
</form>
```
因为有return false，所以不会被发送到别处
submit前会触发click事件，哪怕用的是enter


方法 form.submit()
```js
let form = document.createElement('form')
form.action = 'https://google.com/search'
form.method = 'GET'
form.innerHTML = 'aa'
document.body.append(form) //必须添加到文档才能提交
form.submit()
```

## 加载文档和其他资源
### 页面生命周期
DOMContentLoaded: 浏览器已经完全加载HTML， 并构建了DOM树， 但img和样式表可能尚未加载完成
load： img和样式表等外部资源加载完成
beforeunload/unload： 当用户正在离开页面

##### DOMContentLoaded
```js
document.addEventListener('DOMContentLoaded', function() {})
```

```html
<script>
        document.addEventListener('DOMContentLoaded', () => {
            console.log('2')
        })
    </script>
    <script>console.log('1')</script>
```
先1和2， 要等dom构建完成再DOMContentLoaded

外部样式表不影响DOM，DOMContentLoaded不等待它，但是如果script脚本在外部样式表后面，就需要等待


##### window.onload
```js
window.onload = function() {

}
```
全部资源加载完成后执行

##### window.onunload
当访问者离开时，触发。做一些不涉及延迟的操作，如关闭相关的弹出窗口。可以用navigator.sendBeacon(url, data)将数据保存到服务器，且不产生延迟
MDN：navigator.sendBeacon() 方法可用于通过 HTTP POST 将少量数据异步传输到 Web 服务器。

它主要用于将统计数据发送到 Web 服务器，同时避免了用传统技术（如：XMLHttpRequest）发送分析数据的一些问题。

##### window.onbeforeunload
```js
window.onbeforeunload = function() {
  return false;
};
```
关闭页面时进行确认


##### readyState
loading: 文档正在被加载
interactive: 文档被全部读取
complete: 文档被全部读取，所有资源加载完成

```js
function work() {}
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', work)
}
```

readystatechange事件，在状态发生改变时触发
```js
document.addEventListener('readystatechange', () => console.log('a'))
```

事件流
```html
<body>

    <script>
        console.log('readyState' + document.readyState)
        document.addEventListener('readystatechange', () => console.log('readystate'+  document.readyState))
        document.addEventListener('DOMContentLoaded', () => console.log('DOMContentLoaded'))
        window.onload = () => console.log('onload')
    </script>
    <iframe src="iframe.html" onload="console.log('iframe onload')"></iframe>

    <img src="http://en.js.cx/clipart/train.gif" id="img">
    <script>
        img.onload = () => console.log('img onload');
      </script>
</body>
```
loading -> interactive -> DOMContentLoade -> iframe onload -> img onload ->complete -> onload

### 脚本async defer
传统的<script>会按顺序加载，如果放在前面，影响dom加载，但是也不是那么好的解决方案，于是有了如下

##### defer
其加载不阻塞页面，在DOM解析完毕，DOMContentLoaded时间之前执行
```js
<p>...content before scripts...</p>

<script>
  document.addEventListener('DOMContentLoaded', () => alert("DOM ready after defer!"));
</script>

<script defer src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>

<p>...content after scripts...</p>
```

多个defer按相对顺序执行，但是是并行下载

##### async
它不阻塞页面执行，多个async并行下载，谁下载完成谁执行，且与DOMContentLoaded无关系

##### 动态脚本
动态脚本是一步的，按加载顺序执行
```js
let script = document.createElement('script')
script.src = ''
document.body.append(script)
```
script.async = false 则是defer模式

## 资源加载
跟踪外部资源加载：脚本 iframe img
```js
script.onload = () => {}
script.onerror = () => {}
```

跨域
三个级别的跨域访问：
无 crossorigin 特性 —— 禁止访问。
crossorigin="anonymous" —— 如果服务器的响应带有包含 * 或我们的源（origin）的 header Access-Control-Allow-Origin，则允许访问。浏览器不会将授权信息和 cookie 发送到远程服务器。
crossorigin="use-credentials" —— 如果服务器发送回带有我们的源的 header Access-Control-Allow-Origin 和 Access-Control-Allow-Credentials: true，则允许访问。浏览器会将授权信息和 cookie 发送到远程服务器。
 "anonymous"（不会发送 cookie，需要一个服务器端的 header）和 "use-credentials"（会发送 cookie，需要两个服务器端的 header）

 ```html
 <script crossorigin="anonymous"></script>
 ```

## DOM变动观察器
```js
let observer = new MutationObserver(callback)
observer.observe(node, config)

callback = (MutationRecord) => {}
```

```js
let observer = new MutationObserver(mutationRecords => {
            console.log(mutationRecords)
        })
        observer.observe(elem, {
            childList: true, //观察直接子节点
            subtree: true, //及其更低的后代节点
            characterDataOldValue: true//将旧的数据传递给回调
        })
```

例如：
```html
<div contentEditable id="elem">Click and <b>edit</b>, please</div>

<script>
let observer = new MutationObserver(mutationRecords => {
  console.log(mutationRecords); // console.log(the changes)
});

// 观察除了特性之外的所有变动
observer.observe(elem, {
  childList: true, // 观察直接子节点
  subtree: true, // 及其更低的后代节点
  characterDataOldValue: true // 将旧的数据传递给回调
});
</script>
```

修改div，显示
```js
mutationRecords = [{
  type: "characterData",
  oldValue: "edit",
  target: <text node>,
  // 其他属性为空
}];
```

observer.disconnect() 停止观察
observer.takeRecords() 获取尚未处理的变动记录列表，表中记录的是已经发生但回调暂时未处理的变动

## 微任务 宏任务
一个宏任务完成，就去完成微任务，然后再去下一个宏任务


