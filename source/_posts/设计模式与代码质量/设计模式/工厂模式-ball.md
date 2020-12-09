**球类工厂**
> 需求：我们要生产不同类型的球：basketball  footerball tennis(网球)...
```js
// 球类工厂
function ballFactory(type){

  switch(type){
    case 'football':
      return new football();
      break;
    case 'basketball':
      return new basketball();
      break;
    case 'tennis':
      return new tennis();
      break;
    default:
      break;
  }
  
  // 可以在局部 
  function football(){} 
  function basketball(){}
  function tennis(){}
}

// 可以在原型链上
ball.prototype.football = function(){}
ball.prototype.basketball = function(){}
ball.prototype.tennis = function(){}

```