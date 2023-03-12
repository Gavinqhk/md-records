# call 方法实现

call方法改变调用函数的this指向，是的没有该函数的对象能够执行该函数。

从现象看：

1. call方法接受多个参数，第一个参数为this指向（对象）。后续参数单独传入。
2. 结果有两种情况，有返回数据，无返回数据。
3. 当没有指定第一个参数时，挂载到widow上。

```js
//形参指定的this（需要调用函数的对象）
Function.prototype.mycall = function(context) {
    //参数有可能为null等情况，做兼容处理。
    let ctx = context || window;
    // 获取函数并添加到ctx中
    ctx.fn = this;
    // 获取参数
    let args = [];
    for(let i = 1; i < arguments.length; i++) {
        args.push(arguments[i]);
    }
    // 执行参数,有返回值则保存起来
    const resp = eval('ctx.fn(' + args + ')');
    delete ctx.fn;
    return resp;
}
```

## apply 的实现

apply 与 call 的差别就是传参的区别。
apply 第二个参数为数组。

```js
//形参指定的this（需要调用函数的对象）
Function.prototype.myapply = function(context) {
    //参数有可能为null等情况，做兼容处理。
    let ctx = context || window;
    // 获取函数并添加到ctx中
    ctx.fn = this;
    // 获取参数
    let args = arguments[1];
    // 执行参数,有返回值则保存起来
    const resp = eval('ctx.fn(' + args + ')');
    delete ctx.fn;
    return resp;
}
```

```js
function call() {
    // 指定this值为第一个参数，若第一参数不存在则指定window
    const context = arguments[0] || window
    // this为调用call的方法。将其存储在context中以方便执行。避免fn重复，使用symbol为唯一值。
    const fn = new Symbol('fn')
    context[fn] = this
    // for 循环获取参数值
    const args = []
    // for(let i = 0; i < arguments.length - 1; i++) {
    //     args[i] = arguments[i + 1]
    // }
    for(let i = 1; i < arguments.length; i++) {
        args.push('arguments['+i+']')
    }
    if(args.length) {
        const result = eval('context[fn]('+args+')')
    } else {
        const result = context[fn]()
    }
    delete context[fn]
    return result
}
function apply() {
    const context = arguments[0] || window
    const arr = arguments[1]
    const fn = new Symbol('fn')
    context[fn] = this
    let result = null
    if(!arr) {
        result = context[fn]()
    } else {
        const args = []
        for(let i = 0; i < arr.length; i++) {
            args.push('arr['+i+']')
        }
        result = eval('context[fn]('+args+')')
    }
    delete context[fn]
    return result
}
```
