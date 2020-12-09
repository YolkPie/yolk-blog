**移动Div**
> 需求2：一个div 实现 上、下、左、右、左上、左下、....这样移动
  moveDiv('left') // 左移
  moveDiv('left','top') // 左上移

  ```js
  // 用状态模式实现
  // 这时候moveDiv就成了一个类了，不是一个方法了
  function moveDiv(params){
    this.stateArr = [];//因为存在复合运动的行为，所以需要一个数组去存储数据
  }
  moveDiv.prototype.run = function(params){
    // arguments 类数组  将类数组转成真正数组
    // 这里有几种方法把类数组转成数组？
    this.stateArr = Array.prototype.slice.call(arguments);
    // 策略模式
    let strage = {
      left: moveLeft,
      right:moveRight,
      top:moveTop
    };
    this.stateArr.forEach(state => {
      strage[state]();
    });
  }
  let moveObj = new moveDiv();
  moveObj.run('left', 'top');
  ```

  使用设计模式之后，这个模块的代码更简洁更易读了