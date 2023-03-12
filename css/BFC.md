# BFC是什么？

BFC（block Formatting context)块级格式化上下文，是Web页面的可视化css渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素和其它元素交互的区域。

BFC是独立的容器，在内部自元素不管怎么布局都不会影响到外部的元素。

## 触发BFC的条件。（包含其中之一就可）

1. body根元素
2. 浮动布局float除了none意外的值。
3. 绝对定位布局（absolute和fixed）
4. overflow: hidden， auto, scroll
5. display为flex，inline-block， table-cells

## BFC解决的问题

1. 同一BFC作用下的外边距（margin上下边距）取消重叠。
2. float引起的父级高度塌陷

    ```js
    <div style="border: 1px solid #000;">
        <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
    </div>
    ```

    ![flaot父级高度塌陷](https://pic4.zhimg.com/80/v2-371eb702274af831df909b2c55d6a14b_720w.png)

    ```js
    <div style="border: 1px solid #000;overflow: hidden">
        <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
    </div>
    ```

    ![1](https://pic4.zhimg.com/80/v2-cc8365db5c9cc5ca003ce9afe88592e7_720w.png)

3. 阻止元素被浮动元素覆盖

```js
<div style="height: 100px;width: 100px;float: left;background: lightblue">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;background: #eee">我是一个没有设置浮动, 
也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>
```

```js
<div style="height: 100px;width: 100px;float: left;background: lightblue">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;background: #eee">我是一个没有设置浮动, 也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>

这时候其实第二个元素有部分被浮动元素所覆盖，(但是文本信息不会被浮动元素所覆盖) 如果想避免元素被覆盖，可触第二个元素的 BFC 特性，在第二个元素中加入 overflow: hidden，就会变成：
```

```js
<div style="height: 100px;width: 100px;float: left;background: lightblue">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;background: #eee; overflow：hidden">我是一个没有设置浮动, 
也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>
```

![浮动覆盖](https://pic3.zhimg.com/80/v2-5ebd48f09fac875f0bd25823c76ba7fa_720w.png)
