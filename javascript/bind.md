# bind 实现原理

从现象入手：

1. 参数传递（thisArg[,arg[,arg,...]] ）
2. 返回一个函数
3. 返回函数可以new，

> new返回的函数，需要确保new生产的对象隐式原型对象__proto__指向绑定函数的原型prototype。所以需要将返回的函数的原型prototype指向绑定函数的prototype。

```js
Function.prototype.mybind = function(context) {
    if(typeof this !== "function") {
        throw new Error("must be a function!!!!!");
    }
    let thisArg = context || window;
    let self = this;
    let outArgs = Array.prototype.slice.call(arguments, 1);
    let F = function() {};
    let fbound = function() {
        let args = outArg.concat(arguments);
        //若函数为匿名函数，则不可new，会报错
        self.apply(this instanceof self ? this : thisArg, args);
    }
    F.portotype = self.prototype;
    fbound.prototype = new F();
    
    return fbound;
}
```

```js
function bind() {
    const context = arguments[0] || window
    const self = this
    const args = []
    for(let i = 1; i < arguments.length; i++) {
        args.push(arguments[i])
    }
    const bound = function () {
        for(let i = 0; i < arguments.length; i++) {
            args.push(arguments[i])
        }
        return self.apply(this instanceof bound ? this : context, args)
    }
    const fnop = function() {}
    fnpo.portotype = self.prototype
    bound.prototype = new fnop()

    return bound
}
```
