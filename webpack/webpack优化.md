# webpack 优化

优化主要从两个方向思考

1. 打包速度
2. 打包体积

## 优化打包速度

优化打包速度从几方面思考

1. 解析优化
    - resolve使用alias来减少webpack解析路径，extensions 根据项目中的文件类型，定义 extensions，以覆盖 webpack 默认的 extensions，加快解析速度。
    - loader解析的优化 配置loader时设置include和exclude来设置解析路径和不解析路径来加快loader解析的速度。
    - modules 表示 webpack 解析模块时需要解析的目录，指定目录可缩小 webpack 解析范围，加快构建速度。
    - externals 对第三方包进行公共包CDN引用，降低包大小
    - mainFields

2. 缓存。将一些不变动的模块代码打包后缓存起来，之后的打包就不会重新打包，减少不少时间。webpack5已经内置了缓存模块cache
    - cache

3. 多进程打包。当某个解析打包任务繁重的时候会阻塞很长时间，开启多进程打包能有效减少时间。thread-loaders

4. 区分环境：区分dev/prod环境，对应不同环境进行区别操作
    - 开发环境
    - 生产环境

## 打包体积

1. 代码分离
    - 抽离重复代码：- optimization -> splitChunks -> cacheGroups -> common&vendor
    - 提取css到单独文件：mini-css-extract-plugin

2. 代码压缩
    - js压缩：terser-webpack-plugin
    - css压缩：css-minimizer-webpack-plugin

3. tree sharking
    - js
    - css：purgecss-webpack-plugin

4. CDN 静态文件上传cdn

5. 图片，字体图标处理等

## 加快加载速度

1. 按需加载：减少初次加载到体积
2. 浏览器缓存
3. cdn

## 总结

在加快构建时间方面，作用最大的是配置 cache，可大大加快二次构建速度。

在减小打包体积方面，作用最大的是压缩代码、分离重复代码、Tree Shaking，可最大幅度减小打包体积。

在加快加载速度方面，按需加载、浏览器缓存、CDN 效果都很显著。
