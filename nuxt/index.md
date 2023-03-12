# nuxt.js

nuxt.js 是基于vue.js开发的服务端渲染的前端框架

## 为什么要用nuxt.js

1. 更好的SEO
2. 更快的内容呈现（更快的首屏加载）

## nuxt 性能差的原因

众所周知nuxt的性能差体现在高并发的情况下，错误率响应时长都不友好，那具体原因是什么呢。

- 猜想nuxt是需要在node服务器下运行，将程序执行最终生成html字符串，在这个过程中会执行nuxt服务端生命周期里的所有任务。
- 为避免实例污染，一个请求一个实例，当高并发情况下同时执行多个实例，这就造成了执行慢点问题。
- 那么为什么会慢呢？node不是号称非阻塞式I/O吗。没错，node是号称非阻塞I/O，但是在服务端执行nuxt生命周期（创建组件实例，执行渲染函数生成虚拟dom，执行patch生成dom等方法，再加上虚拟dom调用链层级太深）是非常消耗cpu。

## 项目问题分析

当知道了nuxt性能问题，接下来我们就应该从各方面去思考如何优化。

- 首先分析代码，看有哪些影响性能等问题
  - 项目未缓存
  - pc 移动端 一套代码 存在代码冗余情况，图片都是使用pc的大图
  - 网站大量图片（渐变色小图标）
  - api请求位置不太合理，没有完整明确区分哪些是服务端/客户端请求的借口
  - seo: 内容过多，没有很好区分哪些是seo所需要的内容，所有内容都全部服务端直出。
  - ......

- 再分析构建阶段
  - 通过webpack-bundle-analyzer分析项目打包结构，看分包是否合理，第三方包是否处理好等问题
  - loader等配置排除（exclude）文件夹路径，只搜索特定路径（include）
  - 体积大的文件需要分包
  - 构建阶段也可以使用缓存减少打包时间
  - ......

## 优化方案

- 缓存。缓存是最重要的方案，针对 Nuxt 项目可以做三级缓存，页面缓存、组件缓存以及 API 缓存。页面缓存是最重量级的缓存方案，能不能做页面缓存可以从以下两个点判断
  - 页面缓存。页面缓存需要注意的两点是，1、同一url在用户登录/非登录状态下是否有差异。2、同一url对于不同登录用户是否有差异。需要页面缓存的应该是不同情况下返回的html结构是一样的。
    - 这里分几种情况考虑一下，页面所有内容不同情况下返回都是一致的这种适合做页面缓存
    - 页面大多内容在不同情况下返回一致，只有用户信息部分是不一致的这种如何做缓存策略？（做页面级页面缓存，但是用户信息部分在客户端渲染，不在服务端渲染）

    ```js
    // 服务端中间件middleware/page-cache.js
    const LRU = require('lru-cache')
    let cachePage = new LRU({
      max: 100, // 缓存队列长度
      maxAge: 1000 * 60 // 缓存1分钟
    })
    export default function(req, res, next){
      let url = req._parsedOriginalUrl
      let pathname = url.pathname
      // 通过路由判断，只有首页才进行缓存
      if (['/home'].indexOf(pathname) > -1) {
        const existsHtml = cachePage.get('homeData')
        if (existsHtml) {
          return res.end(existsHtml.html, 'utf-8')
        } else {
          res.original_end = res.end
          // 重写res.end
          res.end = function (data) {
            if (res.statusCode === 200) {
              // 设置缓存
              cachePage.set('homeData', { html: data})
            }
            // 最终返回结果
            res.original_end(data, 'utf-8')
          }
        }
      }
      next()
    }
    // nuxt.config.js配置项修改，引入服务端中间件
    //针对home路由做缓存
    serverMiddleware: [
      { path: '/home', handler: '~/middleware/page-cache.js' },
    ]
    ```

  - 组件缓存
    - 将渲染后的组件DOM结构存入缓存，定时刷新，有效期通过缓存获取组件DOM结构，减小生成DOM结构所需时间；适用于渲染后结构不变或只有几种变换、并不影响上下文的组件。适用于单纯的v-for的数据展示列表 并且是根据某个属性id变更后才需要更新的列表

    ```js
    // nuxt.config.js配置项修改
    const LRU = require('lru-cache')
    module.exports = {
      render: {
        bundleRenderer: {
          cache: LRU({
            max: 1000, // 缓存队列长度
            maxAge: 1000 * 60 // 缓存1分钟
          })
        }
      }
    }

    // 需要做缓存的 vue 组件， 需增加 name 以及 serverCacheKey 字段，以确定缓存的唯一键值。
    export default {
      name: 'zzZyHome',
      props: ['type'],
      serverCacheKey: props => props.type
    }

    //如果组件依赖于很多的全局状态，或者状态取值非常多，缓存会因频繁被设置而导致溢出，这样的组件做缓存就没有多大意义了；
    //另外组件缓存，只是缓存了dom结构，如created等钩子中的代码逻辑并不会被缓存，如果其中逻辑会影响上下边变更，是不会再执行的，此种组件也不适合缓存
    ```

  - API缓存
    - 将服务端获取的数据，全部缓存到node进程内存中，定时刷新，有效期内请求都通过缓存获取API接口数据，减小数据获取时间；此种缓存适用于缓存的部分API数据，基本保持不变，变更不频繁，与用户个人数据无关。纯数据列表展示,不对列表某些值进行手动修改,比如新闻列表等数据流

    ```js
    import LRU from 'lru-cache'
    const CACHED = new LRU({
        max: 100, // 缓存队列长度
        maxAge: 1000 * 60 // 缓存时间
    })
    export default {
      async asyncData({ app, query }) {
        try {
          let banner, footer
          if (CACHED.has('baseData')) {
            // 存在缓存，使用缓存数据
            let data = CACHED.get('baseData')
            data = JSON.parse(data)
            banner = data.banner
            footer = data.footer
          } else {
            // 获取页面顶部轮播图信息
            const getBanner = () => {
              return app.$axios.$get('zz/zy/banner')
            }
            // 获取底部配置信息
            const getFooter = () => {
              return app.$axios.$get('zz/zy/footer', {
                params: {
                  smark: query.smark
                }
              })
            }
            [banner, footer] = await Promise.all([getBanner(), getFooter()])
            // 将数据写入缓存
            CACHED.set('baseData', JSON.stringify({ banner: banner, footer: footer}))
          }
          return {mods: mods, footer: footer}
        } catch (e) {
          console.log('interface timeout or format error => ', e)
          return {}
        }
      }
    }
    ```

- 图片优化
  - 压缩，用webp格式图片替换
  - 小图片不需要做成雪碧图，在构建时候小图标会生成base64。
  - 对于不同屏幕使用不同图片

- 非服务端渲染部分内容的API接口请求放在mounted生命周期中确保只在客户端请求

- 同一套代码存在的问题就是代码冗余，html代码和css代码会出现大量冗余代码，就算是通过css媒体查询写的简洁，也是会有多余代码。解决方法:可以是将不同端代码写在不同css文件，在服务端判断设备ua区分移动端和pc端，当页面加载时判断不同设备尺寸来行设备兼容。

## 服务端输出模式

目前这个我还没完全搞懂，三种模式？

一是服务端将html数据完全生成之后发送给客户端，客户端完全接收之后再进行渲染。
二是服务端将html数据完全生产之后发送客户端，客户端边接收数据边渲染。
三是服务端将html数据边生成边传输给客户端？客户端边接受边渲染。

个人理解是模式二。
服务端是通过vue/server-renderer插件将.vue生成html数据，这时候就已经是完整的html数据了，然后res.send(html)返回给客户端，这里也是完整数据。
客户端边接收数据边边渲染，这个应该是浏览器的特性而不是服务端的特性，浏览器在接收数据的时候判断是html数据，就会在网络进程和渲染进程创建个通信管道，陆续将网络进程接收的数据通过管道通信提交给渲染进程，渲染进程在接受数据的同时就会解析html数据结构进行后续的加载渲染等等操作。

💣：有可能是我个人理解不到位。服务端在生成html数据的时候，也有可能是边生成html数据边send给客户端。也是严格按照顺序发送，这样就可以根据到达客户端的顺序边渲染。
