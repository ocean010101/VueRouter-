## VueRouter 构造函数

new VueRouter(options)

作用：

​	构建**路由器对象**。

window.history对象： 

​	用来存放允许操作浏览器的曾经在标签页或者框架里访问的会话历史记录

1. 初始化VueRouter实例中的属性
2. 根据mode构建VueRouter实例中history对象

具体实现如下：

```js
var VueRouter = function VueRouter(options) {
    if (options === void 0) options = {};
	// 1. 初始化VueRouter实例中的属性
    //用来存放调用VueRouter实例的当前Vue实例
    this.app = null;
    //用来存放调用VueRouter实例的所有Vue实例
    this.apps = [];
    //用来存放构建选项options
    this.options = options;
    //用来存放调用router.beforeEach 方法时传入的guard function	
    this.beforeHooks = [];
    //用来存放调用router.beforeR//esolve方法时传入的guard function	
    this.resolveHooks = [];
    //用来存放调用router.afterEach方法时传入的guard function
    this.afterHooks = [];
    //this.matcher具有match方法和addRoutes方法
    //存储
    this.matcher = createMatcher(options.routes || [], this);

    //根据构建选项options和当前运行环境判断应该使用什么模式
    var mode = options.mode || 'hash'; 
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false; // 
    if (this.fallback) {
        mode = 'hash';
    }
    if (!inBrowser) {
        mode = 'abstract';
    }
    //用来存放模式
    this.mode = mode;

    //3. 根据mode构建VueRouter实例中history对象
    switch (mode) {
        case 'history':
            this.history = new HTML5History(this, options.base);
            break
        case 'hash':
            this.history = new HashHistory(this, options.base, this.fallback);
            break
        case 'abstract':
            this.history = new AbstractHistory(this, options.base);
            break
        default:
            {
                assert(false, ("invalid mode: " + mode));
            }
    }
};
```



我们先看一下构造函数的参数options

### 构建选项options

##### routes

​	是一个RouterConfig类型的数组

​	RouterConfig 是一个 对象，记录路由信息

```js
interface RouteConfig = {
  path: string, //激活当前路由所需要匹配的URL 模式
  component?: Component, //是激活当前路由需要在<router-view />组件中渲染的组件
  name?: string, //路由名称：用于命名路由
  components?: { [name: string]: Component }, //用于命名视图组件，需要与< router-view /> 的name prop 配合使用
  redirect?: string | Location | Function, //用于路由重定向
  props?: boolean | Object | Function, //TODO
  alias?: string | Array<string>,// 别名
  children?: Array<RouteConfig>, //用于定义子路由 嵌套路由
  beforeEnter?: (to: Route, from: Route, next: Function) => void, //用于导航守卫
  meta?: any,//路由元信息，可以将任何其他信息放在meta中以扩展路由器的功能

  // 2.6.0+
  caseSensitive?: boolean, // 匹配规则是否大小写敏感？(默认值：false)
  pathToRegexpOptions?: Object // 编译正则的选项
}
```

###### path

含义：

​	记录激活当前路由所需要匹配的URL 模式

类型：

​	string

###### component

含义：

​	是激活当前路由需要在<router-view />组件中的组件

类型：

​	Component

###### name

含义:

​	路由名称

类型：

​	string

###### alias

含义:

​	别名

类型：

​	string

###### components

含义:

​	命名视图组件

​	需要与< router-view /> 的name prop 配合使用

类型：

###### redirect

作用:

​	用于路由重定向

类型：

​	string | Location | Function



###### props

含义:

类型：

​	boolean | Object | Function



###### beforeEnter

作用：

​	用于导航守卫

类型：

​	守卫函数

使用示例：

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})

```



###### meta

含义：

​	路由元信息

作用：

​	可以将任何其他信息放在meta中以扩展路由器的功能

​	

###### children 

作用：

​	用于定义子路由

##### 

```js
var mode = options.mode || 'hash';
//!supportsPushState 如果当前环境不支持HTML5 History Api
this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false; // 
if (this.fallback) { 
    mode = 'hash'; //把mode从history回退到hash
}
if (!inBrowser) { //如果当前环境不能使用window对象，设置mode为abstract
    mode = 'abstract';
}
this.mode = mode;
```

根据VueRouter构造函数内部代码， 我们获得以下信息：

##### fallback

含义：

​	当浏览器不支持 history.pushState控制路由时是否应该回退到 hash模式

默认值：

​	true

作用：

​	当fallback 值为true，mode为history， 并且当前环境不支持history.pushState 时， mode将回退到hash

##### mode

含义：

​	路由器模式。用于标记当前运行环境 。VueRouter构造函数 根据mode构建实例中history对象

默认值：

​	'hash'

###### hash

​	浏览器环境默认值，这是最“安全的”模式，因为它与任何浏览器（包括不支持 HTML5 History Api 的浏览器）和服务器都兼容。它使用URL 的hash 部分（指＃符号后面的部分），并对其进行更改或响应其变化。

​	URL: http://localhost:8080/#/faq

​	优点：当hash 部分改变时，不会改变应用运行的真实网页。

​	缺点：它迫使我们使用不那么优雅的＃符号将URL 分成两部分。

###### history

​	Node.js 环境默认值。依赖 HTML5 History API 和服务器配置--**HTML5 History 模式**

​	URL:http://localhost:8080/faq	

​	优点： 摆脱了＃符号，为应用获得一个真实的URL

​	缺点： 

​		a) 浏览器需要支持这个history.pushState API，IE浏览器 <=9 的版本不支持 这个HTML5 API。

​		b) 因为我们的应用是个单页客户端应用, 所以服务器必须配置 当访问诸如/faq 之类的路由时发送主页 而不是抛出404 错误，因为它并不真正存在（没有名为fag.html 的文件）。这也意味着我们将不得不自己实现404 页面。

###### abstract

​		可以在任何JavaScript 环境中使用，如果没有发现可用的浏览器API, 路由器将被迫使用此模式。



##### base

含义：

​	应用的基路径

默认值:“/”

例如，如果整个单页应用服务在 /app/ 下，然后 base 就应该设为 "/app/"



RouterLink 部分代码：

```js
//1. 处理当路由被激活时<router-link /＞渲染的模板上需要添加的类
var classes = {};
//从全局VueRouter实例的构造选项中获取linkActiveClass
var globalActiveClass = router.options.linkActiveClass;
//从全局VueRouter实例的构造选项中获取linkExactActiveClass
var globalExactActiveClass = router.options.linkExactActiveClass;
// Support global empty active class
// 如果没有linkActiveClass， 那么activeClassFallback默认值为router-link-active
var activeClassFallback =
  globalActiveClass == null ? 'router-link-active' : globalActiveClass;
//如果没有linkExactActiveClass， exactActiveClassFallback默认值为router-link-active
var exactActiveClassFallback =
  globalExactActiveClass == null
    ? 'router-link-exact-active'
    : globalExactActiveClass;
// 如果RouterLink 的props中没有设置activeClass， 那么取activeClassFallback的值
var activeClass =
  this.activeClass == null ? activeClassFallback : this.activeClass;
// 如果RouterLink 的props中没有设置exactActiveClass， 那么取activeClassFallback的值
var exactActiveClass =
  this.exactActiveClass == null
    ? exactActiveClassFallback
    : this.exactActiveClass;

var compareTarget = route.redirectedFrom
  ? createRoute(null, normalizeLocation(route.redirectedFrom), null, router)
  : route;

// 如果当前路由current 与 目标路由相同， 那么添加exactActiveClass 类
classes[exactActiveClass] = isSameRoute(current, compareTarget);

// 如果exact prop 为true， 那么添加classes[exactActiveClass] 类
classes[activeClass] = this.exact
  ? classes[exactActiveClass]
  : isIncludedRoute(current, compareTarget);

// ...
var data = { class: classes };

// ...
return h(this.tag, data, this.$slots.default)
```

根据RouterLink代码， 我们获得以下信息：

##### linkActiveClass 

类型:

​	string

默认值：

​	"router-link-active"

作用：

​	用于全局配置 路由匹配时<router-link />上需要添加的类的默认值 

​	如果没有设置linkActiveClass 选项并且RouterLink的**active-class** prop 为 null，当路由匹配后激活时添加名字为**‘router-link-active’** 的类

##### linkExactActiveClass

作用：

​	用于全局配置 路由精准匹配时<router-link />上需要添加的类的默认值 

​	如果没有设置linkExactActiveClass选项并且RouterLink的**exact-active-class** prop 为 null，当路由精准匹配后激活时添加名字为‘**router-link-exact-active’** 的类

类型:

​	string

默认值：

​	"router-link-exact-active"



##### scrollBehavior

作用：	

​	用于设置前端路由的滚动行为

​	这个功能只在支持 history.pushState 的浏览器中可用

类型: Function

使用示例：

​	参考：https://github.com/vuejs/vue-router/blob/dev/examples/scroll-behavior/app.js

```js
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const Home = { template: '<div class="home">home</div>' }
const Foo = { template: '<div class="foo">foo</div>' }
const Bar = {
  template: `
    <div class="bar">
      bar
      <div style="height:1500px"></div>
      <p id="anchor" style="height:500px">Anchor</p>
      <p id="anchor2" style="height:500px">Anchor2</p>
      <p id="1number">with number</p>
    </div>
  `
}

// scrollBehavior:
// - only available in html5 history mode
// - defaults to no scroll behavior
// - return false to prevent scroll
const scrollBehavior = function (to, from, savedPosition) {
  if (savedPosition) {
    // savedPosition is only available for popstate navigations.
    return savedPosition
  } else {
    const position = {}

    // scroll to anchor by returning the selector
    if (to.hash) { // 将检查路由是否有模仿浏览器行为的散列值
      position.selector = to.hash

      // specify offset of the element
      if (to.hash === '#anchor2') {
        position.offset = { y: 100 }
      }

      // bypass #1number check
      if (/^#\d/.test(to.hash) || document.querySelector(to.hash)) {
        return position
      }

      // if the returned position is falsy or an empty object,
      // will retain current scroll position.
      return false
    }

    return new Promise(resolve => {
      // check if any matched route config has meta that requires scrolling to top
      if (to.matched.some(m => m.meta.scrollToTop)) {
        // coords will be used if no selector is provided,
        // or if the selector didn't match any element.
        // { x: 0, y: 0 } //在路由改变时滚动到页面的顶部
        position.x = 0
        position.y = 0
      }

      // wait for the out transition to complete (if necessary)
      this.app.$root.$once('triggerScroll', () => {
        // if the resolved position is falsy or an empty object,
        // will retain current scroll position.
        resolve(position)
      })
    })
  }
}

const router = new VueRouter({
  mode: 'history',
  base: __dirname,
  scrollBehavior,
  routes: [
    { path: '/', component: Home, meta: { scrollToTop: true }},
    { path: '/foo', component: Foo },
    { path: '/bar', component: Bar, meta: { scrollToTop: true }}
  ]
})

new Vue({
  router,
  template: `
    <div id="app">
      <h1>Scroll Behavior</h1>
      <ul>
        <li><router-link to="/">/</router-link></li>
        <li><router-link to="/foo">/foo</router-link></li>
        <li><router-link to="/bar">/bar</router-link></li>
        <li><router-link to="/bar#anchor">/bar#anchor</router-link></li>
        <li><router-link to="/bar#anchor2">/bar#anchor2</router-link></li>
        <li><router-link to="/bar#1number">/bar#1number</router-link></li>
      </ul>
      <transition name="fade" mode="out-in" @after-leave="afterLeave">
        <router-view class="view"></router-view>
      </transition>
    </div>
  `,
  methods: {
    afterLeave () {
      this.$root.$emit('triggerScroll')
    }
  }
}).$mount('#app')
```



##### parseQuery / stringifyQuery

类型：

​	Function

作用：

​	自定义查询字符串的解析/反解析函数。覆盖默认行为。

​	VueRouter 内部定义了默认的parseQuery  和 stringifyQuery 函数用于查询字符串的解析/反解析。	



#### Router实例属性

##### router.app

含义：

​	配置了 router 的 Vue 根实例

类型: 

​	Vue实例

##### router.mode

含义：

​	路由使用的模式

类型: 

​	string

##### router.currentRoute

含义：

​	当前路由对应的路由信息对象。值与this.$route相同

类型

​	Route

### Router实例方法

#### beforeEach

作用：

​	注册全局前置守卫（before guards）

参数：

​	守卫函数（guard function）

返回值：

​	一个函数， 这个函数用来移除已注册的前置守卫

具体实现：

```js
VueRouter.prototype.beforeEach = function beforeEach(fn) {
    return registerHook(this.beforeHooks, fn)
};
```

使用：

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

##### 守卫函数

参数：

​	to:Route 即将要进入的目标路由

​	from:Route 当前要离开的路由

​	next:Function  用来解析导航守卫钩子。

###### next

参数to的取值

​	false: 中断当前导航，调用ensureURL方法重置URL地址为`from` 路由对应的地址

​	Error实例： 中断当前导航， 并且把这个Error实例传递给通过onError注册的回调

​	字符串 或对象，这个对象包含 string 类型的path属性或name属性： 当前的导航被中断，然后进行一个新的导航

​	其他值：进入队列中下一个函数

工作原理：

```js
var iterator = function (hook, next) {
  if (this$1.pending !== route) {
    return abort()
  }
  try {
    hook(route, current, function (to) { // next
      
      if (to === false || isError(to)) {
        // next(false) -> abort navigation, ensure current URL
        this$1.ensureURL(true);
        abort(to);
      } else if (
        typeof to === 'string' ||
        (typeof to === 'object' &&
          (typeof to.path === 'string' || typeof to.name === 'string'))
      ) {
        // next('/') or next({ path: '/' }) -> redirect
        //当前的导航被中断，然后进行一个新的导航
        abort();
        if (typeof to === 'object' && to.replace) {
          this$1.replace(to);
        } else {
          this$1.push(to);
        }
      } else {
        // confirm transition and pass on the value
        next(to);
      }
    });
  } catch (e) {
    abort(e);// 执行导航守卫函数出错
  }
};

function runQueue(queue, fn, cb) {
  var step = function (index) {
    if (index >= queue.length) {
      cb();
    } else {
      if (queue[index]) {
        fn(queue[index], function () {
          step(index + 1); 
        });
      } else {
        step(index + 1);
      }
    }
  };
  step(0);
}

//迭代queue中函数
runQueue(queue, iterator, function () {
  // ...
});

```

##### registerHook

作用：

​	注册钩子

参数：

​	list ： 保存守卫函数钩子的列表

​	fn： 守卫函数

返回值：

​	一个函数， 这个函数用来从保存守卫函数的钩子中移除已注册的守卫

具体实现：

```js
function registerHook(list, fn) {
    list.push(fn);
    return function () {
        var i = list.indexOf(fn);
        if (i > -1) { list.splice(i, 1); }
    }
}
```



#### beforeResolve

作用：

​	注册全局解析守卫（resolve guards）

参数：

​	守卫函数（guard function）

返回值：

​	一个函数， 这个函数用来移除已注册的解析守卫



具体实现：

```js
VueRouter.prototype.beforeResolve = function beforeResolve(fn) {
    return registerHook(this.resolveHooks, fn)
};
```

使用：

```js
const router = new VueRouter({ ... })

router.beforeResolve((to, from, next) => {
  // ...
})
```



#### afterEach

作用：

​	注册全局后置守卫（after guards）

参数：

​	守卫函数（guard function）

返回值：

​	一个函数， 这个函数用来移除已注册的后置守卫

具体实现：

```js
VueRouter.prototype.afterEach = function afterEach(fn) {
    return registerHook(this.afterHooks, fn)
};
```

使用：

```js
const router = new VueRouter({ ... })

router.afterEach((to, from) => {
  // ...
})
```



#### onReady

onReady(callback, [errorCallback])

作用：

​	注册router初始化导航的回调函数。

参数：

​	callback ： Function - -在路由完成初始导航时调用

​	errorCallback：Function -- 在初始化路由解析运行出错时被调用 

返回值：	

​	无

工作原理：

​	触发导航时， 内部通过**transitionTo**方法进行导航。 **transitionTo**在通过**confirmTransition** 确认导航后调用this.readyCbs 或 this.readyErrorCbs中的回调函数。 

this.readyCbs 和this.readyErrorCbs 中的回调函数都是通过onReady 注册的 。

​	在路由完成初始导航时调用this.readyCbs中的回调。这意味着它可以解析所有的异步进入钩子和路由初始化相关联的异步组件。这可以有效确保服务端渲染时服务端和客户端输出的一致。

具体实现如下：

```js
VueRouter.prototype.onReady = function onReady(cb, errorCb) {
    this.history.onReady(cb, errorCb);
};

History.prototype.onReady = function onReady(cb, errorCb) {
  if (this.ready) {
      cb(); // 直接调用回调函数
  } else {
      this.readyCbs.push(cb); // 把callback添加到this.readyCbs
      if (errorCb) {
          // 把errorCallback添加到this.readyErrorCbs
          this.readyErrorCbs.push(errorCb);
      }
  }
};
```



#### onError

定义:

​	onError(callback)

作用：

​	注册一个回调，该回调会在路由导航过程中出错时被调用

参数：

​	callback ： Function

返回值：

​	无

工作原理：

​	当触发导航时，VueRouter内部通过**confirmTransition** 确认导航。confirmTransition在确认导航过程中会执行导航守卫函数和解析异步路由组件。 在**执行导航守卫函数**和**解析异步路由组件** 过程中出现错误会调用内部的abort函数停止导航并调用onError 注册的回调函数。

通过代码可以看到出错的情况有以下三种：

1. 调用守卫函数错误
2. 在守卫函数的next中出错

​	3.通过resolveAsyncComponents 方法解析异步路由组件出错

具体实现如下：

```js
VueRouter.prototype.onError = function onError(errorCb) {
    this.history.onError(errorCb);
};
```

```js
History.prototype.onError = function onError(errorCb) {
  // 把出错回调函数添加到this.errorCbs
  this.errorCbs.push(errorCb);
};
```



#### push

定义：

​	router.push(location, onComplete?, onAbort?)
​	router.push(location).then(onComplete).catch(onAbort)

作用：

​	导航到指定的URL，并且把这条新的 history条目添加到history栈中。 

参数：

​	location: 是一个字符串路径，或者一个描述地址的对象

​	onComplete： 一个回调函数，在导航成功(所有的异步钩子被解析)后执行

​	onAbort：一个回调函数，在导航终止（导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由）时调用



如果RouterLink组件没有设置replace prop, 那么当触发导航时RouterLink内部调用VueRouter实例的push方法进行导航。

所以点击`<router-link  :to="..."/>`等同于调用`router.push("...") `



工作原理：

​	通过代码可以看到push实际上通过调用**transitionTo**方法进行导航。

具体实现如下：

```js
VueRouter.prototype.push = function push(location, onComplete, onAbort) {
    var this$1 = this;

    // $flow-disable-line 
    //如果没有onComplete 也没有onAbort但是支持Promise， 返回一个Promise对象
    if (!onComplete && !onAbort && typeof Promise !== 'undefined') {
        return new Promise(function (resolve, reject) {
            this$1.history.push(location, resolve, reject);
        })
    } else {
        this.history.push(location, onComplete, onAbort);
    }
};
HTML5History.prototype.push = function push(location, onComplete, onAbort) {
  var this$1 = this;
  var ref = this;
  var fromRoute = ref.current;
  this.transitionTo(location, function (route) {
    pushState(cleanPath(this$1.base + route.fullPath));
    handleScroll(this$1.router, route, fromRoute, false);
    onComplete && onComplete(route);
  }, onAbort);
};
HashHistory.prototype.push = function push(location, onComplete, onAbort) {
  var this$1 = this;

  var ref = this;
  var fromRoute = ref.current;
  this.transitionTo(
    location,
    function (route) {
      pushHash(route.fullPath);
      handleScroll(this$1.router, route, fromRoute, false);
      onComplete && onComplete(route);
    },
    onAbort
  );
};
AbstractHistory.prototype.push = function push(location, onComplete, onAbort) {
  var this$1 = this;

  this.transitionTo(
    location,
    function (route) {
      this$1.stack = this$1.stack.slice(0, this$1.index + 1).concat(route);
      this$1.index++;
      onComplete && onComplete(route);
    },
    onAbort
  );
};
```



#### replace

定义：

​	router.replace(location, onComplete?, onAbort?)
​	router.replace(location).then(onComplete).catch(onAbort)

作用：

​	导航到指定的URL，并且在history 栈中替换当前history 条目。

参数：

​	location: 是一个字符串路径，或者一个描述地址的对象

​	onComplete： 一个回调函数，在导航成功(所有的异步钩子被解析)后执行

​	onAbort：一个回调函数，在导航终止（导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由）时调用



如果在RouterLink组件设置replace prop， 那么当触发导航时RouterLink内部调用VueRouter实例的replace方法进行导航。

所以点击`<router-link :to="..." replace>` 等同于调用`router.replace(...)`

工作原理：

​	通过代码可以看到replace 实际上通过调用**transitionTo**方法进行导航。

具体实现如下：

```js
VueRouter.prototype.replace = function replace(location, onComplete, onAbort) {
    var this$1 = this;

    // $flow-disable-line
    // 如果没有onComplete 也没有onAbort但是支持Promise， 返回一个Promise对象
    if (!onComplete && !onAbort && typeof Promise !== 'undefined') {
        return new Promise(function (resolve, reject) {
            this$1.history.replace(location, resolve, reject);
        })
    } else {
        this.history.replace(location, onComplete, onAbort);
    }
};

HTML5History.prototype.replace = function replace(location, onComplete, onAbort) {
  var this$1 = this;
  var ref = this;
  var fromRoute = ref.current;
  this.transitionTo(location, function (route) {
    replaceState(cleanPath(this$1.base + route.fullPath));
    handleScroll(this$1.router, route, fromRoute, false);
    onComplete && onComplete(route);
  }, onAbort);
};
HashHistory.prototype.replace = function replace(location, onComplete, onAbort) {
  var this$1 = this;

  var ref = this;
  var fromRoute = ref.current;
  this.transitionTo(
    location,
    function (route) {
      replaceHash(route.fullPath);
      handleScroll(this$1.router, route, fromRoute, false);
      onComplete && onComplete(route);
    },
    onAbort
  );
};
AbstractHistory.prototype.replace = function replace(location, onComplete, onAbort) {
  var this$1 = this;

  this.transitionTo(
    location,
    function (route) {
      this$1.stack = this$1.stack.slice(0, this$1.index).concat(route);
      onComplete && onComplete(route);
    },
    onAbort
  );
};
```



#### go

定义：

​	router.go(n)

作用：

​	在 history 栈中向前或者后退n步 。如果history记录不够用，就不导航到新页面了，停留在当前页面

参数：

​	n: 整数

使用示例：

```js
router.go(-2);//后退2步

router.go(3);// 前进3步
```

具体实现如下：

```js
VueRouter.prototype.go = function go(n) {
    this.history.go(n);
};
```

#### back

定义:

​	back()

作用：

​	在 history stack 中后退1步 

使用示例：

```js
router.back()
```

具体实现如下：

```js
VueRouter.prototype.back = function back() {
    this.go(-1);
};
```



#### forward

定义：

​	forward()

作用：

​	在 history 栈中前进1步 

使用示例：

```js
router.forward()
```

具体实现如下：

```js
VueRouter.prototype.forward = function forward() {
    this.go(1);
};
```



#### getMatchedComponents

定义：

```js
const matchedComponents: Array<Component> = router.getMatchedComponents(location?)
```

作用：

​		返回目标位置或是当前路由匹配的组件数组 (是数组的定义/构造类，不是实例)。通常在服务端渲染的数据预加载时使用

具体实现如下：

```js
VueRouter.prototype.getMatchedComponents = function getMatchedComponents(to) {
    var route = to
        ? to.matched
            ? to
            : this.resolve(to).route
        : this.currentRoute;
    if (!route) {
        return []
    }
    return [].concat.apply([], route.matched.map(function (m) {
        return Object.keys(m.components).map(function (key) {
            return m.components[key]
        })
    }))
};
```



#### resolve

```js
const resolved: {
  location: Location;
  route: Route;
  href: string;
} = router.resolve(location, current?, append?)
```

作用：

​	反向url解析。

参数：

​	location ：与`<router link/>`  中 **to** prop 的格式相同

​	current： 是当前默认的路由 

​	append： 允许你在 current 路由上附加路径 (如同 router-link）

返回值：

```js
{
  location: location,
  route: route,
  href: href,
  // for backwards compat
  normalizedTo: location,
  resolved: route
}
```



具体实现如下：

```js
VueRouter.prototype.resolve = function resolve(
    to,
    current,
    append
) {
    console.log('VueRouter resolve', to, current, append);
    current = current || this.history.current;
    var location = normalizeLocation(
        to,
        current,
        append,
        this
    );
    var route = this.match(location, current);
    var fullPath = route.redirectedFrom || route.fullPath;
    var base = this.history.base;
    var href = createHref(base, fullPath, this.mode);
    return {
        location: location,
        route: route,
        href: href,
        // for backwards compat
        normalizedTo: location,
        resolved: route
    }
};
```



#### addRoutes

定义：

​	addRoutes(routes: Array<RouteConfig>)

作用：

​	动态地向路由器添加更多路由。参数必须是使用与“routes”构造函数选项相同的路由配置格式的数组。

返回值：

​	无

具体实现如下：

```js
VueRouter.prototype.addRoutes = function addRoutes(routes) {
    this.matcher.addRoutes(routes); 
    if (this.history.current !== START) {
        this.history.transitionTo(this.history.getCurrentLocation());
    }
};
```



### 路由对象route

值与VueRouter示例的属性currentRoute相同



#### 属性

##### $route.path

含义：

​	当前路由的路径，总是解析为绝对路径

类型:

​	 string

示例：



对应当前路由的路径，总是解析为绝对路径，如 "/foo/bar"。



##### $route.params

含义：

​	一个 key/value 对象，包含了动态片段和全匹配片段，如果没有路由参数，就是一个空对象。

类型:

​	Object

##### $route.query

含义：

​	一个 key/value 对象，表示 URL 查询参数。例如，对于路径 /foo?user=1，则有 $route.query.user == 1，如果没有查询参数，则是个空对象。

类型: 

​	Object



##### $route.hash

含义：

​	当前路由的 hash 值(带 #) ，如果没有 hash 值，则为空字符串。

类型：

​	string



##### $route.fullPath

含义：	

​	完成解析后的 URL，包含查询参数和 hash 的完整路径。

类型：

​	string

##### $route.matched

含义：

​	包含当前路由的所有嵌套路径片段的路由记录 。路由记录就是 routes 配置数组中的对象副本 (还有在 children 数组)。

类型: 

​	Array<RouteRecord>

示例：

​	当 URL 为 /foo/bar时， $route.matched 是一个按父子顺序包含两个对象（clone)的数组.

```js
const router = new VueRouter({
  routes: [
    // 下面的对象就是路由记录
    {
      path: '/foo',
      component: Foo,
      children: [
        // 这也是个路由记录
        { path: 'bar', component: Bar }
      ]
    }
  ]
})
```


当 URL 为 /foo/bar，$route.matched 将会是一个包含从上到下的所有对象 (副本)。



##### $route.redirectedFrom

含义：

​	重定向来源的路由的名字

类型：string

示例：

​	导航到“http://localhost:8080/bar”， 将被重定向到"http://localhost:8080/"

​	当前$route.redirectedFrom值为"/bar"

```js
const routes = [
  { path: '/', name: 'home', component: Home },
  { path: '/bar', name: 'bar', redirect: '/'},
]
```



### 组件注入

通过在 Vue 根实例的 `router` 配置传入 router 实例，每个子组件会被注入**this.$router** 和 **this.$route**

```js
new Vue({
  el: '#app',
  router,
  render: h => h(AppLayout),
})
```



#### 增加的组件配置选项

在安装vueRouter插件时， 调用vueRouter插件内部的install函数， 在install函数内部会添加beforeRouteEnter，beforeRouteLeave， 和 beforeRouteUpdate 钩子函数

```js
function install(Vue) {
	//...

    // 给Vue对象添加路由钩子函数 beforeRouteEnter， beforeRouteLeave， beforeRouteUpdate
    var strats = Vue.config.optionMergeStrategies;
    // use the same hook merging strategy for route hooks
    strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created;
}
```

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
     //参考完整导航解析流程，可以看到beforeRouteEnter守卫在导航确认前被调用, 这时组件还没被创建，所以这里不能获取组件实例 `this`
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
      
     next()
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    // next()
    // next(false) 取消离开
  }
}
```

##### beforeRouteEnter

是支持给 `next` 传递回调的唯一守卫。

我们可以通过传一个回调给 `next`来访问组件实例。这个回调在导航已经确认后，DOM 更新后执行。

```js
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```



##### beforeRouteUpdate

使用示例：

```js
beforeRouteUpdate (to, from, next) {
  // just use `this`
  this.name = to.params.name
  next()
}
```



##### beforeRouteLeave

使用示例：

```js
beforeRouteLeave (to, from , next) {
  const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
  if (answer) {
    next()
  } else {
    next(false) // 取消离开
  }
}

```

