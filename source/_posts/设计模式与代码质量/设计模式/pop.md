**弹框提示插件**
> 分析： 因为一个页面的消息提示可能会很多，添加成功，添加失败，网络出错
> 会需要频繁创建对象，所以考虑用工厂模式实现
```js
// 工厂模式实现 调用如下
pop('message')
// OR
pop.confirm('message')

// 建造者模式实现，调用如下
pop('message')
// OR
let popObj = new pop('message')
popObj.show();

// 哪个用起来更方便， 虽然看起来只少了一个new
```