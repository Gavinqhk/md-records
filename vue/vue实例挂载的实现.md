# vue实例挂载的实现

>涉及到$mount, mountComponent, render, update
render -->_createElement create vnode
update -->_patch   finally create dom

Vue中我们通过$mount实例方法去挂载vm

```js
const mount = Vue.prototype.$mount;//先将vue原型上到$mount函数缓存
Vue.prototype.$mount = function(
    el?: string | Element,
    hydrating?: boolean;
): Component {
    el = el && query(el);

    if (el === document.body || el === document.documentElement) {
        process.env.NODE_ENV !== 'production' && warn(
        `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
        )
        return this
    }

    const options = this.$options;
    //resolve template/el and convert to render function
    if(!options.render) {
        let template = options.template;
        if(template) {
            if(typeof template === 'string') {
                if (el === document.body || el === document.documentElement) {
                    process.env.NODE_ENV !== 'production' && warn(
                    `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
                    )
                    return this
                }
            } else if (template.nodeType) {
                template = template.innerHtml;
            } else {
                if (process.env.NODE_ENV !== 'production') {
                warn('invalid template option:' + template, this)
                }
                return this
            }
        } else if (el) {
            template = getOuterHTML(el)
        }
        if (template) {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
            mark('compile')
        }

        const { render, staticRenderFns } = compileToFunctions(template, {
            shouldDecodeNewlines,
            shouldDecodeNewlinesForHref,
            delimiters: options.delimiters,
            comments: options.comments
        }, this)
        options.render = render
        options.staticRenderFns = staticRenderFns

        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
            mark('compile end')
            measure(`vue ${this._name} compile`, 'compile', 'compile end')
        }
        }
    }
    return mount.call(this, el, hydrating)
}

```

首先将$mount缓存，再重新定义该方法。它对el做了限制，Vue不能挂载到body，html这样到根结点上。  
接下来到关键到逻辑：如果没有定义render函数，则将el或template字符串转换成render方法，（render方法将el生成虚拟节点virtualDom）。无论是写了el或template，还是.vue单文件方式开发组件，最终都会转换成render方法。这个过程叫做vue‘在线编译’过程，它是调用compileToFunctions方法实现到。
最后调用之前缓存到mount方法挂载。

## 原型上到$mount方法

```js
//public mount method
Vue.prototype.$mount = function(
    el?:string | Element,
    hydrating?: boolean
):Component {
    el = el && isBrowser ? query(el) : undefined;
    return mountComponent(this, el, hydrating);
}
```

mount 方法实际上会调用mountComponent方法。

```js
export function mountComponent(
    vm: Component,
    el?: Element,
    hydrating?: boolean
): Component {
    vm.$el = el;
    if(!vm.$options.render) {
        vm.$options.render = createEmptyVNode
        if (process.env.NODE_ENV !== 'production') {
            /* 错误兼容写法 */
            if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
                vm.$options.el || el) {
                warn(
                'You are using the runtime-only build of Vue where the template ' +
                'compiler is not available. Either pre-compile the templates into ' +
                'render functions, or use the compiler-included build.',
                vm
                )
            } else {
                warn(
                'Failed to mount component: template or render function not defined.',
                vm
                )
            }
        }
    }

    callHook(vm, 'beforeMount'); // lifeCycle
    let updataComponent;
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        updataComponent = () => {
            const name = vm._name;
            const id = vm._uid;
            const startTag = `vue-perf-start:${id}`;
            const endTag = `vue-perf-end:${id}`;
            mark(startTag)
            const vnode = vm._render()
            mark(endTag)
            measure(`vue ${name} render`, startTag, endTag)

            mark(startTag)
            vm._update(vnode, hydrating)
            mark(endTag)
            measure(`vue ${name} patch`, startTag, endTag)
        }
    } else {
        updataComponent = () => {
            vm._updata(vm._render, hydrating);
        }
    }

    // we set this to vm._watcher inside the watcher's constructor
    // since the watcher's initial patch may call $forceUpdate (e.g. inside child
    // component's mounted hook), which relies on vm._watcher being already defined

    new Watcher(vm, updataComponent, {
        before() {
            if(vm._isMounted) {
                callHook(vm, 'beforeUpdate');
            }
        }
    }, true /* isRenderWatcher */);
    hydrating = false;
    // manually mounted instance, call mounted on self
    // mounted is called for render-created child components in its inserted hook
    if (vm.$vnode == null) {
        vm._isMounted = true
        callHook(vm, 'mounted')
    }
    return vm
}
```

mountComponent核心就是实例化一个渲染watcher，在回调函数中执行updateComponent方法，在方法中调用vm._render生成虚拟Node，vm._update更新DOM。  

Watcher在这里起两个作用，一是初始化是会执行回调函数，另一个是当vm实例中的监测数据发生变化时执行回调函数。

函数最后判断为根节点的时候设置 vm._isMounted 为 true， 表示这个实例已经挂载了，同时执行 mounted 钩子函数。 这里注意 vm.$vnode 表示 Vue 实例的父虚拟 Node，所以它为 Null 则表示当前是根 Vue 的实例。

### render 方法

```js
Vue.prototype._render = function (): VNode {
  const vm: Component = this
  const { render, _parentVnode } = vm.$options

  // reset _rendered flag on slots for duplicate slot check
  if (process.env.NODE_ENV !== 'production') {
    for (const key in vm.$slots) {
      // $flow-disable-line
      vm.$slots[key]._rendered = false
    }
  }

  if (_parentVnode) {
    vm.$scopedSlots = _parentVnode.data.scopedSlots || emptyObject
  }

  // set parent vnode. this allows render functions to have access
  // to the data on the placeholder node.
  vm.$vnode = _parentVnode
  // render self
  let vnode
  try {
    vnode = render.call(vm._renderProxy, vm.$createElement)
  } catch (e) {
    handleError(e, vm, `render`)
    // return error render result,
    // or previous vnode to prevent render error causing blank component
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      if (vm.$options.renderError) {
        try {
          vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e)
        } catch (e) {
          handleError(e, vm, `renderError`)
          vnode = vm._vnode
        }
      } else {
        vnode = vm._vnode
      }
    } else {
      vnode = vm._vnode
    }
  }
  // return empty vnode in case the render function errored out
  if (!(vnode instanceof VNode)) {
    if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
      warn(
        'Multiple root nodes returned from render function. Render function ' +
        'should return a single root node.',
        vm
      )
    }
    vnode = createEmptyVNode()
  }
  // set parent
  vnode.parent = _parentVnode
  return vnode
}
```

createElement() 生成 VNode。

```js
// wrapper function for providing a more flexible interface
// without getting yelled at by flow
export function createElement (
  context: Component,
  tag: any,
  data: any,
  children: any,
  normalizationType: any,
  alwaysNormalize: boolean
): VNode | Array<VNode> {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children
    children = data
    data = undefined
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```

createElement 方法实际上是对 _createElement 方法的封装，它允许传入的参数更加灵活，在处理这些参数后，调用真正创建 VNode 的函数_createElement：

```js
export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode()
  }
  // object syntax in v-bind
  if (isDef(data) && isDef(data.is)) {
    tag = data.is
  }
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode()
  }
  // warn against non-primitive key
  if (process.env.NODE_ENV !== 'production' &&
    isDef(data) && isDef(data.key) && !isPrimitive(data.key)
  ) {
    if (!__WEEX__ || !('@binding' in data.key)) {
      warn(
        'Avoid using non-primitive value as key, ' +
        'use string/number value instead.',
        context
      )
    }
  }
  // support single function children as default scoped slot
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if (isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children)
  }
  if (Array.isArray(vnode)) {
    return vnode
  } else if (isDef(vnode)) {
    if (isDef(ns)) applyNS(vnode, ns)
    if (isDef(data)) registerDeepBindings(data)
    return vnode
  } else {
    return createEmptyVNode()
  }
}
```

### update方法

Vue的_update是实例一个私有方法，它被调用的2个时机，一是初次渲染，一个是数据更新的时候。

```js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean){
    const vm: Component = this;
    const prevEl = vm.$el;
    const prevNode = vm._vnode;
    const prevActiveInstance = activeInstance;
    activeInstance = vm;
    vm._vnode = vnode;

    if(!prevNode) {
        vm.$el = vm.__patch__(vm, $el, vnode, hydrating, false /* removeOnly */)
    } else {
        // updates
        vm.$el = vm.__patch__(prevNode, vnode);
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    if (prevEl) {
        prevEl.__vue__ = null
    }
    if (vm.$el) {
        vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
        vm.$parent.$el = vm.$el
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
}
```

_update 的核心就是调用 vm.__patch__ 方法,这个方法实际上在不同的平台，比如 web 和 weex 上的定义是不一样的
```Vue.prototype.__patch__ = inBrowser ? patch : noop```

![挂载过程](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)
