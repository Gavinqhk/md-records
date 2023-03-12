# chrome 插件

chrome插件，为用户增强体验多样性。可以在生产环境中提高网站的实用性，也可以在开发过程中优化简化各种流程。
我考虑开发Chrome插件的原因是，微前端本地调试太过于繁琐。当本地调试子应用时，需要启动主应用才可以看到基座+子应用的页面。
通过插件页面配置子应用入口，写入开发环境localStorage，再通过开发环境主应用读取localStroage设置入口。减少本地调试的时候启用基座项目

参考[chrome 插件开发](https://zhuanlan.zhihu.com/p/438896257)

## 插件架构

web page <-> content-script <-> background <-> devtool-page

## web page

    这部分是用户界面，需要做到的事获取localStroage。也可通过window.postMessage向content-script发送消息

## content-script

    这部分代码是插入到用户界面程序的js。可接受webpage传来的消息，也需要发送给background数据。属于用户界面webpage和background的中间页。
    通过window.addEventListener('message', callback)来接受webpage的数据，chrome.runtime.sendMessage发送数据给background，
    chrome.runtime.onMessage.addListener接收background发送来的数据

## background

    这部分是在后台运行的页面，会生产一个背景页，可在扩展页面打开背景页做调试。通过chrome.runtime.sendMessage、chrome.runtime.onMessage.addListener来做数据的发送接收。也可以通过chrome.runtime.connect做一个长链接（目前没有这样做，后续使用到再详细了解）

## devtool-page

    这部分就是插件显示的页面，需要一个html文件来展示。也是可以通过chrome.runtime.sendMessage、chrome.runtime.onMessage.addListener来做数据的发送接收。也可以直接通过chrome.devtools.inspectedWindow.eval(str)写入到webpage让它执行这段str字符串。
    注意这里的stript需要在一个单独文件中，不然会报错，也可以通过配置修改。在写入webpage时候，需要找到对应的tabId才可以写入。chrome.runtime.tabs、chrome.devtools.inspectedWindow.tabId、chrmoe.tabs.query({active: true, ....})等。
