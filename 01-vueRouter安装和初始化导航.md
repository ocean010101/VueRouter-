# VueRouter安装和初始化导航

想在Vue应用中使用VueRouter, 我们首先需要安装VueRouter。 方法如下：

## 安装VueRouter

```js
Vue.use(VueRouter)

const router = new VueRouter(options)

new Vue({
    el: '#app',
    data: state,
    router, //将路由器对象router作为一个选项添加到Vue根实例中
    render: h => h(AppLayout),
  })
```

安装VueRouter，分为以下两步：

1. 使用Vue.use通过调用VueRouter 插件中的install方法把VueRouter 安装到Vue

2. 为了每个Vue实例都能访问VueRouter实例， 创建Vue根实例时，把路由器对象router注入到Vue实例 

### 使用Vue.use把VueRouter 安装到Vue

Vue.use通过调用VueRouter 插件中的install方法把VueRouter 安装到Vue。我们从VueRouter的install方法入手看一下安装流程

#### install 方法

做了以下两件事：

1. 判断VueRouter是否安装

   如果已经安装，退出函数

2. 更新Vue对象

   2.1 通过Vue.mixin更新Vue对象的**beforeCreate**和**destroyed** 钩子

   2.2 通过Object.defineProperty 给Vue对象添加$router 和 $route 属性。

   2.3 在Vue对象上注册RouterView 和RouterLink 组件

   2.4 给Vue对象添加路由钩子函数 beforeRouteEnter， beforeRouteLeave和 beforeRouteUpdate。

具体实现如下：

```js
var _Vue;

function install(Vue) {
    if (install.installed && _Vue === Vue) { return } // 已安装过，退出函数
    install.installed = true; // installed 标记已安装

    //保存Vue，同时用于检测是否重复安装
    _Vue = Vue;

    // 
    var isDef = function (v) { return v !== undefined; };

    var registerInstance = function (vm, callVal) {
        var i = vm.$options._parentVnode;
        if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
            i(vm, callVal);
        }
    };
	// 更新Vue对象中的beforeCreate和destroyed 钩子
    Vue.mixin({
        beforeCreate: function beforeCreate() {
            if (isDef(this.$options.router)) { // 调用Vue构造函数创建实例时传入了router选项（根组件）
                this._routerRoot = this;
                this._router = this.$options.router;
                this._router.init(this);
                Vue.util.defineReactive(this, '_route', this._router.history.current);
            } else {
                this._routerRoot = (this.$parent && this.$parent._routerRoot) || this;
            }
            registerInstance(this, this); //把RouterView添加到虚拟Node
        },
        destroyed: function destroyed() {
            registerInstance(this);// 从虚拟node中移除RouterView
        }
    });
	// 给Vue对象添加$router属性
    Object.defineProperty(Vue.prototype, '$router', {
        get: function get() { return this._routerRoot._router }
    });
	// 给Vue对象添加$route属性
    Object.defineProperty(Vue.prototype, '$route', {
        get: function get() { return this._routerRoot._route }
    });

    Vue.component('RouterView', View); //注册RouterView组件，对应模板中的<router-view />
    Vue.component('RouterLink', Link);//注册 RouterLink组件，对应模板中的<router-link />

    //给Vue对象添加路由钩子函数 beforeRouteEnter， beforeRouteLeave， beforeRouteUpdate
    var strats = Vue.config.optionMergeStrategies;
    // use the same hook merging strategy for route hooks
    strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created;
}
```



### 创建Vue根实例时，把路由器对象router注入到Vue实例

```js
new Vue({
    el: '#app',
    data: state,
    router, //将路由器对象router作为一个选项添加到Vue根实例中
    render: h => h(AppLayout),
  })
```



由于 调用install 方法时，更新了Vue对象构造器， 那么调用new Vue(options) 构建Vue根实例时，

​	  每个Vue实例的初始化阶段都会在**beforeCreate**钩子中更新路由相关的操作，销毁阶段在**destroyed** 钩子中销毁路由相关的虚拟节点

​	  每个Vue实例中都有$router 和 $route 属性。每个Vue实例都可以通过this.$router 访问全局的VueRouter实例，通过this.$route访问当前路由记录 。

​	  每个Vue实例都能使用RouterView 和RouterLink 组件

​	  每个Vue实例包含路由钩子函数， 可用于**组件内**路由导航守卫。



##### Vue实例的初始化阶段在**beforeCreate**钩子做了哪些路由相关的操作？

- ​	给Vue实例添加_routerRoot 属性， 用于$router属性 和 $route 属性从中读取值

​		如果当前Vue实例是根实例

​			那么给当前Vue实例添加_routerRoot 属性， _router 属性 和 _route 属性

​			由于_routerRoot 指向当前Vue实例， 那么 _routerRoot 中就包含了 _router属性 和   _route属性。

​		如果不是根实例，

​			那么给当前Vue实例添加_routerRoot 属性， _routerRoot 是从父实例中获取的。

​		这样一来所有Vue实例就都有了_routerRoot 属性 。所有Vue实例 _routerRoot 属性都是从相同的地方获取值。

- ​	把RouterView组件需要展示的视图对应的虚拟节点添加到虚拟节点树

##### Vue实例的销毁阶段在**destroyed** 钩子做了哪些路由相关的操作？

-    把RouterView组件需要展示的视图对应的虚拟节点从虚拟节点树移除

```js
Vue.mixin({
  beforeCreate: function beforeCreate() { //混淆进Vue的beforeCreacte钩子中
      if (isDef(this.$options.router)) { // 调用Vue构造函数创建实例时传入了router选项（根组件）
          this._routerRoot = this;// this._routerRoot 指向当前Vue实例
          this._router = this.$options.router; 
          this._router.init(this); // 调用VueRouter实例的init方法进行初始化导航
          //使Vue根实例中的_route响应式
          //history.current记录当前路由对象信息
          Vue.util.defineReactive(this, '_route', this._router.history.current);
      } else {
          this._routerRoot = (this.$parent && this.$parent._routerRoot) || this;
      }
      registerInstance(this, this); //把RouterView组件需要展示的视图对应的虚拟节点添加到虚拟节点树
  },
  destroyed: function destroyed() { //混淆进Vue的destroyed钩子中
      registerInstance(this); // 把RouterView组件需要展示的视图对应的虚拟节点从虚拟节点树移除
  }
});
```



Vue实例的$router属性 和 $route 属性都从Vue实例的_routerRoot属性中获取值。

```js
// 给Vue对象添加$router属性
Object.defineProperty(Vue.prototype, '$router', {
  get: function get() { return this._routerRoot._router }
});
// 给Vue对象添加$route属性
Object.defineProperty(Vue.prototype, '$route', {
  get: function get() { return this._routerRoot._route }
});
```



下面我们看一下如何调用VueRouter实例的init方法进行初始化导航。

##### VueRouter 的init方法

做了以下几件事：

1. 检查VueRouter安装

2. 更新VueRouter实例apps属性 和 app属性

   	​	在VueRouter实例的apps属性中记录调用当前VueRouter实例的所有Vue实例。

   ​	在VueRouter实例的app属性中记录调用当前VueRouter实例的Vue实例。

3. 根据路由的当前路径进行导航

   history.getCurrentLocation()获取当前路径 eg :  '/login' 

4. 监听VueRouter实例中记录会话历史的history对象。如果发生变化，更新apps数组中每个Vue实例的_route 属性。

```js
VueRouter.prototype.init = function init(app /* Vue component instance */) {
    var this$1 = this;

    assert(
        install.installed,
        "not installed. Make sure to call `Vue.use(VueRouter)` " +
        "before creating root instance."
    ); //1. 检查VueRouter安装

    this.apps.push(app);//  2. 把当前Vue实例添加到VueRouter实例的apps属性中

    // set up app destroyed handler
    // https://github.com/vuejs/vue-router/issues/2639
    app.$once('hook:destroyed', function () {
        // clean out app from this.apps array once destroyed
        // 如果当前Vue实例调用了destroyed钩子函数。 那么从VueRouter实例的apps属性中删除当前Vue实例
        var index = this$1.apps.indexOf(app);
        if (index > -1) { this$1.apps.splice(index, 1); }
        // ensure we still have a main app or null if no apps
        // we do not release the router so it can be reused
        if (this$1.app === app) { this$1.app = this$1.apps[0] || null; }
    });

    // main app previously initialized
    // return as we don't need to set up new history listener
    if (this.app) {
        return
    }

    this.app = app; // 更新VueRouter实例中的app属性指向当前Vue实例

    var history = this.history; // 根据router的mode不同，HTML5History对象，HashHistory对象，AbstractHistory对象

    if (history instanceof HTML5History) { // history是HTML5History对象, mode == 'history'
        //history.getCurrentLocation() 获取当前路径'/'
        history.transitionTo(history.getCurrentLocation()); //导航到当前路径对应的页面
    } else if (history instanceof HashHistory) { // history是HashHistory对象, mode == 'hash'
        var setupHashListener = function () {
            history.setupListeners();
        };
        history.transitionTo(
            history.getCurrentLocation(),
            setupHashListener,
            setupHashListener
        );
    }

    //监听记录会话历史的history对象。如果路由发生变化，更新apps数组中每个Vue实例的_route 属性
    history.listen(function (route) {
        this$1.apps.forEach(function (app) {
            app._route = route;
        });
    });
};
```



通过代码可以看到 VueRouter中实现导航的使用transitionTo方法。

#### transitionTo

作用：

​	导航到期望的路由

参数：

​	location: 是一个字符串路径

​	onComplete： 一个回调函数，在确认导航后执行

​	onAbort：一个回调函数，在终止导航解析(导航到相同的路由， 或在当前导航完成之前导航到另一个不同的路由)时执行

工作流程：

1. 根据参数location获取目的路由对象route

2. 调用**confirmTransition** 确认导航

   2.1 如果确认要导航到目的路由，执行以下操作

   ​	a) 调用**updateRoute**更新路由

   ​	b) 如果存在onComplete， 调用**onComplete**回调函数

   ​	c) 调用**ensureURL**进行导航

   ​	d) 如果还没有执行需要在**完成初始化导航**后需要调用的回调函数，那么就执行回调，这些回调函数通过onReady注册，保存在readyCbs中

   2.2 如果终止导航解析，执行以下操作

   ​	a) 如果存在onAbort ， 调用onAbort 回调函数

   ​	b) 如果还没有执行 **初始化路由解析运行出错**时需要调用的回调函数，那么就执行回调。这些回调函数通过onReady定义，保存在readyErrorCbs中

具体实现如下：

```js
History.prototype.transitionTo = function transitionTo(
    location,
    onComplete,
    onAbort
) {
    var this$1 = this;

    // 获取目的路由对象
    var route = this.router.match(location, this.current);
    this.confirmTransition( // 确认导航
        route,
        function () { // 进行导航
            this$1.updateRoute(route); //更新路由
            onComplete && onComplete(route);
            this$1.ensureURL();//进行导航

            // fire ready cbs once
            if (!this$1.ready) {//没有执行需要在完成初始化导航后需要调用的回调函数
                this$1.ready = true;
                this$1.readyCbs.forEach(function (cb) {
                    cb(route);
                });
            }
        },
        function (err) { // 终止导航
            if (onAbort) { // 如果调用transitionTo 传入的参数中有回调函数
                onAbort(err);
            }
            if (err && !this$1.ready) {//没有执行初始化路由解析运行出错时需要调用的回调函数
                this$1.ready = true;
                this$1.readyErrorCbs.forEach(function (cb) {
                    cb(err);
                });
            }
        }
    );
};
```



##### 调用confirmTransition确认导航

confirmTransition

作用：

​	确认导航

参数：

​	route: 要确认的路由对象

​	onComplete： 一个回调函数，在确认导航后执行

​	onAbort：一个回调函数，在终止导航解析(导航到相同的路由， 或在当前导航完成之前导航到另一个不同的路由)时执行



工作原理：

要解析的目的路由对象和当前路由对象是否相同，

- 如果相同

  调用ensureURL直接导航到当前路由

  调用 abort 来停止导航解析流程

- 如果不同，按照以下顺序解析导航

1. 在失活的组件里调用离开路由守卫***beforeRouteLeave***

2. 调用全局前置守卫***beforeEach***

3. 在重用的组件里调用更新路由守卫 ***beforeRouteUpdate***

4. 在路由配置里调用***beforeEnter***

5. 解析**异步路由组件**

6. 在被激活的组件里调用***beforeRouteEnter***

7. 调用全局解析守卫***beforeResolve***

8. 导航被确认

9. 调用**updateRoute**更新路由

   调用全局后置守卫***afterEach***

10. 调用**ensureURL**进行导航

11. 触发 DOM 更新

12. 在DOM更新之后调用回调函数， 这些回调函数是通过beforeRouteEnter守卫传给 next 的。



具体实现如下：

```js
History.prototype.confirmTransition = function confirmTransition(route, onComplete, onAbort) {
    var this$1 = this;

    var current = this.current;
    var abort = function (err) {
        // after merging https://github.com/vuejs/vue-router/pull/2771 we
        // When the user navigates through history through back/forward buttons
        // we do not want to throw the error. We only throw it if directly calling
        // push/replace. That's why it's not included in isError
        if (!isExtendedError(NavigationDuplicated, err) && isError(err)) {
            if (this$1.errorCbs.length) { // this$1.errorCbs 是通过onError 注册的回调
                this$1.errorCbs.forEach(function (cb) {
                    cb(err);
                });
            } else {
                warn(false, 'uncaught error during route navigation:');
                console.error(err);
            }
        }
        onAbort && onAbort(err);
    };
    // 如果目的路由对象和当前路由对象相同--导航重复
    if (
        isSameRoute(route, current) &&
        // in the case the route map has been dynamically appended to
        route.matched.length === current.matched.length
    ) {
        //那么直接导航到当前路由
        this.ensureURL();
        return abort(new NavigationDuplicated(route))//停止导航解析流程
    }

    var ref = resolveQueue(
        this.current.matched,
        route.matched
    );
    var updated = ref.updated;
    var deactivated = ref.deactivated;
    var activated = ref.activated;

    // 把守卫钩子存在queue中，按顺序执行
    var queue = [].concat(
        // in-component leave guards -- 在失活的组件里调用离开路由守卫beforeRouteLeave
        extractLeaveGuards(deactivated),
        // global before hooks -- 调用全局前置守卫beforeEach
        this.router.beforeHooks,
        // in-component update hooks -- 在重用的组件里调用更新路由守卫beforeRouteUpdate
        extractUpdateHooks(updated),
        // in-config enter guards -- 在路由配置里调用beforeEnter
        activated.map(function (m) { return m.beforeEnter; }),
        // async components -- 解析异步组件
        resolveAsyncComponents(activated)
    );

    this.pending = route;
    var iterator = function (hook, next) {
        if (this$1.pending !== route) {
            return abort()
        }
        try {
            hook(route, current, function (to) {
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
            abort(e);
        }
    };

    runQueue(queue, iterator, function () {//执行完queue 中的钩子函数后
        var postEnterCbs = [];
        var isValid = function () { return this$1.current === route; };
        // wait until async components are resolved before
        // extracting in-component enter guards -- 在被激活的组件里调用beforeRouteEnter
        var enterGuards = extractEnterGuards(activated, postEnterCbs, isValid);
        //this$1.router.resolveHooks 调用全局解析守卫beforeResolve
        var queue = enterGuards.concat(this$1.router.resolveHooks);
        runQueue(queue, iterator, function () {
            if (this$1.pending !== route) {
                return abort() //停止导航解析流程
            }
            this$1.pending = null;
            onComplete(route); //导航被确认，调用在transitionTo中定义的确认后的回调函数
            if (this$1.router.app) {
                this$1.router.app.$nextTick(function () {
                    //在DOM更新之后调用回调函数， 这些回调函数是通过beforeRouteEnter守卫传给 next 的
                    postEnterCbs.forEach(function (cb) {
                        cb();
                    });
                });
            }
        });
    });
};
```



##### 调用**updateRoute**更新路由

更新路由， 并调用全局后置守卫afterEach

```js
History.prototype.updateRoute = function updateRoute(route) {
  var prev = this.current;
  //更新保存当前路由记录的变量 this.current
  this.current = route;
  this.cb && this.cb(route);
  //调用全局后置守卫afterEach
  this.router.afterHooks.forEach(function (hook) {
    hook && hook(route, prev);
  });
};
```



### 完整导航解析流程

1. 导航被触发，进入**transitionTo** 函数

2. 获取目的路由对象route

3. 调用**confirmTransition** 确认导航目的路由对象route

4. 在失活的组件里调用离开路由守卫***beforeRouteLeave***

5. 调用全局前置守卫***beforeEach***

6. 在重用的组件里调用更新路由守卫 ***beforeRouteUpdate***

7. 在路由配置里调用***beforeEnter***

8. 解析**异步路由组件**

9. 在被激活的组件里调用***beforeRouteEnter***

10. 调用全局解析守卫***beforeResolve***

11. 确认要导航到目的路由

12. 调用**updateRoute**更新路由

    调用全局后置守卫***afterEach***

13. 调用**ensureURL**进行导航

14. 触发 DOM 更新

15. 在DOM更新之后调用回调函数， 这些回调函数是通过beforeRouteEnter守卫传给 next 的。