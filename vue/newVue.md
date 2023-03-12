# new Vue 做了什么事情

new 关键字在JavaScript中代表实例化一个对象，而Vue实际上是一个类，类在JavaScript中是用Function来实现的。

```js
function Vue(options) {
    if(process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
        warn('Vue is a constructor and should be called with `new` keyword');
    }
    this._init(options);
}
```

Vue 只是通过new关键字初始化，并调用_init()方法。

```js

    Vue.prototype._init(options?: object) {
        const vm: Component = this;
        vm._uid = uid++;
        // 省略部分操作
        .....
        // 省略部分操作end
        vm._isVue = true;

        //merge optipns
        if(options && options._isComponent) {
            //疑问：_isComponent属性是用来判断是否是组件？
            initInternalComponent(vm, options);
        } else {
            vm.$options = mergeOptions(
                resolveConstructorOptions(vm.constructor),
                options || {},
                vm
            );
        }
        if(process.env.NODE_ENV !== 'production') {
            initProxy(vm);
        } else {
            vm._renderProxy = vm;
        }
        //expose real self
        vm._self = vm;
        initLifeCycle(vm);
        initEvents(vm);
        initRender(vm);
        callHook(vm, 'beforeCreate');//callHook 调用执行生命周期函数
        initInjections(vm);//resolve injections before data/props
        initState(vm);
        initProvide(vm); // resolve provide after data/props
        callHook(vm, 'created');

        //疑问：这段if是什么意思？
        if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
            vm._name = formatComponentName(vm, false)
            mark(endTag)
            measure(`vue ${vm._name} init`, startTag, endTag)
        }

        //挂载
        if(vm.$options.el) {
            vm.$mount(vm.$options.el)
        }
    }

```

Vue初始化主要干了几件事，合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化props、data、computed、watcher等等。

遗留问题：弄明白上述疑问，以及代码中几个init函数的原理。