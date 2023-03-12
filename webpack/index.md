# webpack

基于node环境的前端打包工具，通过配置打包的入口文件，出口文件，loader，plugin来进行打包。

## 入口文件

entry可配置一个入口也可配置多个入口

## 出口文件

outPut配置出口文件的路径，文件名等

## loader

webpack只能识别JavaScript和json文件，但是在项目中会有很多类型文件，比如.vue,.ts,.tsx,.jsx等等。都需要用loader来进行编译成js/json文件。

## plugin

webpack的plugin系统是核心功能，plugin是处理loader处理不了的一些任务，为loader增强功能。

plugin是通过一个apply函数来进行挂载到webpack的生命周期中暴露出来的钩子函数，当webpack执行到对应的钩子函数则将挂载的插件执行。

> plugin在执行过程中会创建一个compiler编译器对象，全局都可访问它。还会有个compilation对象，具体内容继续学习中。
> 还有很多内容，比如module, resolve等等

问题：

1. 组件异步加载是怎么实现的import()原理是什么
2. 打包输出的chunk存在的hash（有几种hash，分别是什么之间的区别是什么）
3. 如何优化打包速度和体积值得思考
