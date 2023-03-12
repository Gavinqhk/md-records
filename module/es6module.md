# es module

[es6 module](https://es6.ruanyifeng.com/#docs/module)
JavaScript一直没有模块（module）体系，无法将大型的项目拆分成互相依赖的小文件，再用简单的方式拼接起来。其他语言都有这项功能，比如 Ruby 的require、Python 的import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，这对开发大型的、复杂的项目形成了巨大障碍。

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

es6 module 设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系。

默认是严格模式

## export

## export default

## import

## 模块的加载

## export 与 import 的复合写法

## 模块的继承

## 跨模块的常量
