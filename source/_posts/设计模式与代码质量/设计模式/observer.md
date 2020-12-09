**观察者模式**
> 需求：抽奖转盘，特点是越转越快，
模块分析： 奖品初始化html --> 最终结果选定 --> 转动控制 --> 转动效果 
>>转动控制模块 调用 转动效果模块，
转动效果只负责转动效果; 转完之后，询问控制模块接下来怎么转;
转动效果模块接收消息体{moveTime: 10, speed: 200} 多长时间内转动几个奖品 --> setInterval 异步了，
转动控制与转动效果的沟通问题在于：转动控制不知道什么时候转动效果结束，因为转动速度是不恒定的

```js
// 观察者三要素：队列，注册，触发
function observer(params){
  // 1、队列
  this.message = {

  }
}
// 2、注册
observer.prototype.regist = function(type, fn){
  this.message[key] = fn;
}
// 3、触发
observer.prototype.fire = function(type){
  this.message[key]();
}

var observerObj = new observer();

var _domArr = [];
function htmlInt(params){
  for(let i = 1; i <= 5; i++){
    _domArr.push(document.body.append(`<div>${i}</div>`));
  }
}
function getResult(params){
  let _num = Math.random() * 10 + 40; // 40是基础动画圈数
  return Math.floor(_num, 0);
}
function moveControll(params){
  let result = getResult();
  let _circle = Math.floor(result/10, 0);// 基础动画圈
  let _runCircle = 0; // 已经转了多少圈
  let stopNum = result%10; // 最终停留的奖品数
  let _speed = 200; // 转速
  observerObj.regist('finish', () => {
    let _time = 0; // 应该转几个数
    _speed -=50; // 转一圈 速度加快50
    _runCircle++; // 已转的圈数
    if(_runCircle <= _circle){ // 未达到指定的圈数
      _time = 10; // 继续转10个数
    } else {
      _time = stopNum;
    }

    move({moveTime: _time, speed: _speed});
  });
}
// 动画效果：1，2，3，4，5...10 依次高亮显示
function move(config){
  let nowIn = 0;
  let rmNum = 9; // 移除效果的索引
  let timer = setInterval(() => {
    // 单独处理第10个跳第1个的情况
    if(nowIn == 0){  // 代码相似了，可以优化了
      _domArr[9].setAttribute('class', 'item');
      _domArr[nowIn].setAttribute('class', 'item item-on');
    } else{
      _domArr[nowIn-1].setAttribute('class', 'item');
      _domArr[nowIn].setAttribute('class', 'item item-on');
    }
    
    // 享元模式 
    // if(nowIn != 9){
    //   rmNum = nowIn--
    // }
    // _domArr[rmNum].setAttribute('class', 'item');
    // _domArr[nowIn].setAttribute('class', 'item item-on');

    nowIn++;
    if(nowIn == config.moveTime){
      clearInterval(timer);
      observerObj.fire('finish');
    }
  }, config.speed);
}
```