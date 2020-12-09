**Vue2 数组的双向绑定**
```js

// 数组：利用装饰者模式，给数组的 push pop 
let arr = ['push', 'pop', 'shift']
let arrProto = Array.prototype;
let arrCopy = Object.create(arrProto); // 为了不污染旧原型链，提前拷贝一份出来
arr.forEach(method => {
  arrCopy[method] = function (){
    let _ref = arrProto[method].apply(this, arguments);
    dep.notify();// 触发更新
  }
})
// 把data里的所有数组，都变成这里的这个arrCopy
// 主要是一些技巧
```