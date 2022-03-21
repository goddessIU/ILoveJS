# 现代JavaScript教程学习笔记
## 弹窗
```js
window.open('') //打开一个弹窗
```

阻止弹窗
```js
window.open() //阻止
button.onclick = () => window.open() //可以， 弹窗是在用户触发的事件
setTimeout(() => window.open(), 3000) //chrome可以打开弹窗
```

```js
window.open(url, name, params) //url, name: 窗口的name，params： 配置字符串
```

访问弹窗
```js
let newWin = window.open()
newWin.document.write('')
```

从弹窗访问窗口
window.opener访问opener窗口

关闭
win.close()
win.closed属性检查是否被关闭

滚动调整大小,对于弹窗整体而言
win.moveBy()
win.moveTo()
win.resizeBy()
win.resizeTo()
win.onresize事件
对于弹窗内部
win.scrollBy()

win.scrollTo()
elem.scrollIntoView()
win.onscroll()

弹窗聚焦失焦
window.focus()
window.blur()
focus/blur事件


## 跨窗口通信
跨窗口访问受同源限制
不是同源，我们无法访问窗口中的内容，但是可以修改location

iframe.contentWindow  <iframe>的window
iframe.contentDocument  <iframe>的document
```html
<iframe src="https://example.com" id="iframe"></iframe>

<script>
  iframe.onload = function() {
    // 我们可以获取对内部 window 的引用
    let iframeWindow = iframe.contentWindow; // OK
    try {
      // ...但是无法获取其中的文档
      let doc = iframe.contentDocument; // ERROR
    } catch(e) {
      alert(e); // Security Error（另一个源）
    }

    // 并且，我们也无法 **读取** iframe 中页面的 URL
    try {
      // 无法从 location 对象中读取 URL
      let href = iframe.contentWindow.location.href; // ERROR
    } catch(e) {
      alert(e); // Security Error
    }

    // ...我们可以 **写入** location（所以，在 iframe 中加载了其他内容）！
    iframe.contentWindow.location = '/'; // OK

    iframe.onload = null; // 清空处理程序，在 location 更改后不要再运行它
  };
</script>
```

```html
<script>
  let oldDoc = iframe.contentDocument; //错误的
  iframe.onload = function() {
    let newDoc = iframe.contentDocument;
    // 加载的文档与初始的文档不同！
    alert(oldDoc == newDoc); // false
  };
</script>
```

所以为了更早获取document可以这样
```js
let oldDoc = iframe.contentDocument
  let timer = setInterval(() => {
    let newDoc = iframe.contentDocument
    if (newDoc === oldDoc) return;
    clearInterval(timer)
  }, 100)
```

window.frames获取iframe的window对象
如
window.frames[0]
window.frames.iframeName (name='iframeName'的iframe)

sandbox特性允许在iframe中禁止某些特定行为，也可以通过选项放松限制

##### 跨窗口通信
```js
win.postMessage(data, targetOrigin) //发给win
//data数据， targetOrigin指定目标窗口的源
```

接收，message事件
```js
window.addEventListener('message', function(e) {
    e.data //数据
    e.origin //发送方的源
    e.source//对发送方窗口引用，可以source.postMessage()回信
})
```

## 点击劫持攻击
就是我们在一个点击链接上放一个z-index更高的透明frame，然后用户点了，就中招了
```html
<style>
iframe { /* 来自受害网站的 iframe */
  width: 400px;
  height: 100px;
  position: absolute;
  top:0; left:-20px;
  opacity: 0.5; /* 在实际中为 opacity:0 */
  z-index: 1;
}
</style>

<div>点击即可变得富有：</div>

<!-- 来自受害网站的 url -->
<iframe src="/clickjacking/facebook.html"></iframe>

<button>点这里！</button>

<div>……你很酷（我实际上是一名帅气的黑客）！</div>
```

##### 防御：
传统防御
```js
if (top !== window) {
    top.location = window.location
}
如果window不在顶部，就将他放置在顶部
```
window.frames —— “子”窗口的集合（用于嵌套的 iframe）。
window.parent —— 对“父”（外部）窗口的引用。
window.top —— 对最顶级父窗口的引用。

但是，会被绕过
阻止顶级导航
```js
window.onbeforeunload = function() {
    return false
}
```
这样询问用户是否离开页面，很多情况下用户不会离开

sandbox特性，阻止更改top.location

更好的方法
X-Frame-Options
服务端header X-Frame-Options可以允许或禁止在frame中显示页面

显示禁用的功能， 设一个样式width100% height100%的 <div>覆盖页面，拦截点击，如果window==top则移除div
```html
<style>
  #protector {
    height: 100%;
    width: 100%;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 99999999;
  }
</style>

<div id="protector">
  <a href="/" target="_blank">前往网站</a>
</div>

<script>
  // 如果顶级窗口来自其他源，这里则会出现一个 error
  // 但是在本例中没有问题
  if (top.document.domain == document.domain) {
    protector.remove();
  }
</script>
```


samesite cookie特性 具有samesite特性的cookie仅在网站是通过直接方式打开情况下才发送到网站


## 浏览器数据存储
### cookie
cookie常用在登录，登录后，服务器设置cookie，下次请求戴上cookie就知道是谁了

```js
console.log(ducument.cookie)//显示cookie
document.cooke = ''//写入，但是只是添加，不是替换
cookie格式 name=value; age=23;
```


path
path=/admin 该cookie在/admin /admin/some下可见， /home不行

domain
可以访问cookie的域 在site.com下的cookie， 在other.com下无法获取， 在子域forum.site.com也无法获取

expires
采用GMT格式
```js
let date = new Date(Date.now() + 86400e3)
date = date.toUTCstring()
document.cookie = "user=John; expires" + date
```
如果为过去的时间，cookie删除


max-age
```js
document.cookie = "user=John; max-age=3600" //一小时后失效，距离现在时间的秒数，零或负数删除
```

secure
```js
document.cookie = "user=John; secure" //只能HTTPS下访问
```

samesite
避免XSRF攻击
一个网站具有与另一个网站相关的请求，就会得到另一个网站的cookie，容易造成危险
samesite=strict
来自同一网站之外，cookie永远不发送
samesite=lax
宽松模式，在HTTP方法是安全的且操作执行与顶级导航（不是frame)，那么就可以使用lax，雨荨懈怠cookie

httpOnly
js不能获取cookie，也不能修改；因为黑客可以把js代码诸如网页,等待用户访问页面发起攻击，这个选项可以预防。如果网站有bug给了黑科技会，黑客就可能通过js代码得到cookie就会危险

GDRP
就是很多外国网站的询问，就是他们的一种法案，必须询问用户，才能跟踪用户cookie  如果我们要设置带有身份验证会话（session）或跟踪 id 的 cookie，那么必须得到用户的允许。

### LocalStorage, SessionStorage
setItem(key, value)
getItem(key)
removeItem(key)
clear()
key(index) //获取该索引下的键名
length //存储内容的长度
localStroage在同源页面 窗口共享

遍历
```js
for (let i = 0; i< localStorage.length; i++) {
    let key = localStorage.key(i)
}
```

sessionStorage 只存在当前标签页，刷新还有，关闭无

##### storage事件
以上两个发生更新，触发事件

key —— 发生更改的数据的 key（如果调用的是 .clear() 方法，则为 null）。
oldValue —— 旧值（如果是新增数据，则为 null）。
newValue —— 新值（如果是删除数据，则为 null）。
url —— 发生数据更新的文档的 url。
storageArea —— 发生数据更新的 localStorage 或 sessionStorage 对象。

## 网络请求
fetch可以发送请求，和axios差不多(不知道axios源码是怎样)

```js
let promise = fetch(url, [options])


let response = await fetch(url)
if (response.ok) { //status为200---299， 就是true
    let json = await response.json() //以json方式访问response body
} else {
    console.log(response.status) //status是http状态码， 例如200
}
```

访问body方法
response.text()  json()  formData() blob() arrayBuffer()

Response header 位于response.headers中一个类似map的header对象
```js
response.headers.get('Content-Type')
```

Request header 在fetch中设置
```js
let response = fetch(url, {
    headers: {
        Authentication: 'secret'
    }
})
```

POST请求
```js
let response = await fecth(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json;charset=utf-8'
    },  
    body: JSON.stringfy(obj)
})
```

### FormData
FormData对象表示HTML表单数据
```js
let formData  = new FormData([form])
```
```html
<form id="formElem">
  <input type="text" name="name" value="John">
  <input type="text" name="surname" value="Smith">
  <input type="submit">
</form>

<script>
  formElem.onsubmit = async (e) => {
      e.preventDefault()
      let response = await fetch(url, {
          method: 'POST',
          body: new FormData(formElem)
      })
      let res = await response.json()

  }
</script>
```

方法
formData.append(name, value)
formData.append(name, blob, fileName)  就是<input type="file">类型
formData.delete(name)
formData.get(name)
formData.has(name)

formData.set(name, value) 删去原有name的字段

##### 跟踪下载进度
response.body属性可以帮助  它是 ReadableStream —— 一个特殊的对象，它可以逐块（chunk）提供 body。
```js
// Step 1：启动 fetch，并获得一个 reader
let response = await fetch('https://api.github.com/repos/javascript-tutorial/en.javascript.info/commits?per_page=100');

const reader = response.body.getReader();

// Step 2：获得总长度（length）
const contentLength = +response.headers.get('Content-Length');

// Step 3：读取数据
let receivedLength = 0; // 当前接收到了这么多字节
let chunks = []; // 接收到的二进制块的数组（包括 body）
while(true) {
  const {done, value} = await reader.read();

  if (done) {
    break;
  }

  chunks.push(value);
  receivedLength += value.length;

  console.log(`Received ${receivedLength} of ${contentLength}`)
}

// Step 4：将块连接到单个 Uint8Array
let chunksAll = new Uint8Array(receivedLength); // (4.1)
let position = 0;
for(let chunk of chunks) {
  chunksAll.set(chunk, position); // (4.2)
  position += chunk.length;
}

// Step 5：解码成字符串
let result = new TextDecoder("utf-8").decode(chunksAll);

// 我们完成啦！
let commits = JSON.parse(result);
alert(commits[0].author.login);
```

### 终止fetch
内建对象AbortController
```js
let controller = new AbortController()
```
控制器有方法abort(), 属性signal，在它上面设置监听器
abort()调用时，signal触发abort事件， signal.aborted为true

所以现在signal上设置监听器，再调用abort()
```js
let controller = new AbortController()
let signal = controller.signal

signal.addEventListener('abort', () => console.log('abort'))

controller.abort()
console.log(signal.aborted)
```

fetch很好的和AbortController工作
```js
let controller = new AbortController()
setTimeout(() => controller.abort(), 1000)
try {
    let response = await fetch(url, {
        signal: controller.signal
    }) 
} catch(err) {
        
}
```
终止会reject promise

### Fetch API
```js
let promise = fetch(url, {
  method: "GET", // POST，PUT，DELETE，等。
  headers: {
    // 内容类型 header 值通常是自动设置的
    // 取决于 request body
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined // string，FormData，Blob，BufferSource，或 URLSearchParams
  referrer: "about:client", // 或 "" 以不发送 Referer header，
  // 或者是当前源的 url
  referrerPolicy: "no-referrer-when-downgrade", // no-referrer，origin，same-origin...
  mode: "cors", // same-origin，no-cors
  credentials: "same-origin", // omit，include
  cache: "default", // no-store，reload，no-cache，force-cache，或 only-if-cached
  redirect: "follow", // manual，error
  integrity: "", // 一个 hash，像 "sha256-abcdef1234567890"
  keepalive: false, // true
  signal: undefined, // AbortController 来中止请求
  window: window // null
});
```

mode
cors : 允许跨域
same-origin: 不允许
no-cors: 简单跨域请求

credentials
是否应该随请求发送cookie和HTTP-Authorization header
same-origin 对于跨域不发送
include 总是发送，需要来自跨源服务器的 Accept-Control-Allow-Credentials，才能使 JavaScript 能够访问响应
omit: 不发送，即使同源

### CORS
两种跨域请求
简单请求 和 其他请求
简单请求： 
1. 方法是GET POST HEAD
2. 简单的header 仅允许自定义下列 header：
Accept，
Accept-Language，
Content-Language，
Content-Type 的值为 application/x-www-form-urlencoded，multipart/form-data 或 text/plain。

当我们尝试发送一个非简单请求时，浏览器会发送一个特殊的“预检（preflight）”请求到服务器 —— 询问服务器，你接受此类跨源请求吗？

并且，除非服务器明确通过 header 进行确认，否则非简单请求不会被发送。


跨域请求会添加 Origin header，包含确切的源，服务器可以检查 Origin，如果同意接受这样的请求，就会在响应中添加一个特殊的 header Access-Control-Allow-Origin该 header 包含了允许的源（在我们的示例中是 https://javascript.info），或者一个星号 *。然后响应成功，否则报错。

于跨源请求，默认情况下，JavaScript 只能访问“简单” response header：

Cache-Control
Content-Language
Content-Type
Expires
Last-Modified
Pragma

要授予 JavaScript 对任何其他 response header 的访问权限，服务器必须发送 Access-Control-Expose-Headers header。它包含一个以逗号分隔的应该被设置为可访问的非简单 header 名称列表。
如
```js
Access-Control-Expose-Headers: Content-Length,API-Key
```

对于非简单请求
它发送这类请求前，会先发送“预检（preflight）”请求来请求许可。预检请求使用 OPTIONS 方法，它没有 body，但是有两个 header：
Access-Control-Request-Method header 带有非简单请求的方法。
Access-Control-Request-Headers header 提供一个以逗号分隔的非简单 HTTP-header 列表。

如果服务器同意处理请求，那么它会进行响应，此响应的状态码应该为 200，没有 body，具有 header：

Access-Control-Allow-Origin 必须为 * 或进行请求的源（例如 https://javascript.info）才能允许此请求。
Access-Control-Allow-Methods 必须具有允许的方法。
Access-Control-Allow-Headers 必须具有一个允许的 header 列表。
另外，header Access-Control-Max-Age 可以指定缓存此权限的秒数。因此，浏览器不是必须为满足给定权限的后续请求发送预检。

先预检， 然后服务器预检响应，然后实际请求，然后实际响应

js跨域请求不带任何平局（cookies 或者 HTTP authentication),如果带，需要
```js
fetch(url, {
    credentials: "include"
})
```

fetch 将把 cookie 和我们的请求发送到该网站。

如果服务器同意接受 带有凭据 的请求，则除了 Access-Control-Allow-Origin 外，服务器还应该在响应中添加 header Access-Control-Allow-Credentials: true。对于具有凭据的请求，禁止 Access-Control-Allow-Origin 使用星号 *。如上所示，它必须有一个确切的源

## URL对象
用于创建和解析URL
```js
let url = new URL(url, [base])
url: 完整的URL， 或者仅路径
base: 如果url只有路径，则根据base生成url

let url1 = new URL('https://javascript.info/profile/admin');
let url2 = new URL('/profile/admin', 'https://javascript.info');等价
```

```js
let url = new URL('https://javascript.info/profile/admin');
let newUrl = new URL('tester', url);

alert(newUrl); // https://javascript.info/profile/tester

let url = new URL('https://javascript.info/url');
alert(url.protocol); // https:
alert(url.host);     // javascript.info
alert(url.pathname); // /url
```

url.searchParams方法
append(name, value) —— 按照 name 添加参数，
delete(name) —— 按照 name 移除参数，
get(name) —— 按照 name 获取参数，
getAll(name) —— 获取相同 name 的所有参数（这是可行的，例如 ?user=John&user=Pete），
has(name) —— 按照 name 检查参数是否存在，
set(name, value) —— set/replace 参数，
sort() —— 按 name 对参数进行排序，很少使用，
……并且它是可迭代的，类似于 Map。

URL对象会自动编码

如果使用字符串，则需要手动编码/解码特殊字符。

下面是用于编码/解码 URL 的内建函数：

encodeURI —— 编码整个 URL。
decodeURI —— 解码为编码前的状态。
encodeURIComponent —— 编码 URL 组件，例如搜索参数，或者 hash，或者 pathname。
decodeURIComponent —— 解码为编码前的状态。

### XMLHttpRequest
fetch 使得其某种程度被弃用
```js
let xhr = new XMLHttpRequest()
xhr.open(method, URL, [async, user, password]) //初始化
xhr.send([body]) //发送请求

//监听响应
xhr.onload = function() {
    //请求完成， HTTp状态400或500， 并且响应已完全下载
}

xhr.oneeror = function() {
    //无法发出请求
}

xhr.onprogress = function() {
   //定期触发，报告下载多少
}
```

xhr属性:
status: HTTP状态码， 200， 404
statusText: HTTP状态信息， 200为OK， 404为Not found
response: response body
xhr.timeout = 1000//超时

xhr.responseType设置响应格式
```js
xhr.responseType = 'json';
```

xhr.readyState 来了解当前状态。
```js
UNSENT = 0; // 初始状态
OPENED = 1; // open 被调用
HEADERS_RECEIVED = 2; // 接收到 response header
LOADING = 3; // 响应正在被加载（接收到一个数据包）
DONE = 4; // 请求完成
```

readystatechange 事件来跟踪它们：
```js
xhr.onreadystatechange = function() {
    if (xhr.readyState === 3) {

    }
    if (xhr.readyState === 4) {
        
    }
}
```

xhr.abort()终止请求，xhr.status=0


async参数为false是同步请求，js暂停等待完成后继续

自定义header
```js
setRequestHeader(name, value)
xhr.setRequestHeader('Content-Type', 'application/json')

getResponseHeader(name)
获取具有给定 name 的 header（Set-Cookie 和 Set-Cookie2 除外）
xhr.getResponseHeader('Content-Type')

getAllResponseHeaders()
返回除 Set-Cookie 和 Set-Cookie2 外的所有 response header。
```

关于POST
xhr.open('POST', ...) —— 使用 POST 方法。
xhr.send(formData) 将表单发送到服务器。
```js
let xhr = new XMLHttpRequest()
let json = JSON.stringify({
    name: "John",
    surname: "Smith"
  });

  xhr.open("POST", '/submit')
  xhr.setRequestHeader('Content-Type', 'application/json')
  xhr.send(json)
```

这里有另一个对象，它没有方法，它专门用于跟踪上传事件：xhr.upload。

它会生成事件，类似于 xhr，但是 xhr.upload 仅在上传时触发它们：

loadstart —— 上传开始。
progress —— 上传期间定期触发。
abort —— 上传中止。
error —— 非 HTTP 错误。
load —— 上传成功完成。
timeout —— 上传超时（如果设置了 timeout 属性）。
loadend —— 上传完成，无论成功还是 error。

跨域
和fetch策略一样
xhr.withCredentials = true//可以携带cookies