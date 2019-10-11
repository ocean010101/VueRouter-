## RouterView组件

### Props

#### name

​	类型: 

​		string

​	默认值: 

​		"default"

工作原理：

​	默认情况下，RouterView会获取route.matched.components['default']的组件，渲染到视图中。

​	如果设置了name  prop，RouterView会获取route.matched.components[name]的组件，渲染到视图中。

具体实现：

```js

var View = {
    name: 'RouterView',
    functional: true,
    props: {
        name: {
            type: String,
            default: 'default'
        }
    },
    render: function render(_, ref) { // 渲染函数
        var props = ref.props;
        var children = ref.children;
        var parent = ref.parent;
        var data = ref.data;

        // used by devtools to display a router-view badge
        data.routerView = true;

        // directly use parent context's createElement() function
        // so that components rendered by router-view can resolve named slots
        var h = parent.$createElement;
        var name = props.name;
        var route = parent.$route;
        var cache = parent._routerViewCache || (parent._routerViewCache = {});

        // determine current view depth, also check to see if the tree
        // has been toggled inactive but kept-alive.
        var depth = 0;
        var inactive = false;
        while (parent && parent._routerRoot !== parent) {
            var vnodeData = parent.$vnode && parent.$vnode.data;
            if (vnodeData) {
                if (vnodeData.routerView) {
                    depth++;
                }
                if (vnodeData.keepAlive && parent._inactive) {
                    inactive = true;
                }
            }
            parent = parent.$parent;
        }
        data.routerViewDepth = depth;

        // render previous view if the tree is inactive and kept-alive
        if (inactive) {
            return h(cache[name], data, children)
        }

        var matched = route.matched[depth];
        // render empty node if no matched route
        if (!matched) { //如果没有匹配的路由，则呈现空节点
            cache[name] = null;
            return h()
        }

        var component = cache[name] = matched.components[name];


        //附加实例注册钩子这将在实例的注入生命周期钩子中调用 
        //在创建Vue实例的生命周期钩子beforeCreate和destroyed中调用
        data.registerRouteInstance = function (vm, val) {
            // val could be undefined for unregistration
            var current = matched.instances[name];
            if (
                (val && current !== vm) ||
                (!val && current === vm)
            ) {
                matched.instances[name] = val;
            }
        }

            // also register instance in prepatch hook
            // in case the same component instance is reused across different routes
        	//如果同一个组件实例在不同的路由中被重用， 也在prepatch hook中注册实例
            ; (data.hook || (data.hook = {})).prepatch = function (_, vnode) {
                matched.instances[name] = vnode.componentInstance;
            };

        // 在init hook中注册实例，以防路由更改时kept-alive组件处于活动状态
        data.hook.init = function (vnode) {
            if (vnode.data.keepAlive &&
                vnode.componentInstance &&
                vnode.componentInstance !== matched.instances[name]
            ) {
                matched.instances[name] = vnode.componentInstance;
            }
        };

        // resolve props
        var propsToPass = data.props = resolveProps(route, matched.props && matched.props[name]);
        if (propsToPass) {
            // clone to prevent mutation
            propsToPass = data.props = extend({}, propsToPass);
            // pass non-declared props as attrs
            var attrs = data.attrs = data.attrs || {};
            for (var key in propsToPass) {
                if (!component.props || !(key in component.props)) {
                    attrs[key] = propsToPass[key];
                    delete propsToPass[key];
                }
            }
        }
		// 创建组件的虚拟node
        return h(component, data, children)
    }
};

//解析props
function resolveProps(route, config) {
    switch (typeof config) {
        case 'undefined':
            return
        case 'object':
            return config
        case 'function':
            return config(route)
        case 'boolean':
            return config ? route.params : undefined
        default:
            {
                warn(
                    false,
                    "props in \"" + (route.path) + "\" is a " + (typeof config) + ", " +
                    "expecting an object, function or boolean."
                );
            }
    }
}
```







