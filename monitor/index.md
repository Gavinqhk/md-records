# 前端监控

前端监控主要为了做下面的事情。

1. 异常监控
    js错误监控，资源加载异常监控等等
2. 性能监控
    fp, fcp, tti等监控
3. 数据埋点
    操作系统，分辨率，浏览器，事件分类，访客标识，用户标识等信息，
4. 用户行为采集。
    pv,uv等数据上报，记录用户访问信息等。自定义数据上报，用于分析用户行为，如进入页面，离开页面，点击元素，滚动页面等操作。

需要实现监控需要做到以下的几步

1. 指标采集
    SDK形式介入web/h5/mini-program。通过前端集成的 SDK 收集请求、性能、异常等指标信息；在客户端简单的处理一次，然后上报到服务器。
2. 指标存储
    用于接收前端上报的采集信息，主要目的是数据落地（前端把数据发送到后端服务，后端服务将数据存储起来）
3. 统计与分析
    自动分析，通过数据的统计，让程序发现问题从而触发报警。人工分析，是通过可视化的数据面板，让使用者看到具体的日志数据，从而发现异常问题根源。
4. 可视化展示
    通过可视化的平台；在这些指标（API 监控、异常监控、资源监控、性能监控）中，追查用户行为来定位各项问题。

## 技术实现

### 数据采集

采集应满足对原有程序无侵入性，不会对原有程序产生影响。只需要在原程序注入一段SDK代码，SDK代码自定采集数据，并自动上报数据。SDK会自动监听几类数据，分别是：js错误监听，资源加载错误监听，页面性能监听，API调用监听。通过这几项监听，看完完成三项指标对采集。

1. 异常采集：js异常/资源加载异常。主要监听error/unhandledrejection事件，用于捕获js,css,图片等资源异常。error可监听同步错误、异步错误、资源加载错误，不能捕获Promise 的错误，unhandledrejection可监听为捕获的Promise错误、async/await。
2. 性能采集：调用浏览器原生的 performance.timing API 捕获页面的性能指标。
3. 接口采集：通过 Object.definePropety 代理全局的 XHR 用于捕获浏览器的 XHR/FETCH 的请求。

### 性能指标获取方式

我们借助于浏览器原生的 Navigation Timing API 能够获取到上述页面加载过程中的各项性能指标数据，用于性能分析，它的时间单位是纳秒级。
也借助于 PerformanceObserver API 等用于测量 FCP、LCP、FID、TTI、TBT、CLS 等关键性指标

| 指标 | 含义 | 计算公式|
| -- | -- | -- |
| ttfb | 首字节时间 | timing.responseStart - timing.requestStart |

![性能指标](./%E6%80%A7%E8%83%BD.png)

### 网络请求采集方案

重写XMLHttpRequest/fetch构造函数

```js
const _XMLHttpRequest = window.XMLHttpRequest;
if (_XMLHttpRequest) {
    // noinspection JSValidateTypes
    window.XMLHttpRequest = function XMLHttpRequest() {
        const xhr = new _XMLHttpRequest();

        const errorHandler = function() {
            // 上报请求失败
        };

        const timeoutHandler = function() {
            // 上报请求超时
        };

        const readyHandler = function() {
            if (xhr.readyState === xhr.DONE) {
                // 上报请求成功
            }
        };

        xhr.addEventListener('error', errorHandler, true);
        xhr.addEventListener('timeout', timeoutHandler, true);
        xhr.addEventListener('readystatechange', readyHandler, true);

        return xhr;
    };
}
```

```js
const _fetch = window.fetch;
if (_fetch) {
    window.fetch = function fetch(url, options = {}) {
        return _fetch.call(window, url, options)
            .then(function(res) {
                // 上报请求成功
                return res;
            })
            .catch(function(err) {
                // 上报请求失败
                throw err;
            });
    };
}
```

接口监控是最好是不要实时上报数据，这样对网络资源是一种消耗，并且会影响到源程序对执行。可将上报数据存储到一个数据结构中，当前页面不再发起请求当时候上报，或者当达到一定的数据量的时候再上报数据。可使用requestIdleCallback上报数据。
