# 对vue 和 spa 的理解

## 对于vue的理解

<https://mp.weixin.qq.com/s/dJ8WPIQ9xtI_SYfxGdUKeg>

vue是用于开发用户界面的一个JavaScript框架，致力于易用，灵活，高效的实现前端开发。

核心内容：

1. 双向数据绑定MVVM
2. 虚拟dom
3. 组件化
4. 指令系统
5. 与传统对比，不需要在通过直接操作dom来实现视图的变化，而是数据变化之后vue监听到数据变化通过diff等操作最终更新视图。
6. vue 与 react 的区别（没有好坏，只是场景不同）

形同点：

1. 都有组件化思想
2. 都有虚拟dom
3. 都是数据驱动（MVVM）
4. 都支持服务器渲染（vue框架nuxt.js, react框架next.js）
5. 都有自己的构建工具 vue-cli， creat-react-app
6. 都有支持native的能力，weex， react native

不同点：（都要具体搞清楚）

1. 数据流向不同，react是单向数据流向，vue是双向数据流向（这个需要仔细搞清楚）
2. 数据变化的实现原理不同，react的数据是不可变的，vue数据是可变的（这个也要仔细搞清楚）
3. 组件通信不同。react组件中通过回调函数来进行通信，vue通过事件和回调。（这个也要具体搞清楚）
4. diff的实现不同，react主要适用diff队列保存需要更新的dom，得到patch树，在统一批量更新，vue使用双指针，边对比边更新（这个也要具体搞清楚）

## 对spa的理解

spa(single-page application), 就是单页面应用。

SPA是一种网络应用程序或网站的模型，通过动态重写当前页面来与用户交互，这种方式避免了页面之间切换打断用户体验。

SPA所有链接都会重定向到项目的index.html路径，通过这个入口文件解析加载html css js来实现页面的渲染和交互。**（这一步会把所有页面的js,css都加载出来，这就是引起首屏加载慢出现白屏的情况，需要理解项目是怎么一次性把所有内容加载出来的并思考解决方法。）**

那么怎么确定这个url具体是呈现什么页面呢？那就需要通过router来实现页面的呈现。router的实现通过浏览器的hash模式和history模式来实现对浏览器url的监控处理。

hash模式下监听hashChange来出发对于的页面显示。

history模式下通过监听popState事件还确定history发生了变化。并通过pushState,replaceState来添加或者修改history的记录。
**（这里可以思考一下vue-router和react-router是怎么实现的，vue-router 通过```<router-view>```来实现页面的挂载，这也是个vue组件，思考一下是如何实现的。react-router 同理 是通过 ```<Route>``` 来实现组件显示，思考是如何实现router，如何实现组件挂载显示，如何操作router等等问题）**
