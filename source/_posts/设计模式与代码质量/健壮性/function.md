**function 还可能是一个类**
>如何保证function被当作类调用，而不是当成方法调用？
>关键点就在this上, 我们都知道this是方法执行时的上下文环境，
如果是一个类，那this就应当指向当前类的实例
```js
function vue(){
  if(this instanceof vue){ // 通过instanceof vue 判断this 是不是vue实例
    // 类实例
  } else {
    // 当作一个方法调用
    // return new vue(); // 这样即使当一个方法调用 ，拿到的还是一个vue的实例
    
    // 像一些框架之类的都会有这种措施  vue是这样
    throw new Error('vue is a constructor and should be called with the `new` keyword ')
  }
}

vue(); // 直接调用时，this ---> window
new vue(); // new 操作符调用时， this 指向vue实例 通过instanceof vue 可以判断

```