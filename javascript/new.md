# new 实现的原理

new操作发生了一下事情

1. 新建了一个空对象
2. 将对象的__proto__指向构造函数的原型prototype
3. 返回了该对象。若构造函数有返回对象则返回构造函数返回的对象，否则返回新建的对象。

```js
function newFun(consfun, args1, args2 ....) {
    let obj = Object.create(null);
    let constructor = arguments[0];
    let args = Array.prototype.slice.call(arguments, 1);

    obj.__proto__ = constructor.prototype;

    let resp = constructor.apply(obj, args);

    return typeof resp === "object" ? resp : obj;
}
```

