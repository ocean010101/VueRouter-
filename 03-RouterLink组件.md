## RouterLink组件

VueRouter支持使用VueRouter实例方法进行编程式导航， 也支持使用RouterLink组件声明式导航。

RouterLink组件本质上也是使用VueRouter实例的push和replace 方法进行导航

具体实现如下：

```js
// work around weird flow bug
var toTypes = [String, Object];
var eventTypes = [String, Array];

var noop = function () { };

//初始化props
var Link = {
  name: 'RouterLink',
  props: {
    to: {
      type: toTypes,
      required: true
    },
    tag: {
      type: String,
      default: 'a'
    },
    exact: Boolean,
    append: Boolean,
    replace: Boolean,
    activeClass: String,
    exactActiveClass: String,
    event: {
      type: eventTypes,
      default: 'click'
    }
  },
  render: function render(h) {
    var this$1 = this;

    var router = this.$router; //全局VueRouter实例
    var current = this.$route; //当前路由对象
    var ref = router.resolve(
      this.to,
      current,
      this.append
    );
    var location = ref.location;
    var route = ref.route;
    var href = ref.href;

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

    var handler = function (e) { // 当出触发绑定此函数的事件时，调用此函数进行导航
      if (guardEvent(e)) {
        if (this$1.replace) { // 如果replace prop 为true
          // 调用VueRouter实例的replace方法进行导航，替换history 栈中当前记录
          router.replace(location, noop);
        } else {
          // 调用VueRouter实例的push方法进行导航， 向 history 栈添加一个新的记录
          router.push(location, noop);
        }
      }
    };

    //2. 把handler处理函数绑定到通过event prop定义的事件中。
    var on = { click: guardEvent };
    if (Array.isArray(this.event)) {
      this.event.forEach(function (e) {
        on[e] = handler;
      });
    } else {
      on[this.event] = handler;
    }

    var data = { class: classes };

    // 3. 处理带作用域插槽的模板
    var scopedSlot =
      !this.$scopedSlots.$hasNormal &&
      this.$scopedSlots.default &&
      this.$scopedSlots.default({
        href: href,
        route: route,
        navigate: handler,
        isActive: classes[activeClass],
        isExactActive: classes[exactActiveClass]
      });

    if (scopedSlot) {
      if (scopedSlot.length === 1) {
        return scopedSlot[0]
      } else if (scopedSlot.length > 1 || !scopedSlot.length) {
        {
          warn(
            false,
            ("RouterLink with to=\"" + (this.props.to) + "\" is trying to use a scoped slot but it didn't provide exactly one child.")
          );
        }
        return scopedSlot.length === 0 ? h() : h('span', {}, scopedSlot)
      }
    }

    //4. 处理模板的事件和属性
    if (this.tag === 'a') { // 如果是<a>元素，更新监听的事件和属性href
      data.on = on;
      data.attrs = { href: href };
    } else { // 如果不是<a>元素
      // find the first <a> child and apply listener and href
      var a = findAnchor(this.$slots.default);
      if (a) { // 如果在被插槽分发的内容中查找到<a>元素
        // in case the <a> is a static node
        a.isStatic = false;
        var aData = (a.data = extend({}, a.data));
        aData.on = aData.on || {}; // 更新元素监听的事件
        // transform existing events in both objects into arrays so we can push later
        for (var event in aData.on) {
          var handler$1 = aData.on[event];
          if (event in on) {
            aData.on[event] = Array.isArray(handler$1) ? handler$1 : [handler$1];
          }
        }
        // append new listeners for router-link
        for (var event$1 in on) {
          if (event$1 in aData.on) {
            // on[event] is always a function
            aData.on[event$1].push(on[event$1]);
          } else {
            aData.on[event$1] = handler;
          }
        }
        //更新元素的属性href
        var aAttrs = (a.data.attrs = extend({}, a.data.attrs));
        aAttrs.href = href;
      } else {
        // doesn't have <a> child, apply listener to self
        // 没有在被插槽分发的内容中查找到<a>子节点， 更新这个非<a>元素的监听的事件
        data.on = on;
      }
    }
    // 5. 创建RouterLink模板的虚拟node
    return h(this.tag, data, this.$slots.default)
  }
};
```

#### props

通过源代码可以看到RouterLink组件的props如下：

```js
var toTypes = [String, Object];
var eventTypes = [String, Array];

props: {
  to: {
      type: toTypes,
      required: true
  },
  tag: {
      type: String,
      default: 'a'
  },
  exact: Boolean,
  append: Boolean,
  replace: Boolean,
  activeClass: String,
  exactActiveClass: String,
  event: {
      type: eventTypes,
      default: 'click'
  }
},

```



示例：

```vue
<router-link :to="{name : 'home'}" exact>Home</router-link>
<router-link :to="{name : 'faq'}">FAQ</router-link>
```



```js
const routes = [
  { path: '/', name: 'home', component: Home },
  { path: '/faq', name: 'faq', component: FAQ },
  // ...
]

const router = new VueRouter({
  routes,
  mode: 'history',
  // ...
})
```



##### to

作用：

​	表示目标路由的链接。

类型：

​	string | Object

​	一个字符串或者是描述目标位置的对象

代码：

```js
var toTypes = [String, Object];
to: {
    type: toTypes,
    required: true // 必须存在
},
```



##### tag

作用：

​	 设置RouterLink 组件模板的HTML元素

类型：

​	String

默认值：

​	'a'

代码：

```js
tag: {
    type: String,
    default: 'a' // 默认值:  'a'
},
```

示例：

```vue
<router-link :to="{name : 'faq'}">FAQ</router-link>
```

默认情况下 渲染成<a>元素：

```html
<a data-v-575eba22="" href="/faq" class="">FAQ</a>
```

我们设置tag，期望把<router-link />渲染成button

```vue
<router-link :to="{name : 'faq'}" tag="button">FAQ</router-link> 
```

渲染成<button>元素：

```html
<button data-v-575eba22="" class="">FAQ</button>
```



##### event

作用：

​	声明可以用来触发导航的事件。

类型: 

​	String| Array<string>

默认值:

​	click

代码：

```
var eventTypes = [String, Array];

event: {
    type: eventTypes,
    default: 'click'
}
```



当触发事件时，调用事件处理函数进行导航

```js
var handler = function (e) { // 当出触发绑定此函数的事件时，调用此函数进行导航
  if (guardEvent(e)) {
      if (this$1.replace) { // 如果replace prop 为true
          // 调用VueRouter实例的replace方法进行导航，替换history 栈中当前记录
          router.replace(location, noop); 
      } else { 
          // 调用VueRouter实例的push方法进行导航， 向 history 栈添加一个新的记录
          router.push(location, noop);
      }
  }
};
```



##### replace

作用：

​	设置replace prop， 当触发导航时， 调用VueRouter实例的replace方法进行导航，替换history 栈中当前记录

类型：

​	Boolean

默认值：

​	false

​	没有设置replace prop， 当触发导航时，调用VueRouter实例的push方法进行导航， 向 history 栈添加一个新的记录



##### append

作用：

​	在当前路径上附加相对路径

​	例如，我们从 `/a` 导航到一个相对路径 `b`，如果没有配置 `append`，则路径为 `/b`，如果配了，则为 `/a/b`

类型：

​	Boolean

默认值：

​	false



##### exact

作用：

​	设置active class 匹配行为是精准匹配

类型：

​	Boolean

默认值：

​	false

​	默认active class 匹配行为是**包含匹配**。



示例：

​	默认情况， 点击FAQ链接时，检查匹配时，被查找的字符串为/faq，  把path对应的值作为正则表达式进行匹配，可以匹配到/faq的表达式都会被激活。所以当点击FAQ链接时， “/” 或 “/faq/” 都会被匹配.  

```vue
<router-link :to="{name : 'home'}">Home</router-link>
<router-link :to="{name : 'faq'}">FAQ</router-link>
```

点击FAQ链接， path为“/” 的链接也会被匹配， 会添加名字为"router-link-active"的类

```html
<a data-v-575eba22="" href="/" class="router-link-active">Home</a>
<a data-v-575eba22="" href="/faq" class="router-link-exact-active router-link-active">FAQ</a>
```



如果想精准匹配，点击FAQ链接时 只匹配到“/faq”，不要匹配“/”，  需要在path为“/”<router-link />上使用exact属性

```vue
<router-link :to="{name : 'home'}" exact>Home</router-link>
<router-link :to="{name : 'faq'}">FAQ</router-link>
```

```html
<a data-v-575eba22="" href="/" class="">Home</a>
<a data-v-575eba22="" href="/faq" class="router-link-exact-active router-link-active">FAQ</a>
```



##### active-class

作用：

​	当链接被匹配时，配置链接处于活动状态时使用的 CSS 类名。

默认值可以通过VueRouter的构建选项 **linkActiveClass** 进行全局配置。

类型：

​	String

默认值：

​	“router-link-active”

示例：

```vue
<router-link :to="{name : 'faq'}">FAQ</router-link>
```

当/faq被匹配时，默认情况下会添加名字为“router-link-exact-active”的类和名字为”router-link-active“的类

```html
<a data-v-575eba22="" href="/faq" class="router-link-exact-active router-link-active">FAQ</a>
```

如果设置active-class 为"faq-active"

```vue
<router-link :to="{name : 'faq'}" active-class="faq-active" >FAQ</router-link> 
```

那么路由被匹配时会添加名字为faq-active的类

```html
<a data-v-575eba22="" href="/faq" class="router-link-exact-active faq-active">FAQ</a>
```

##### exact-active-class

作用：

​	当链接被**精准**匹配时， 配置链接处于活动状态时使用的 CSS 类名。

默认值可以通过VueRouter的构造选项 **linkExactActiveClass** 进行全局配置。

类型：

​	String

默认值：

​	”router-link-exact-active“

示例：

如果设置exact-active-class 为"faq-exact-active"

```vue
<router-link :to="{name : 'faq'}" exact-active-class="faq-exact-active">FAQ</router-link>
```

那么路由被匹配时会添加名字为”faq-exact-active“的类

```html
<a data-v-575eba22="" href="/faq" class="faq-exact-active router-link-active">FAQ</a>
```



#### 渲染过程render

1.  处理当路由被激活时<router-link /＞渲染的模板上需要添加的类--data.class
2.  把handler处理函数绑定到通过event prop定义的事件-data.on
3.  处理带作用域插槽的模板
4.  处理模板的事件和属性--data.on和data.attrs
5.  创建RouterLink模板的虚拟node

具体实现如下：

```js
render: function render(h) {
  var this$1 = this;

  var router = this.$router; //全局VueRouter实例
  var current = this.$route; //当前路由对象
  var ref = router.resolve(
      this.to,
      current,
      this.append
  );
  var location = ref.location;
  var route = ref.route;
  var href = ref.href;

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

  var handler = function (e) { // 当出触发绑定此函数的事件时，调用此函数进行导航
      if (guardEvent(e)) {
          if (this$1.replace) { // 如果replace prop 为true
              // 调用VueRouter实例的replace方法进行导航，替换history 栈中当前记录
              router.replace(location, noop); 
          } else { 
              // 调用VueRouter实例的push方法进行导航， 向 history 栈添加一个新的记录
              router.push(location, noop);
          }
      }
  };

  //2. 把handler处理函数绑定到通过event prop定义的事件中。
  var on = { click: guardEvent };
  if (Array.isArray(this.event)) {
      this.event.forEach(function (e) {
          on[e] = handler;
      });
  } else {
      on[this.event] = handler;
  }

  var data = { class: classes };

  // 3. 处理带作用域插槽的模板
  var scopedSlot =
      !this.$scopedSlots.$hasNormal &&
      this.$scopedSlots.default &&
      this.$scopedSlots.default({
          href: href,
          route: route,
          navigate: handler,
          isActive: classes[activeClass],
          isExactActive: classes[exactActiveClass]
      });

  if (scopedSlot) {
      if (scopedSlot.length === 1) {
          return scopedSlot[0]
      } else if (scopedSlot.length > 1 || !scopedSlot.length) {
          {
              warn(
                  false,
                  ("RouterLink with to=\"" + (this.props.to) + "\" is trying to use a scoped slot but it didn't provide exactly one child.")
              );
          }
          return scopedSlot.length === 0 ? h() : h('span', {}, scopedSlot)
      }
  }

  //4. 处理模板的事件和属性
  if (this.tag === 'a') { // 如果是<a>元素，更新监听的事件和属性href
      data.on = on; 
      data.attrs = { href: href };
  } else { // 如果不是<a>元素
      // find the first <a> child and apply listener and href
      var a = findAnchor(this.$slots.default);
      if (a) { // 如果在被插槽分发的内容中查找到<a>元素
          // in case the <a> is a static node
          a.isStatic = false;
          var aData = (a.data = extend({}, a.data));
          aData.on = aData.on || {}; // 更新元素监听的事件
          // transform existing events in both objects into arrays so we can push later
          for (var event in aData.on) {
              var handler$1 = aData.on[event];
              if (event in on) {
                  aData.on[event] = Array.isArray(handler$1) ? handler$1 : [handler$1];
              }
          }
          // append new listeners for router-link
          for (var event$1 in on) {
              if (event$1 in aData.on) {
                  // on[event] is always a function
                  aData.on[event$1].push(on[event$1]);
              } else {
                  aData.on[event$1] = handler;
              }
          }
			//更新元素的属性href
          var aAttrs = (a.data.attrs = extend({}, a.data.attrs));
          aAttrs.href = href;
      } else {
          // doesn't have <a> child, apply listener to self
          // 没有在被插槽分发的内容中查找到<a>子节点， 更新这个非<a>元素的监听的事件
          data.on = on;
      }
  }
// 5. 创建RouterLink模板的虚拟node
  return h(this.tag, data, this.$slots.default)
}
```



#### 工作流程

1. 点击模板， 触发模板中监听的事件，

2. 调用事件处理函数进行导航

   如果组件replace prop为true，那么调用VueRouter实例的replace方法导航到新的路由，并且在history 栈中替换当前history 条目。

   如果组件replace prop为false，那么调用VueRouter实例的puhs方法导航到新的路由，并把这条新的history 条目添加到history 栈中。

