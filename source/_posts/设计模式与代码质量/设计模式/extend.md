**jquery.extend 享元模式实现**
> extend实现的功能：
  \$.extend({a:1}) // 会把对象扩展到\$对象上\$.a = 1; 为jquery对象扩展方法属性使用
  \$.extend({a:1},{b:2}) // ==> {a:1,b:2}
```js
// 不用设计模式时，是这样实现的
$.extend = function(){
  // 享元模式，提取不同点：
  // 1、for in 的对象不同，
  // 2、接收的对象不同
  let source = arguments[0];
  let target = this;
  if(arguments.length === 2){
    source = arguments[1];
    target = arguments[0];
  }
  for(let item in source){
    target[item] = source[item]
  }

}
```
**总结应用场景： 两个if else 分支中，两段代码块非常相似时，就可以用享元模式了**
