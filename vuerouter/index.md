# vue-router

前端单页应用路由，常用有hash和history模式。

## hash && history

**Hash模式：** url上/#/后的参数为hash。通过监听hashchange时间监听url变化。

**History模式：** 通过h5新属性 history 实现，通过代码pushState, replaceState实现手动推入history.state实现路由的切换。当然还有用户操作浏览器的前进后退等按钮需要实现路由的监听，这是监听popstate事件来实现的。注意：pushState, replaceState不会触发popstate事件，history.back(), history.go()可以触发。

history模式需要服务端配置转发到index.html。

个人思考：看源码发现hash模式并不一定是监听hashchange实现的。

```js
    // vue2版本 vue-router源码位置 src/history/hash.js line:49
    const eventType = supportsPushState ? 'popstate' : 'hashchange'
    window.addEventListener(
      eventType,
      handleRoutingEvent
    )

    // supportsPushState是判断浏览器是否支持popstate(即判断是否支持html5history)
    // 源码位置 src/utils/push-state.js line: 8

    export const supportsPushState =
        inBrowser &&
        (function () {
            const ua = window.navigator.userAgent

            if (
            (ua.indexOf('Android 2.') !== -1 || ua.indexOf('Android 4.0') !== -1) &&
            ua.indexOf('Mobile Safari') !== -1 &&
            ua.indexOf('Chrome') === -1 &&
            ua.indexOf('Windows Phone') === -1
            ) {
            return false
            }

            return window.history && typeof window.history.pushState === 'function'
        })()


        // vue3 版本直接使用popstate 并不作兼容hashchange。个人理解是随着浏览器发展已经都支持html5history了。所以没必要兼容。
        // vue3版本 router 源码位置 src/history/hash.ts line:31
        export function createWebHashHistory(base?: string): RouterHistory {
            // Make sure this implementation is fine in terms of encoding, specially for IE11
            // for `file://`, directly use the pathname and ignore the base
            // location.pathname contains an initial `/` even at the root: `https://example.com`
            base = location.host ? base || location.pathname + location.search : ''
            // allow the user to provide a `#` in the middle: `/base/#/app`
            if (!base.includes('#')) base += '#'

            if (__DEV__ && !base.endsWith('#/') && !base.endsWith('#')) {
                warn(
                `A hash base must end with a "#":\n"${base}" should be "${base.replace(
                    /#.*$/,
                    '#'
                )}".`
                )
            }
            return createWebHistory(base)
        }

        // createWebHistory 源码位置 src/history/html5.ts
        // 最终监听到是popstate  line:146
        window.addEventListener('popstate', popStateHandler)
```

不确定我是不是对的，希望看到这篇文章的大佬批评指正，万分感谢。

## 路由守卫

参考[路由守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

### 全局钩子函数

- beforeEach
- beforeResolve
- afterEach

### 路由钩子

- beforeEnter

### 组件内钩子

- beforeRouteEnter
- beforeRouteUpdate
- beforeRouteLeave

### 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫(2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫(2.5+)。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

```js
    // TODO 写demo验证执行顺序
```

## router 实现的原理

从现象看本质。首先来看看router是如何使用的。

```js
import { createApp } from 'vue'
import { createRouter, createWebHistory } from 'vue-router'
import App from './App.vue'
import router from './router.ts'
createApp(App).use(router).mount('#app')
```

```js
import { createRouter, createWebHistory } from 'vue-router'
const routes = [
    { path: '/home', redirect: '/' },
    {
      path: '/children',
      name: 'WithChildren',
      component: Nested,
      children: [
        { path: '', alias: 'alias', name: 'default-child', component: Nested },
        { path: 'a', name: 'a-child', component: Nested },
        {
          path: 'b',
          name: 'b-child',
          component: Nested,
          children: [
            { path: '', component: Nested },
            { path: 'a2', component: Nested },
            { path: 'b2', component: Nested },
          ],
        },
      ],
    },
]
export default createRouter({
    history: createWebHistory(),
    routes,
})
```

首先知道router返回一个对像，内部有install方法（因为vue插件必须有install方法才可在use(xxx)的时候挂在到vue实例上），除此之外还提供了一些操作路由的方法入push，replace，go，等等，还有全局收尾beforeEach，beforeResolve，afterEach。少不了的当前路由currentRoute。还提供了动态操作路由表的方法addRoute，removeRoute等，还有初始化参数option。在install时还注册了全局组件router-view和router-link

```js
 // 源码位置src/router.ts line:1151
 const router: Router = {
    currentRoute,

    addRoute,
    removeRoute,
    hasRoute,
    getRoutes,
    resolve,
    options,

    push,
    replace,
    go,
    back: () => go(-1),
    forward: () => go(1),

    beforeEach: beforeGuards.add,
    beforeResolve: beforeResolveGuards.add,
    afterEach: afterGuards.add,

    onError: errorHandlers.add,
    isReady,

    install(app: App) {
      const router = this
      app.component('RouterLink', RouterLink)
      app.component('RouterView', RouterView)

      app.config.globalProperties.$router = router
      Object.defineProperty(app.config.globalProperties, '$route', {
        enumerable: true,
        get: () => unref(currentRoute),
      })

      // this initial navigation is only necessary on client, on server it doesn't
      // make sense because it will create an extra unnecessary navigation and could
      // lead to problems
      if (
        isBrowser &&
        // used for the initial navigation client side to avoid pushing
        // multiple times when the router is used in multiple apps
        !started &&
        currentRoute.value === START_LOCATION_NORMALIZED
      ) {
        // see above
        started = true
        push(routerHistory.location).catch(err => {
          if (__DEV__) warn('Unexpected error when starting the router:', err)
        })
      }

      const reactiveRoute = {} as {
        [k in keyof RouteLocationNormalizedLoaded]: ComputedRef<
          RouteLocationNormalizedLoaded[k]
        >
      }
      for (const key in START_LOCATION_NORMALIZED) {
        // @ts-expect-error: the key matches
        reactiveRoute[key] = computed(() => currentRoute.value[key])
      }

      app.provide(routerKey, router)
      app.provide(routeLocationKey, reactive(reactiveRoute))
      app.provide(routerViewLocationKey, currentRoute)

      const unmountApp = app.unmount
      installedApps.add(app)
      app.unmount = function () {
        installedApps.delete(app)
        // the router is not attached to an app anymore
        if (installedApps.size < 1) {
          // invalidate the current navigation
          pendingLocation = START_LOCATION_NORMALIZED
          removeHistoryListener && removeHistoryListener()
          removeHistoryListener = null
          currentRoute.value = START_LOCATION_NORMALIZED
          started = false
          ready = false
        }
        unmountApp()
      }

      if ((__DEV__ || __FEATURE_PROD_DEVTOOLS__) && isBrowser) {
        addDevtools(app, router, matcher)
      }
    },
  }

```

再从这里切入分析。他是从createRouter函数返回的，所以看看它的具体实现。

createRouter(options)接受options参数，会先实例话一个matcher，用来存储options.routes路由参数。源码位置src/router.ts line:355。

```js
export function createRouter(options: RouterOptions): Router {
  // matcher是用于存储routes将每个路由配置对应一个数据，用于后续与路由match，如果match正确则展示对应的route.component
  const matcher = createRouterMatcher(options.routes, options)
    ......
  const router: Router = {
    ......
  }

  return router
}
```

router中有个currentRoute参数，表示当前路由，是个对象，内部还有matched对象数组。router-view是有多层级的，matched则是保存着match对的所有数据。

## router matcher 是如何做的

从上可知createRouterMatcher(options.routes, options)传入开发者配置的options.routes，和全局options。返回的是一个matcher对象。matcher对象应该有上述router提供的addRoute等方法，因为需要通过router操作addRoute从而操作到matcher内部的routes数据，不出意外应该也会有getRouteMatcher方法用来获取路由配对的数据（实际上是叫getRecordMatcher），

matcher实际通过map实现。以开发者自定义routes的时候给路由命名的name属性来做唯一值。

```js
/**
 * 源码位置 src/matcher/index.ts line:58
 * 会先出初始化一个数组matchers和一个matcherMap
 * matchers 为了给后续提供getRoutes的返回值。避免后续getRoutes的时候再计算（用空间换时间的方式提升效率）
 * 
 * */ 

  const matchers: RouteRecordMatcher[] = []
  const matcherMap = new Map<RouteRecordName, RouteRecordMatcher>()

  // src/matcher/index.ts line:318
  // add initial routes 会将入参routes便利并添加到matchers和matcherMap中。
  routes.forEach(route => addRoute(route))

  // 因为开发者配置的routes对象中有children（子路由）所以必然 addRoute内部会判断是否有子路由，有子路由再递归调用addRoute。line:159
```

## router-link 的实现原理

在初始化的时候已经provide了currentRoute 和 router，再组件中inject获取到这两个数据。
router-link可自定义，就是利用default slot实现，会判断props.custom 为true则是自定义，会渲染slot.default()的内容。
根据props.to传入参数，调用navigate跳转，navigate内部则是通过一些逻辑操作最后条用router.push或者router.replace。router哪里来呢？没错就是前面说的inject(routerKey)。currentRoute则是用来和参数传入的props.to对比判断是否是相同路由
.....

## router-view 的实现原理

首先router-view是可以多层调用的。所以难点是多层级如何匹配。当每次路由跳转的时候都会进行路由解析，matcher.resolve的时候会返回一个routeRecord对象，内部有一个属性metched是个对象数组，存储的就是所有所有层级配对的路由，按照深度存储，index 0 最外层。

router-view的实现就依赖了这个属性。router-view内部有个变量depth存储当前匹配的层数，

当代码执行遇到router-view的时候，内部通过currentRoute获取当前的路由，路由内部的matched就是我们所需要的路由数据。首先获取matched[0].component并执行渲染，这时候需要depth深度+1。很绝妙的是，depth是从router-view内部provide出去的，也是在内部inject的。也就是上一次执行完成之后provide(depth+1)。

后续执行再次遇到router-view的时候inject(depth)获取当前所在深度，然后再执行渲染matched[depth].component。如此循环知道matched所有数据都执行渲染完成。
