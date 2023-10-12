# Axios axios.js源码分析注释

## 目录

- [Axios axios.js源码分析注释](#axios-axiosjs源码分析注释)
  - [目录](#目录)
    - [4.1.2 第二部分](#412-第二部分)
    - [4.1.3 工具方法之 bind](#413-工具方法之-bind)
    - [4.1.4 工具方法之 utils.extend](#414-工具方法之-utilsextend)
    - [4.1.5 工具方法之 utils.forEach](#415-工具方法之-utilsforeach)
    - [4.1.6 第三部分](#416-第三部分)
    - [4.2 核心构造函数 Axios](#42-核心构造函数-axios)
    - [4.3 拦截器管理构造函数 InterceptorManager](#43-拦截器管理构造函数-interceptormanager)
      - [4.3.1 InterceptorManager.prototype.use 使用](#431-interceptormanagerprototypeuse-使用)
      - [4.3.2 InterceptorManager.prototype.eject 移除](#432-interceptormanagerprototypeeject-移除)
      - [4.3.3 InterceptorManager.prototype.forEach 遍历](#433-interceptormanagerprototypeforeach-遍历)
  - [5. 实例结合](#5-实例结合)
    - [5.1 先看调用栈流程](#51-先看调用栈流程)
    - [5.2 Axios.prototype.request 请求核心方法](#52-axiosprototyperequest-请求核心方法)
      - [5.2.1 组成`Promise`链，返回`Promise`实例](#521-组成promise链返回promise实例)
    - [5.3 dispatchRequest 最终派发请求](#53-dispatchrequest-最终派发请求)
      - [5.3.1 dispatchRequest 之 transformData 转换数据](#531-dispatchrequest-之-transformdata-转换数据)
      - [5.3.2 dispatchRequest 之 adapter 适配器执行部分](#532-dispatchrequest-之-adapter-适配器执行部分)
    - [5.4 adapter 适配器 真正发送请求](#54-adapter-适配器-真正发送请求)
    - [5.5 dispatchRequest 之 取消模块](#55-dispatchrequest-之-取消模块)
      - [5.5.1 取消请求模块代码示例](#551-取消请求模块代码示例)
      - [5.5.2 接下来看取消模块的源码](#552-接下来看取消模块的源码)

引入一些工具函数`utils`、`Axios`构造函数、默认配置`defaults`等。

```react&#x20;tsx
// 第一部分： 
// lib/axios 
// 严格模式 'use strict'; 
// 引入 utils 对象，有很多工具方法。 
var utils = require('./utils'); 
// 引入 bind 方法
var bind = require('./helpers/bind');
// 核心构造函数 
Axios var Axios = require('./core/Axios'); 
// 合并配置方法 
var mergeConfig = require('./core/mergeConfig'); 
// 引入默认配置 
var defaults = require('./defaults'); 
```

#### 4.1.2 第二部分

是生成实例对象 `axios`、`axios.Axios`、`axios.create`等。

```react&#x20;tsx

- Create an instance of Axios

\*

- @param {Object} defaultConfig The default config for the instance
- @return {Axios} A new instance of Axios

\*/

function createInstance(defaultConfig) {

// new 一个 Axios 生成实例对象

var context = new Axios(defaultConfig);

// bind 返回一个新的 wrap 函数，

// 也就是为什么调用 axios 是调用 Axios.prototype.request 函数的原因

var instance = bind(Axios.prototype.request, context);

// Copy axios.prototype to instance

// 复制 Axios.prototype 到实例上。

// 也就是为什么 有 axios.get 等别名方法，

// 且调用的是 Axios.prototype.get 等别名方法。

utils.extend(instance, Axios.prototype, context);

// Copy context to instance

// 复制 context 到 intance 实例

// 也就是为什么默认配置 axios.defaults 和拦截器  axios.interceptors 可以使用的原因

// 其实是new Axios().defaults 和 new Axios().interceptors

utils.extend(instance, context);

// 最后返回实例对象，以上代码，在上文的图中都有体现。这时可以仔细看下上图。

return instance;

}

// Create the default instance to be exported

// 导出 创建默认实例

var axios = createInstance(defaults);

// Expose Axios class to allow class inheritance

// 暴露 Axios class 允许 class 继承 也就是可以 new axios.Axios()

// 但  axios 文档中 并没有提到这个，我们平时也用得少。

axios.Axios = Axios;

// Factory for creating new instances

// 工厂模式 创建新的实例 用户可以自定义一些参数

axios.create = function create(instanceConfig) {

return createInstance(mergeConfig(axios.defaults, instanceConfig));

}
```

这里简述下工厂模式。`axios.create`，也就是用户不需要知道内部是怎么实现的。举个生活的例子，我们买手机，不需要知道手机是怎么做的，就是工厂模式。看完第二部分，里面涉及几个工具函数，如`bind`、`extend`。接下来讲述这几个工具方法。 &#x20;

#### 4.1.3 工具方法之 bind

`axios/lib/helpers/bind.js`

js

复制代码

\`'use strict';

// 返回一个新的函数 wrap

module.exports = function bind(fn, thisArg) {

return function wrap() {

var args = new Array(arguments.length);

for (var i = 0; i < args.length; i++) {

args\[i] = arguments\[i];

}

// 把 argument 对象放在数组 args 里

return fn.apply(thisArg, args);

};

};\`&#x20;

传递两个参数函数和`thisArg`指向。把参数`arguments`生成数组，最后调用返回参数结构。其实现在 `apply` 支持 `arguments`这样的类数组对象了，不需要手动转数组。

[面试官问：能否模拟实现JS的bind方法](https://juejin.cn/post/6844903718089916429 "面试官问：能否模拟实现JS的bind方法") &#x20;

\`function fn(){

console.log.apply(console, arguments);

}

fn(1,2,3,4,5,6, '若川');

// 1 2 3 4 5 6 '若川'\`&#x20;

#### 4.1.4 工具方法之 utils.extend

`axios/lib/utils.js`

js

复制代码

\`function extend(a, b, thisArg) {

forEach(b, function assignValue(val, key) {

if (thisArg && typeof val === 'function') {

a\[key] = bind(val, thisArg);

} else {

a\[key] = val;

}

});

return a;

}\`&#x20;

其实就是遍历参数 `b` 对象，复制到 `a` 对象上，如果是函数就是则用 `bind` 调用。

#### 4.1.5 工具方法之 utils.forEach

`axios/lib/utils.js`

遍历数组和对象。设计模式称之为迭代器模式。很多源码都有类似这样的遍历函数。比如大家熟知的`jQuery` `$.each`。

js

复制代码

\`/\*\*

- @param {Object|Array} obj The object to iterate
- @param {Function} fn The callback to invoke for each item

\*/

function forEach(obj, fn) {

// Don't bother if no value provided

// 判断 null 和 undefined 直接返回

if (obj === null || typeof obj === 'undefined') {

return;

}

// Force an array if not already something iterable

// 如果不是对象，放在数组里。

if (typeof obj !== 'object') {

/*eslint no-param-reassign:0*/

obj = \[obj];

}

// 是数组 则用for 循环，调用 fn 函数。参数类似 Array.prototype.forEach 的前三个参数。

if (isArray(obj)) {

// Iterate over array values

for (var i = 0, l = obj.length; i < l; i++) {

fn.call(null, obj\[i], i, obj);

}

} else {

// Iterate over object keys

// 用 for in 遍历对象，但 for in 会遍历原型链上可遍历的属性。

// 所以用 hasOwnProperty 来过滤自身属性了。

// 其实也可以用Object.keys来遍历，它不遍历原型链上可遍历的属性。

for (var key in obj) {

if (Object.prototype.hasOwnProperty.call(obj, key)) {

fn.call(null, obj\[key], key, obj);

}

}

}

}\`&#x20;

如果对`Object`相关的`API`不熟悉，[JavaScript 对象所有API解析](https://link.juejin.cn?target=https://www.lxchuan12.cn/js-object-api/ "JavaScript 对象所有API解析")

#### 4.1.6 第三部分

取消相关API实现，还有`all`、`spread`、导出等实现。

js

复制代码

\`// Expose Cancel & CancelToken

// 导出 Cancel 和 CancelToken

axios.Cancel = require('./cancel/Cancel');

axios.CancelToken = require('./cancel/CancelToken');

axios.isCancel = require('./cancel/isCancel');

// Expose all/spread

// 导出 all 和 spread API

axios.all = function all(promises) {

return Promise.all(promises);

};

axios.spread = require('./helpers/spread');

module.exports = axios;

// Allow use of default import syntax in TypeScript

// 也就是可以以下方式引入

// import axios from 'axios';

module.exports.default = axios;\`&#x20;

这里介绍下 `spread`，取消的`API`暂时不做分析，后文再详细分析。

假设你有这样的需求。

js

复制代码

`function f(x, y, z) {} var args = [1, 2, 3]; f.apply(null, args);`&#x20;

那么可以用`spread`方法。用法：

js

复制代码

`axios.spread(function(x, y, z) {})([1, 2, 3]);`&#x20;

实现也比较简单。源码实现：

js

复制代码

\`/\*\*

- @param {Function} callback
- @returns {Function}

\*/

module.exports = function spread(callback) {

return function wrap(arr) {

return callback.apply(null, arr);

};

};\`&#x20;

上文`var context = new Axios(defaultConfig);`，接下来介绍核心构造函数`Axios`。

### 4.2 核心构造函数 Axios

`axios/lib/core/Axios.js`

构造函数`Axios`。

js

复制代码

\`function Axios(instanceConfig) {

// 默认参数

this.defaults = instanceConfig;

// 拦截器 请求和响应拦截器

this.interceptors = {

request: new InterceptorManager(),

response: new InterceptorManager()

};

}\`&#x20;

js

复制代码

\`Axios.prototype.request = function(config){

// 省略，这个是核心方法，后文结合例子详细描述

// code ...

var promise = Promise.resolve(config);

// code ...

return promise;

}

// 这是获取 Uri 的函数，这里省略

Axios.prototype.getUri = function(){}

// 提供一些请求方法的别名

// Provide aliases for supported request methods

// 遍历执行

// 也就是为啥我们可以 axios.get 等别名的方式调用，而且调用的是 Axios.prototype.request 方法

// 这个也在上面的 axios 结构图上有所体现。

utils.forEach(\['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {

/*eslint func-names:0*/

Axios.prototype\[method] = function(url, config) {

return this.request(utils.merge(config || {}, {

method: method,

url: url

}));

};

});

utils.forEach(\['post', 'put', 'patch'], function forEachMethodWithData(method) {

/*eslint func-names:0*/

Axios.prototype\[method] = function(url, data, config) {

return this.request(utils.merge(config || {}, {

method: method,

url: url,

data: data

}));

};

});

module.exports = Axios;\`&#x20;

接下来看拦截器部分。

### 4.3 拦截器管理构造函数 InterceptorManager

请求前拦截，和请求后拦截。在`Axios.prototype.request`函数里使用，具体怎么实现的拦截的，后文配合例子详细讲述。 &#x20;

[axios github 仓库 拦截器文档](https://link.juejin.cn?target=https://github.com/axios/axios#interceptors "axios github 仓库 拦截器文档")

如何使用：

js

复制代码

\`// Add a request interceptor

// 添加请求前拦截器

axios.interceptors.request.use(function (config) {

// Do something before request is sent

return config;

}, function (error) {

// Do something with request error

return Promise.reject(error);

});

// Add a response interceptor

// 添加请求后拦截器

axios.interceptors.response.use(function (response) {

// Any status code that lie within the range of 2xx cause this function to trigger

// Do something with response data

return response;

}, function (error) {

// Any status codes that falls outside the range of 2xx cause this function to trigger

// Do something with response error

return Promise.reject(error);

});\`&#x20;

如果想把拦截器，可以用`eject`方法。

js

复制代码

`const myInterceptor = axios.interceptors.request.use(function () {/*...*/}); axios.interceptors.request.eject(myInterceptor);`&#x20;

拦截器也可以添加自定义的实例上。

js

复制代码

`const instance = axios.create(); instance.interceptors.request.use(function () {/*...*/});`&#x20;

源码实现：

构造函数，`handles` 用于存储拦截器函数。

js

复制代码

\`function InterceptorManager() {

this.handlers = \[];

}\`&#x20;

接下来声明了三个方法：使用、移除、遍历。

#### 4.3.1 InterceptorManager.prototype.use 使用

传递两个函数作为参数，数组中的一项存储的是`{fulfilled: function(){}, rejected: function(){}}`。返回数字 `ID`，用于移除拦截器。

js

复制代码

\`\`/\*\*

- @param {Function} fulfilled The function to handle `then` for a `Promise`
- @param {Function} rejected The function to handle `reject` for a `Promise`

\*

- @return {Number} 返回ID 是为了用 eject 移除

\*/

InterceptorManager.prototype.use = function use(fulfilled, rejected) {

this.handlers.push({

fulfilled: fulfilled,

rejected: rejected

});

return this.handlers.length - 1;

};\`\`&#x20;

#### 4.3.2 InterceptorManager.prototype.eject 移除

根据 `use` 返回的 `ID` 移除 拦截器。

js

复制代码

\`\`/\*\*

- @param {Number} id The ID that was returned by `use`

\*/

InterceptorManager.prototype.eject = function eject(id) {

if (this.handlers\[id]) {

this.handlers\[id] = null;

}

};\`\`&#x20;

有点类似定时器`setTimeout` 和 `setInterval`，返回值是`id`。用`clearTimeout` 和`clearInterval`来清除定时器。

js

复制代码

\`// 提一下 定时器回调函数是可以传参的，返回值 timer 是数字

var timer = setInterval((name) => {

console.log(name);

}, 1000, '若川');

console.log(timer); // 数字 ID

// 在控制台等会再输入执行这句，定时器就被清除了

clearInterval(timer);\`&#x20;

#### 4.3.3 InterceptorManager.prototype.forEach 遍历

遍历执行所有拦截器，传递一个回调函数（每一个拦截器函数作为参数）调用，被移除的一项是`null`，所以不会执行，也就达到了移除的效果。

js

复制代码

\`/\*\*

- @param {Function} fn The function to call for each interceptor

\*/

InterceptorManager.prototype.forEach = function forEach(fn) {

utils.forEach(this.handlers, function forEachHandler(h) {

if (h !== null) {

fn(h);

}

});

};\`&#x20;

## 5. 实例结合

上文叙述的调试时运行`npm start` 是用`axios/sandbox/client.html`路径的文件作为示例的，可以自行调试。

以下是一段这个文件中的代码。

js

复制代码

\`axios(options)

.then(function (res) {

response.innerHTML = JSON.stringify(res.data, null, 2);

})

.catch(function (res) {

response.innerHTML = JSON.stringify(res.data, null, 2);

});\`&#x20;

### 5.1 先看调用栈流程

如果不想一步步调试，有个偷巧的方法。知道 `axios` 使用了`XMLHttpRequest`。可以在项目中搜索：`new XMLHttpRequest`。定位到文件 `axios/lib/adapters/xhr.js`在这条语句 `var request = new XMLHttpRequest();chrome` 浏览器中 打个断点调试下，再根据调用栈来细看具体函数等实现。 &#x20;

`Call Stack`

bash

复制代码

`dispatchXhrRequest (xhr.js:19) xhrAdapter (xhr.js:12) dispatchRequest (dispatchRequest.js:60) Promise.then (async) request (Axios.js:54) wrap (bind.js:10) submit.onclick ((index):138)`&#x20;

简述下流程： &#x20;

1. `Send Request` 按钮点击 `submit.onclick` &#x20;
2. 调用 `axios` 函数实际上是调用 `Axios.prototype.request` 函数，而这个函数使用 `bind` 返回的一个名为`wrap`的函数。 &#x20;
3. 调用 `Axios.prototype.request` &#x20;
4. （有请求拦截器的情况下执行请求拦截器），中间会执行 `dispatchRequest`方法 &#x20;
5. `dispatchRequest` 之后调用 `adapter (xhrAdapter)` &#x20;
6. 最后调用 `Promise` 中的函数`dispatchXhrRequest`，（有响应拦截器的情况下最后会再调用响应拦截器） &#x20;

如果仔细看了文章开始的`axios 结构关系图`，其实对这个流程也有大概的了解。

接下来看 `Axios.prototype.request` 具体实现。

### 5.2 Axios.prototype.request 请求核心方法

这个函数是核心函数。 主要做了这几件事：

> 1.判断第一个参数是字符串，则设置 url,也就是支持`axios('example/url', [, config])`，也支持`axios({})`。2.合并默认参数和用户传递的参数3.设置请求的方法，默认是是`get`方法4.将用户设置的请求和响应拦截器、发送请求的`dispatchRequest`组成`Promise`链，最后返回还是`Promise`实例。也就是保证了请求前拦截器先执行，然后发送请求，再响应拦截器执行这样的顺序。也就是为啥最后还是可以`then`，`catch`方法的缘故。 &#x20;

js

复制代码

\`\`Axios.prototype.request = function request(config) {

/*eslint no-param-reassign:0*/

// Allow for axios('example/url'\[, config]) a la fetch API

// 这一段代码 其实就是 使 axios('example/url', \[, config])

// config 参数可以省略

if (typeof config === 'string') {

config = arguments\[1] || {};

config.url = arguments\[0];

} else {

config = config || {};

}

// 合并默认参数和用户传递的参数

config = mergeConfig(this.defaults, config);

// Set config.method

// 设置 请求方法，默认 get 。

if (config.method) {

config.method = config.method.toLowerCase();

} else if (this.defaults.method) {

config.method = this.defaults.method.toLowerCase();

} else {

config.method = 'get';

}

// Hook up interceptors middleware

// 组成`Promise`链 这段拆开到后文再讲述

};\`\`&#x20;

#### 5.2.1 组成`Promise`链，返回`Promise`实例

> 这部分：用户设置的请求和响应拦截器、发送请求的`dispatchRequest`组成`Promise`链。也就是保证了请求前拦截器先执行，然后发送请求，再响应拦截器执行这样的顺序也就是保证了请求前拦截器先执行，然后发送请求，再响应拦截器执行这样的顺序也就是为啥最后还是可以`then`，`catch`方法的缘故。 &#x20;

&#x20;[阮一峰老师 的 ES6 Promise-resolve](https://link.juejin.cn?target=http://es6.ruanyifeng.com/#docs/promise#Promise-resolve "阮一峰老师 的 ES6 Promise-resolve") 和 [JavaScript Promise迷你书（中文版）](https://link.juejin.cn?target=http://liubin.org/promises-book/ "JavaScript Promise迷你书（中文版）")

js

复制代码

\`\` // 组成`Promise`链

// Hook up interceptors middleware

// 把 xhr 请求 的 dispatchRequest 和 undefined 放在一个数组里

var chain = \[dispatchRequest, undefined];

// 创建 Promise 实例

var promise = Promise.resolve(config);

// 遍历用户设置的请求拦截器 放到数组的 chain 前面

this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {

chain.unshift(interceptor.fulfilled, interceptor.rejected);

});

// 遍历用户设置的响应拦截器 放到数组的 chain 后面

this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {

chain.push(interceptor.fulfilled, interceptor.rejected);

});

// 遍历 chain 数组，直到遍历 chain.length 为 0

while (chain.length) {

// 两两对应移出来 放到 then 的两个参数里。

promise = promise.then(chain.shift(), chain.shift());

}

return promise;\`\`&#x20;

js

复制代码

`var promise = Promise.resolve(config);`&#x20;

解释下这句。作用是生成`Promise`实例。

js

复制代码

\`var promise = Promise.resolve({name: '若川'})

// 等价于

// new Promise(resolve => resolve({name: '若川'}))

promise.then(function (config){

console.log(config)

});

// {name: "若川"}\`&#x20;

同样解释下后文会出现的`Promise.reject(error);`：

js

复制代码

`Promise.reject(error);`&#x20;

js

复制代码

\`var promise = Promise.reject({name: '若川'})

// 等价于

// new Promise(reject => reject({name: '若川'}))

// promise.then(null, function (config){

//   console.log(config)

// });

// 等价于

promise.catch(function (config){

console.log(config)

});

// {name: "若川"}\`&#x20;

接下来结合例子，来理解这段代码。很遗憾，在`example`文件夹没有拦截器的例子。在`example`中在`example/get`的基础上添加了一个拦截器的示例。`axios/examples/interceptors`，便于调试。

bash

复制代码

`node ./examples/server.js -p 5000`&#x20;

`promise = promise.then(chain.shift(), chain.shift());`这段代码打个断点。

会得到这样的这张图。&#x20;

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/13/16efe5beaa35ac54~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

特别关注下，右侧，`local`中的`chain`数组。也就是这样的结构。

js

复制代码

\`var chain = \[

'请求成功拦截2', '请求失败拦截2',&#x20;

'请求成功拦截1', '请求失败拦截1',&#x20;

dispatch,  undefined,

'响应成功拦截1', '响应失败拦截1',

'响应成功拦截2', '响应失败拦截2',

]\`&#x20;

这段代码相对比较绕。也就是会生成如下类似的代码，中间会调用`dispatchRequest`方法。

js

复制代码

`// config 是 用户配置和默认配置合并的 var promise = Promise.resolve(config); promise.then('请求成功拦截2', '请求失败拦截2') .then('请求成功拦截1', '请求失败拦截1') .then(dispatchRequest, undefined) .then('响应成功拦截1', '响应失败拦截1') .then('响应成功拦截2', '响应失败拦截2') .then('用户写的业务处理函数') .catch('用户写的报错业务处理函数');`&#x20;

这里提下`promise` `then`和`catch`知识：`Promise.prototype.then`方法的第一个参数是`resolved`状态的回调函数，第二个参数（可选）是`rejected`状态的回调函数。所以是成对出现的。`Promise.prototype.catch`方法是`.then(null, rejection)`或`.then(undefined, rejection)`的别名，用于指定发生错误时的回调函数。`then`方法返回的是一个新的`Promise`实例（注意，不是原来那个`Promise`实例）。因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。 &#x20;

结合上述的例子更详细一点，代码则是这样的。

js

复制代码

\`var promise = Promise.resolve(config);

// promise.then('请求成功拦截2', '请求失败拦截2')

promise.then(function requestSuccess2(config) {

console.log('------request------success------2');

return config;

}, function requestError2(error) {

console.log('------response------error------2');

return Promise.reject(error);

})

// .then('请求成功拦截1', '请求失败拦截1')

.then(function requestSuccess1(config) {

console.log('------request------success------1');

return config;

}, function requestError1(error) {

console.log('------response------error------1');

return Promise.reject(error);

})

// .then(dispatchRequest, undefined)

.then( function dispatchRequest(config) {

/\*\*

- 适配器返回的也是Promise 实例

    adapter = function xhrAdapter(config) {

    return new Promise(function dispatchXhrRequest(resolve, reject) {})

    }

\*\*/

return adapter(config).then(function onAdapterResolution(response) {

// 省略代码 ...

return response;

}, function onAdapterRejection(reason) {

// 省略代码 ...

return Promise.reject(reason);

});

}, undefined)

// .then('响应成功拦截1', '响应失败拦截1')

.then(function responseSuccess1(response) {

console.log('------response------success------1');

return response;

}, function responseError1(error) {

console.log('------response------error------1');

return Promise.reject(error);

})

// .then('响应成功拦截2', '响应失败拦截2')

.then(function responseSuccess2(response) {

console.log('------response------success------2');

return response;

}, function responseError2(error) {

console.log('------response------error------2');

return Promise.reject(error);

})

// .then('用户写的业务处理函数')

// .catch('用户写的报错业务处理函数');

.then(function (response) {

console.log('哈哈哈，终于获取到数据了', response);

})

.catch(function (err) {

console.log('哎呀，怎么报错了', err);

});\`&#x20;

仔细看这段`Promise`链式调用，代码都类似。`then`方法最后返回的参数，就是下一个`then`方法第一个参数。`catch`错误捕获，都返回`Promise.reject(error)`，这是为了便于用户`catch`时能捕获到错误。

举个例子：

js

复制代码

\`var p1 = new Promise((resolve, reject) => {

reject(new Error({name: '若川'}));

});

p1.catch(err => {

console.log(res, 'err');

return Promise.reject(err)

})

.catch(err => {

console.log(err, 'err1');

})

.catch(err => {

console.log(err, 'err2');

});\`&#x20;

`err2`不会捕获到，也就是不会执行，但如果都返回了`return Promise.reject(err)`，则可以捕获到。

最后画个图总结下 `Promise` 链式调用。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/16/16f0f56a048b5c7d~tplv-t2oaga2asx-zoom-in-crop-mark:1512:0:0:0.awebp)

> 小结：1. 请求和响应的拦截器可以写`Promise`。2. 如果设置了多个请求响应器，后设置的先执行。3. 如果设置了多个响应拦截器，先设置的先执行。 &#x20;

`dispatchRequest(config)` 这里的`config`是请求成功拦截器返回的。接下来看`dispatchRequest`函数。

### 5.3 dispatchRequest 最终派发请求

这个函数主要做了如下几件事情： &#x20;

> 1.如果已经取消，则 `throw` 原因报错，使`Promise`走向`rejected`。2.确保 `config.header` 存在。3.利用用户设置的和默认的请求转换器转换数据。4.拍平 `config.header`。5.删除一些 `config.header`。6.返回适配器`adapter`（`Promise`实例）执行后 `then`执行后的 `Promise`实例。返回结果传递给响应拦截器处理。 &#x20;

js

复制代码

\`\`'use strict';

// utils 工具函数

var utils = require('./../utils');

// 转换数据

var transformData = require('./transformData');

// 取消状态

var isCancel = require('../cancel/isCancel');

// 默认参数

var defaults = require('../defaults');

/\*\*

- 抛出 错误原因，使`Promise`走向`rejected`

\*/

function throwIfCancellationRequested(config) {

if (config.cancelToken) {

config.cancelToken.throwIfRequested();

}

}

/\*\*

- Dispatch a request to the server using the configured adapter.

\*

- @param {object} config The config that is to be used for the request
- @returns {Promise} The Promise to be fulfilled

\*/

module.exports = function dispatchRequest(config) {

// 取消相关

throwIfCancellationRequested(config);

// Ensure headers exist

// 确保 headers 存在

config.headers = config.headers || {};

// Transform request data

// 转换请求的数据

config.data = transformData(

config.data,

config.headers,

config.transformRequest

);

// Flatten headers

// 拍平 headers

config.headers = utils.merge(

config.headers.common || {},

config.headers\[config.method] || {},

config.headers || {}

);

// 以下这些方法 删除 headers

utils.forEach(

\['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],

function cleanHeaderConfig(method) {

delete config.headers\[method];

}

);

// adapter 适配器部分 拆开 放在下文讲

};\`\`&#x20;

#### 5.3.1 dispatchRequest 之 transformData 转换数据

上文的代码里有个函数 `transformData` ，这里解释下。其实就是遍历传递的函数数组 对数据操作，最后返回数据。

`axios.defaults.transformResponse` 数组中默认就有一个函数，所以使用`concat`链接自定义的函数。

使用：

文件路径 `axios/examples/transform-response/index.html`

这段代码其实就是对时间格式的字符串转换成时间对象，可以直接调用`getMonth`等方法。

js

复制代码

\`var ISO\_8601 = /(\d{4}-\d{2}-\d{2})T(\d{2}:\d{2}:\d{2})Z/;

function formatDate(d) {

return (d.getMonth() + 1) + '/' + d.getDate() + '/' + d.getFullYear();

}

axios.get('[https://api.github.com/users/mzabriskie](https://api.github.com/users/mzabriskie "https://api.github.com/users/mzabriskie")', {

transformResponse: axios.defaults.transformResponse.concat(function (data, headers) {

Object.keys(data).forEach(function (k) {

if (ISO\_8601.test(data\[k])) {

data\[k] = new Date(Date.parse(data\[k]));

}

});

return data;

})

})

.then(function (res) {

document.getElementById('created').innerHTML = formatDate(res.data.created\_at);

});\`&#x20;

源码：

就是遍历数组，调用数组里的传递 `data` 和 `headers` 参数调用函数。

js

复制代码

\`module.exports = function transformData(data, headers, fns) {

/*eslint no-param-reassign:0*/

utils.forEach(fns, function transform(fn) {

data = fn(data, headers);

});

return data;

};\`&#x20;

#### 5.3.2 dispatchRequest 之 adapter 适配器执行部分

适配器，在设计模式中称之为适配器模式。讲个生活中简单的例子，大家就容易理解。

我们常用以前手机耳机孔都是圆孔，而现在基本是耳机孔和充电接口合二为一。统一为`typec`。

这时我们需要需要一个`typec转圆孔的转接口`，这就是适配器。

js

复制代码

\` // adapter 适配器部分

var adapter = config.adapter || defaults.adapter;

return adapter(config).then(function onAdapterResolution(response) {

throwIfCancellationRequested(config);

// Transform response data

// 转换响应的数据

response.data = transformData(

response.data,

response.headers,

config.transformResponse

);

return response;

}, function onAdapterRejection(reason) {

if (!isCancel(reason)) {

// 取消相关

throwIfCancellationRequested(config);

// Transform response data

// 转换响应的数据

if (reason && reason.response) {

reason.response.data = transformData(

reason.response.data,

reason.response.headers,

config.transformResponse

);

}

}

return Promise.reject(reason);

});\`&#x20;

接下来看具体的 `adapter`。

### 5.4 adapter 适配器 真正发送请求

js

复制代码

`var adapter = config.adapter || defaults.adapter;`&#x20;

看了上文的 `adapter`，可以知道支持用户自定义。比如可以通过微信小程序 `wx.request` 按照要求也写一个 `adapter`。接着来看下 `defaults.ddapter`。文件路径：`axios/lib/defaults.js`

根据当前环境引入，如果是浏览器环境引入`xhr`，是`node`环境则引入`http`。类似判断`node`环境，也在[sentry-javascript](https://link.juejin.cn?target=https://github.com/getsentry/sentry-javascript/blob/a876d46c61e2618e3c3a3e1710f77419331a9248/packages/utils/src/misc.ts#L37-L40 "sentry-javascript")源码中有看到。 &#x20;

js

复制代码

\`function getDefaultAdapter() {

var adapter;

// 根据 XMLHttpRequest 判断

if (typeof XMLHttpRequest !== 'undefined') {

// For browsers use XHR adapter

adapter = require('./adapters/xhr');

// 根据 process 判断

} else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '\[object process]') {

// For node use HTTP adapter

adapter = require('./adapters/http');

}

return adapter;

}

var defaults = {

adapter: getDefaultAdapter(),

// ...

};\`&#x20;

`xhr`

接下来就是我们熟悉的 `XMLHttpRequest` 对象。

参考[XMLHttpRequest MDN 文档](https://link.juejin.cn?target=https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest "XMLHttpRequest MDN 文档")。

主要提醒下：`onabort`是请求取消事件，`withCredentials`是一个布尔值，用来指定跨域 `Access-Control` 请求是否应带有授权信息，如 `cookie` 或授权 `header` 头。

这块代码有删减，具体可以看[若川的](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/xhr.js "若川的")[axios-analysis](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/xhr.js "axios-analysis")[仓库](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/xhr.js "仓库")，也可以克隆`axios-analysis`仓库调试时再具体分析。

js

复制代码

\`module.exports = function xhrAdapter(config) {

return new Promise(function dispatchXhrRequest(resolve, reject) {

// 这块代码有删减

var request = new XMLHttpRequest();

request.open()

request.timeout = config.timeout;

// 监听 state 改变

request.onreadystatechange = function handleLoad() {

if (!request || request.readyState !== 4) {

return;

}

// ...

}

// 取消

request.onabort = function(){};

// 错误

request.onerror = function(){};

// 超时

request.ontimeout = function(){};

// cookies 跨域携带 cookies 面试官常喜欢考这个

// 一个布尔值，用来指定跨域 Access-Control 请求是否应带有授权信息，如 cookie 或授权 header 头。

// Add withCredentials to request if needed

if (!utils.isUndefined(config.withCredentials)) {

request.withCredentials = !!config.withCredentials;

}

// 上传下载进度相关

// Handle progress if needed

if (typeof config.onDownloadProgress === 'function') {

request.addEventListener('progress', config.onDownloadProgress);

}

// Not all browsers support upload events

if (typeof config.onUploadProgress === 'function' && request.upload) {

request.upload.addEventListener('progress', config.onUploadProgress);

}

// Send the request

// 发送请求

request.send(requestData);

});

}\`&#x20;

而实际上现在 [fetch](https://link.juejin.cn?target=https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch "fetch") 支持的很好了，阿里开源的 [umi-request](https://link.juejin.cn?target=https://github.com/umijs/umi-request/blob/master/README_zh-CN.md "umi-request") 请求库，就是用`fetch`封装的，而不是用`XMLHttpRequest`。 文章末尾，大概讲述下 `umi-request` 和 `axios` 的区别。

`http`

`http：`[若川的](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/http.js "若川的")[axios-analysis](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/http.js "axios-analysis")[仓库](https://link.juejin.cn?target=https://github.com/lxchuan12/axios-analysis/blob/master/axios/lib/adapters/http.js "仓库")。

js

复制代码

\`module.exports = function httpAdapter(config) {

return new Promise(function dispatchHttpRequest(resolvePromise, rejectPromise) {

});

};\`&#x20;

上文 `dispatchRequest` 有取消模块，我觉得是重点，所以放在最后来细讲：

### 5.5 dispatchRequest 之 取消模块

可以使用`cancel token`取消请求。

axios cancel token API 是基于撤销的 `promise` 取消提议。

The axios cancel token API is based on the withdrawn [cancelable promises proposal.](https://link.juejin.cn?target=https://github.com/tc39/proposal-cancelable-promises "cancelable promises proposal.")

[axios 文档 cancellation](https://link.juejin.cn?target=https://github.com/axios/axios#cancellation "axios 文档 cancellation")

文档上详细描述了两种使用方式。

很遗憾，在`example`文件夹也没有取消的例子。在`example`中在`example/get`的基础上添加了一个取消的示例。`axios/examples/cancel`，便于调试。

bash

复制代码

`node ./examples/server.js -p 5000`&#x20;

`request`中的拦截器和`dispatch`中的取消这两个模块相对复杂，可以多调试调试，吸收消化。

js

复制代码

\`const CancelToken = axios.CancelToken;

const source = CancelToken.source();

axios.get('/get/server', {

cancelToken: source.token

}).catch(function (err) {

if (axios.isCancel(err)) {

console.log('Request canceled', err.message);

} else {

// handle error

}

});

// cancel the request (the message parameter is optional)

// 取消函数。

source.cancel('哎呀，我被若川取消了');\`&#x20;

#### 5.5.1 取消请求模块代码示例

结合源码取消流程大概是这样的。这段放在代码在`axios/examples/cancel-token/index.html`。

参数的 `config.cancelToken` 是触发了`source.cancel('哎呀，我被若川取消了');`才生成的。

js

复制代码

\`// source.cancel('哎呀，我被若川取消了');

// 点击取消时才会 生成 cancelToken 实例对象。

// 点击取消后，会生成原因，看懂了这段在看之后的源码，可能就好理解了。

var config = {

name: '若川',

// 这里简化了

cancelToken: {

promise: new Promise(function(resolve){

resolve({ message: '哎呀，我被若川取消了'})

}),

reason: { message: '哎呀，我被若川取消了' }

},

};

// 取消 抛出异常方法

function throwIfCancellationRequested(config){

// 取消的情况下执行这句

if(config.cancelToken){

//   这里源代码 便于执行，我改成具体代码

// config.cancelToken.throwIfRequested();

// if (this.reason) {

//     throw this.reason;

//   }

if(config.cancelToken.reason){

throw config.cancelToken.reason;

}

}

}

function dispatchRequest(config){

// 有可能是执行到这里就取消了，所以抛出错误会被err2 捕获到

throwIfCancellationRequested(config);

//  adapter xhr适配器

return new Promise((resovle, reject) => {

var request = new XMLHttpRequest();

console.log('request', request);

if (config.cancelToken) {

// Handle cancellation

config.cancelToken.promise.then(function onCanceled(cancel) {

if (!request) {

return;

}

request.abort();

reject(cancel);

// Clean up request

request = null;

});

}

})

.then(function(res){

// 有可能是执行到这里就才取消 取消的情况下执行这句

throwIfCancellationRequested(config);

console.log('res', res);

return res;

})

.catch(function(reason){

// 有可能是执行到这里就才取消 取消的情况下执行这句

throwIfCancellationRequested(config);

console.log('reason', reason);

return Promise.reject(reason);

});

}

var promise = Promise.resolve(config);

// 没设置拦截器的情况下是这样的

promise

.then(dispatchRequest, undefined)

// 用户定义的then 和 catch

.then(function(res){

console.log('res1', res);

return res;

})

.catch(function(err){

console.log('err2', err);

return Promise.reject(err);

});

// err2 {message: "哎呀，我被若川取消了"}\`&#x20;

#### 5.5.2 接下来看取消模块的源码

看如何通过生成`config.cancelToken`。

文件路径：

`axios/lib/cancel/CancelToken.js`

js

复制代码

`const CancelToken = axios.CancelToken; const source = CancelToken.source(); source.cancel('哎呀，我被若川取消了');`&#x20;

由示例看 `CancelToken.source`的实现，

js

复制代码

\`CancelToken.source = function source() {

var cancel;

var token = new CancelToken(function executor(c) {

cancel = c;

});

// token

return {

token: token,

cancel: cancel

};

};\`&#x20;

执行后`source`的大概结构是这样的。

js

复制代码

\`{

token: {

promise: new Promise(function(resolve){

resolve({ message: '哎呀，我被若川取消了'})

}),

reason: { message: '哎呀，我被若川取消了' }

},

cancel: function cancel(message) {

if (token.reason) {

// Cancellation has already been requested

// 已经取消

return;

}

token.reason = {message: '哎呀，我被若川取消了'};

}

}\`&#x20;

接着看 `new CancelToken`

js

复制代码

\`// CancelToken

// 通过 CancelToken 来取消请求操作

function CancelToken(executor) {

if (typeof executor !== 'function') {

throw new TypeError('executor must be a function.');

}

var resolvePromise;

this.promise = new Promise(function promiseExecutor(resolve) {

resolvePromise = resolve;

});

var token = this;

executor(function cancel(message) {

if (token.reason) {

// Cancellation has already been requested

// 已经取消

return;

}

token.reason = new Cancel(message);

resolvePromise(token.reason);

});

}

module.exports = CancelToken;\`&#x20;

发送请求的适配器里是这样使用的。

js

复制代码

\`// xhr

if (config.cancelToken) {

// Handle cancellation

config.cancelToken.promise.then(function onCanceled(cancel) {

if (!request) {

return;

}

request.abort();

reject(cancel);

// Clean up request

request = null;

});

}\`&#x20;

`dispatchRequest` 中的`throwIfCancellationRequested`具体实现：throw 抛出异常。

js

复制代码

\`// 抛出异常函数

function throwIfCancellationRequested(config) {

if (config.cancelToken) {

config.cancelToken.throwIfRequested();

}

}

// 抛出异常 用户 { message: '哎呀，我被若川取消了' }

CancelToken.prototype.throwIfRequested = function throwIfRequested() {

if (this.reason) {

throw this.reason;

}

};\`&#x20;

取消流程调用栈

> 1.source.cancel() &#x20;
> 2.resolvePromise(token.reason); &#x20;
> 3.config.cancelToken.promise.then(function onCanceled(cancel) {}) &#x20;

最后进入`request.abort();``reject(cancel);`

到这里取消的流程就介绍完毕了。主要就是通过传递配置参数`cancelToken`，取消时才会生成`cancelToken`，判断有，则抛出错误，使`Promise` 走向`rejected`，让用户捕获到消息{message: '用户设置的取消信息'}。
