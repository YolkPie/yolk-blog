**装饰者模式**
> 目的：不重写方法的扩展方法
> 应用场景：当一个方法需要扩展，但又不方便或不能去修改方法时 是公共的方法 或他人的方法 或原生方法 或第三方模块的方法
> 简单理解 在不去修改原方法的情况下，扩展方法的功能

> 需求： 删除按钮--> 点击可以删除 ---> 但没有提示  ---> 好多删除按钮都是这样实现，产品需求是要给出删除提示
> 分析：在原来的删除功能基础上，扩展出提示功能
>> 1、全部改写  2、找到原来定义，修改原方法，增加提示
```js
function decorate(dom, fn){
  // 健壮性校验
  if(typeof dom.onClick === 'function'){
    // 装饰者 三步走
    // 1、重写原方法，或定义新方法;
    // 2、提取老方法，并调用
    // 3、加入新方法

    let _oldFn = dom.onClick; // 在方法被重写前提取
    dom.onClick = function(params){
      _oldFn.call(this);
      fn.call(this);
    }
  }
}
// 使用
decorate('btnDel', function(params){
  console.log('删除成功')
});
```