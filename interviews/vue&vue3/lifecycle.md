# 生命周期

在vue中实例从创建到销毁的过程就是生命周期，即指从创建，初始化数据，编译模版，挂载dom->渲染，更新->渲染，卸载等一系列过程
vue2
beforeCreate
created
beforeMount
mounted
beforeUpdata
updated
beforeDestroy
destored
activated // keep-alive 缓存组件激活时调用
deactivated // keep-alive 缓存组件停用时调用
errorCaptured // 捕获一个来自子孙组件等错误时被调用

vue3

| 选项式 API | Hook inside setup |
| --- | --- |
| beforeCreate | Not needed* |
| created | Not needed* |
| beforeMount | onBeforeMount |
| mounted | onMounted |
| beforeUpdate | onBeforeUpdate |
| updated | onUpdated |
| beforeUnmount | onBeforeUnmount |
| unmounted | onUnmounted |
| errorCaptured | onErrorCaptured |
| renderTracked | onRenderTracked |
| renderTriggered | onRenderTriggered |
| activated | onActivated |
| deactivated | onDeactivated |

errorCaptured, renderTracked, renderTriggered 了解这三个生命周期等作用。

errorCaptured：当子孙组件发生错误时，触发这个钩子函数，此钩子可以返回 false 以阻止该错误继续向上传播。

renderTracked：跟踪虚拟 DOM 重新渲染时调用。钩子接收 debugger event 作为参数。此事件告诉你哪个操作跟踪了组件以及该操作的目标对象和键。

renderTriggered：当虚拟 DOM 重新渲染被触发时调用。和 renderTracked 类似，接收 debugger event 作为参数。此事件告诉你是什么操作触发了重新渲染，以及该操作的目标对象和键。
