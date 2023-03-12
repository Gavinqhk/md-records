# vue3源码学习

秉承着从现象看本质的原则来学习vue3源码。首先来看看vue3是怎么使用的。

```js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
// 注册插件，组件，指令，挂载等等
// app.use(plugins)
// app.component(component)
// app.directive(directive)
app.mount('#app')

```

首先看看createApp做了啥。

```js
// packages/runtime-core/src/apiCreateApp.ts
//line: 148
export function createAppContext(): AppContext {
  return {
    app: null as any,
    config: {
      isNativeTag: NO,
      performance: false,
      globalProperties: {},
      optionMergeStrategies: {},
      errorHandler: undefined,
      warnHandler: undefined,
      compilerOptions: {}
    },
    mixins: [],
    components: {},
    directives: {},
    provides: Object.create(null),
    optionsCache: new WeakMap(),
    propsCache: new WeakMap(),
    emitsCache: new WeakMap()
  }
}
//line: 177
export function createAppAPI<HostElement>(
  render: RootRenderFunction,
  hydrate?: RootHydrateFunction
): CreateAppFunction<HostElement> {
    return function createApp(rootComponent, rootProps = null){
        //......
        // 记录是否挂载
        let isMounted = false
        // 创建一个context对象
        const context = createAppContext()
        // 创建context之后，通过context创建app实例
        const app: App = (context.app = {
            _uid: uid++,
            _component: rootComponent as ConcreteComponent,
            _props: rootProps,
            _container: null,
            _context: context,
            _instance: null,
            version,
            get config() {
                return context.config
            },
            set config(v) {
                if (__DEV__) {
                    warn(
                        `app.config cannot be replaced. Modify individual options instead.`
                    )
                }
            },
            // 提供下面方法给实例，为实例添加注册插件，minxin,component, 指令等方法，以及挂载卸载provide方法。
            use(plugin: Plugin, ...options: any[]) {},
            mixin(mixin: ComponentOptions) {},
            component(name: string, component?: Component) {},
            directive(name: string, directive?: Directive) {},
            mount(
                rootContainer: HostElement,
                isHydrate?: boolean,
                isSVG?: boolean
            ): any {},
            unmount() {},
            provide(key, value) {},
        })
        return app
    }
}
```

接下来看看```app.mount(App)```做了些啥

```js

    mount(
        rootContainer: HostElement,
        isHydrate?: boolean,
        isSVG?: boolean
    ): any {
        // 判断是否记挂载
        if(!isMounted) {
            // ......
            // 创建虚拟节点
            const vnode = createVNode(
                rootComponent as ConcreteComponent,
                rootProps
            )
            // 将实例上下文存储在vnode上。
            vnode.appContext = context
            //......
            // 判断是否是服务器渲染
            if (isHydrate && hydrate) {
                // 服务器渲染相关注入函数
                hydrate(vnode as VNode<Node, Element>, rootContainer as any)
            } else {
                // render内部是走patch方法
                render(vnode, rootContainer, isSVG)
            }
            isMounted = true
        } else {
            warn("....")
        }
    },
```

hydrate是关于服务器渲染的暂时先不关注，我们来看看render方法处理了那些事情

```js
// packages/runtime-core/src/renderer.ts
//line: 2326
const render: RootRenderFunction = (vnode, container, isSVG) => {
    if (vnode == null) {
        if (container._vnode) {
        unmount(container._vnode, null, null, true)
        }
    } else {
        patch(container._vnode || null, vnode, container, null, null, null, isSVG)
    }
    flushPostFlushCbs()
    container._vnode = vnode
}
```

再看看patch方法是干啥事的, 如下，是对比两个节点的，n1是旧节点，n2是新节点。根据n2节点不同的类型来进行不同的处理。

```js
  const patch: PatchFn = (
    n1,
    n2,
    container,
    anchor = null,
    parentComponent = null,
    parentSuspense = null,
    isSVG = false,
    slotScopeIds = null,
    optimized = __DEV__ && isHmrUpdating ? false : !!n2.dynamicChildren
  ) => {
    if (n1 === n2) {
      return
    }

    // patching & not same type, unmount old tree
    if (n1 && !isSameVNodeType(n1, n2)) {
      anchor = getNextHostNode(n1)
      unmount(n1, parentComponent, parentSuspense, true)
      n1 = null
    }

    if (n2.patchFlag === PatchFlags.BAIL) {
      optimized = false
      n2.dynamicChildren = null
    }

    const { type, ref, shapeFlag } = n2
    switch (type) {
      case Text:
        processText(n1, n2, container, anchor)
        break
      case Comment:
        processCommentNode(n1, n2, container, anchor)
        break
      case Static:
        if (n1 == null) {
          mountStaticNode(n2, container, anchor, isSVG)
        } else if (__DEV__) {
          patchStaticNode(n1, n2, container, isSVG)
        }
        break
      case Fragment:
        processFragment(
          n1,
          n2,
          container,
          anchor,
          parentComponent,
          parentSuspense,
          isSVG,
          slotScopeIds,
          optimized
        )
        break
      default:
        if (shapeFlag & ShapeFlags.ELEMENT) {
          processElement(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized
          )
        } else if (shapeFlag & ShapeFlags.COMPONENT) {
          processComponent(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense, 
            isSVG,
            slotScopeIds,
            optimized
          )
        } else if (shapeFlag & ShapeFlags.TELEPORT) {
          ;(type as typeof TeleportImpl).process(
            n1 as TeleportVNode,
            n2 as TeleportVNode,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized,
            internals
          )
        } else if (__FEATURE_SUSPENSE__ && shapeFlag & ShapeFlags.SUSPENSE) {
          ;(type as typeof SuspenseImpl).process(
            n1,
            n2,
            container,
            anchor,
            parentComponent,
            parentSuspense,
            isSVG,
            slotScopeIds,
            optimized,
            internals
          )
        } else if (__DEV__) {
          warn('Invalid VNode type:', type, `(${typeof type})`)
        }
    }

    // set ref
    if (ref != null && parentComponent) {
      setRef(ref, n1 && n1.ref, parentSuspense, n2 || n1, !n2)
    }
  }
```

patch时候会判断vnode的类型，走switch语法块，执行对应vnode.type类型的方法。

- 当类型为Text走processText(n1, n2, container, anchor)方法，判断是否有n1（旧节点）无则直接hostInsert()通过n2创建textnode再插入（insertBefore）父元素container。
