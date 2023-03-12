# loader

loader是一个node的模块，用于webpack打包的时候进行对不同文件类型解析成js或者json文件。

## loader用法

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      { test: /\.js$/, use: 'babel-loader' },
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          { loader: 'css-loader' },
          { loader: 'postcss-loader' },
        ]
      }
    ]
  }
};
```

loader的几种用法及优先级 pre(前置)->normal(config)->inline(内联)->post(后置)

inline 就是使用loader解析单一文件

```js
import style from 'style-loader!css-loader?modules!./styles.css';
```

normal就是普通webpack配置的loader

pre/post 也是在webpack配置的，webpack定义了一个属性 enforce，可取值有 pre（为pre loader）、post（为post loader），如果没有值则为（normal loader）。

所有的loader的执行顺序都有两个阶段：pitching和normal阶段，类似于js中的事件冒泡、捕获阶段（有人嫌官方的词描述的比较晦涩，修改为loader的标记阶段（mark stage）和执行阶段（execution/run stage））。

对于第二种方式的解释：webpack 官方解释

Pitching阶段： post，inline，normal，pre
Normal阶段：pre，normal，inline，post

异步loader

pitch

写一个loader
