**表单验证工具**
> 需求：写一个表单验证工具，给我要验证的input 值 value 变化时，应用对应的规则，自动验证
> 分析：
  1、首先是包含的模块：dom初始化模块 --> 事件绑定模块 --> 验证模块 --> 消息提示模块
  2、一个页面上可能有多个需要验证的dom，所以适合工厂模式
```js
// 1、防JQuery
function t(dom, msgDom){
  return new t.init(dom, msgDom);
}
t.init = function(dom, msgDom){
  this.dom = dom;
  this.msgDom = msgDom;
  this.validateArr = [];
}
// 2、思考以后的扩展性，想想模块是否需要细化， 模块越细，扩展越方便
// 验证模块变化最大，细化
// 开启验证模块，验证队列模块
t.init.prototype.initBind = function(params){
  let self = this;
  this.dom.onblur = function(){
    self.run(this.value)
  }
}

t.init.prototype.add = function(fn){
  if(typeof fn === 'function'){
    this.validateArr.push(fn);
  } else if(typeof fn === 'string'){
    // 预置的验证规则 下面这样写，可读性差，代码不清楚
    // 引入设计模式解决
    // if(fn === 'isPhone'){
    //   this.validateArr.push(()=>{/*手机号验证*/})
    // } else if(fn === 'isNumber'){
    //   this.validateArr.push(()=>{/**是否是数字*/}) 
    // }

    // 策略模式解决 
    // 简单if-else可以很好解决
    let strage = {
      isPhone:function(params){},
      isNumber: function(params){},
      // 更多验证规则
    }
    this.validateArr.push(strage[fn]);

    // 这里涉及的逻辑比较简单，如果逻辑更复杂一点，可能这一个简单的策略就无法解决
    // 比如：由单一的条件变化，变成几个组合条件变化

    // 状态模式
    // 核心：根据对象不同的状态，让对象展示不同的行为， 相当于加了状态管理的策略模式
  }
}

t.init.prototype.run = function(value){
  while(this.validateArr.length > 0){
    // _result 是约定好的验证结果的数据结构
    // {success: true|false, msg:''}
    let _result = this.validateArr.shift().run(value);
    if(!_result.success){
      this.sendMsg(_result.msg);
      break;
    }
  }
}

t.init.prototype.sendMsg = function(msg){
  this.msgDom.innerHtml = msg;
}

// 后期的使用   职责链模式
t('input', 'errorMsg')
  .add('isPhone')
  .add(() => {/**自定义验证1 */})
  .add(() => {/**自定义验证2 */})
```