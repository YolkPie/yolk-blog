**Jquery如何实现工厂模式的**
```js
// 静态函数 jquery的实现
function jquery(params){
  return new jquery.fn.init(params);
}
jquery.fn = {};
jquery.fn.init = function (params) {};
window.$ = jquery;

$('.className');// 实际是通过new jquery静态属性fn上的init方法
```
> jquery是通过挂载一个静态属性实现的