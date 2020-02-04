---
title: es6新特性
date: 2020-02-04 14:25:07
author: 刘真延
tags: es6
categories: JavaScript
keywords: es6,新特性
description: ES6，全称 ECMAScript 6.0，是 JavaScript 的下一个版本标准，2015.06 发版。
cover: https://img11.360buyimg.com/imagetools/jfs/t1/94619/32/11516/7174/5e39109dE7d9c2139/0a3e964b274719b1.jpg
top_img: https://img13.360buyimg.com/imagetools/jfs/t1/88947/4/11658/18673/5e3910c7E8d2fd11b/bad9d4543fdc1188.jpg
---
## 一. es6对象的扩展

### 1. 属性的简洁表示法

ES6 允许直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。
```javascript
    let birth ='foo';

    const Person ={

      name:'张三';

     //等同于birth: birth
     birth,   

     // 等同于hello: function ()...
     hello(){ console.log(&#39;我的名字是&#39;,this.name);}  

    };
```
### 2. 属性名表达式

JavaScript 定义对象的属性，有两种方法。
```javascript
    // 方法一

    obj.foo =true;

    // 方法二

    obj['a'+'bc']=123;

```

ES6 允许字面量定义对象时，用表达式作为对象的属性名。
```javascript
    let propKey ='foo';

    let obj ={

     [propKey]:true,

     ['a'+'bc']:123

    };
```
表达式还可以用于定义方法名。
```javascript
    let obj ={

      ['h'+'ello'](){

        return 'hi' ;

      }

    };

    obj.hello()
```
### 3.属性的遍历

遍历对象属性的方法:

**（1）for...in**   循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。

**（2）Object.keys(obj)**   返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。

**（3）Object.getOwnPropertyNames(obj)**   返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。

**（4）Object.getOwnPropertySymbols(obj)**   返回一个数组，包含对象自身的所有 Symbol 属性的键名。

**（5）Reflect.ownKeys(obj)**   返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。

以上的 5 种方法遍历对象的键名，都遵守同样的属性遍历的次序规则。

- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列。
```javascript
       Reflect.ownKeys({[Symbol()]:0, b:0,10:0,2:0, a:0})

       // [2, 10, b, a, Symbol()]
```
### 4.super 关键字

this关键字总是指向函数所在的当前对象，ES6 又新增了另一个类似的关键字super，指向当前对象的原型对象。
```javascript
    const proto ={

        foo:'hello'

    };

    const obj ={

        foo:'world',

        find(){

            return super.foo;

        }

    };

    Object.setPrototypeOf(obj, proto);

    obj.find()
```
JavaScript 引擎内部，super.foo等同于Object.getPrototypeOf(this).foo（属性）或Object.getPrototypeOf(this).foo.call(this)（方法）。
```javascript
    const proto ={

        x:'hello',

        foo(){

            console.log(this.x);

        },

    };

    const obj ={

        x:'world',

        foo(){

            super.foo();

        }

    }

    Object.setPrototypeOf(obj, proto);

    obj.foo()
```
### 5.解构赋值

对象的解构赋值用于将目标对象自身的所有可遍历的（enumerable）、但尚未被读取的属性，分配到指定的对象上面。所有的键和它们的值，都会拷贝到新对象上面。
```javascript
    let{ x, y,...aa }={ x:1, y:2, a:3, b:4};

    x // 1

    y // 2

    z // { a: 3, b: 4 }
```
解构赋值的拷贝是浅拷贝，即如果一个键的值是复合类型的值（数组、对象、函数）、那么解构赋值拷贝的是这个值的引用。
```javascript
    let obj ={ a:{ b:1}};

    let{...x }= obj;

    obj.a.b =2;

    x.a.b // 2
```
另外，扩展运算符的解构赋值，不能复制继承自原型对象的属性。
```javascript
    let o1 ={ a:1};

    let o2 ={ b:2};

    o2.__proto__ = o1;

    let{...o3 }= o2;

    o3 // { b: 2 }

    o3.a// undefined
```
下面是另一个例子。
```javascript
    const o = Object.create({ x:1, y:2});

    o.z =3;

    let{ x,...newObj }= o;

    let{ y, z }= newObj;

    x // 1

    y // undefined

    z // 3
```
### 6.扩展运算符

对象的扩展运算符（...）用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。
```javascript
    let z ={ a:3, b:4};

    let n ={...z };

    n // { a: 3, b: 4 }
```
数组是特殊的对象，所以对象的扩展运算符也可以用于数组。
```javascript
    let foo ={...[a,b,c]};

    foo

    // {0: a, 1: b, 2: c}
```
如果扩展运算符后面不是对象，则会自动将其转为对象。
```javascript
    // 等同于 {...Object(1)}

    {...1} // {}
```
上面代码中，扩展运算符后面是整数1，会自动转为数值的包装对象Number{1}。由于该对象没有自身属性，所以返回一个空对象。

如果扩展运算符后面是字符串，它会自动转成一个类似数组的对象，因此返回的不是空对象。
```javascript
    {...'hello'}

    // {0: 'h', 1: 'e', 2: 'l', 3: 'l', 4: 'o'}
```
扩展运算符的参数对象之中，如果有取值函数get，这个函数是会执行的。
```javascript
// 并不会抛出错误，因为 x 属性只是被定义，但没执行
 
    let aWithXGetter ={

    ...a,

    getx(){

        throw new Error('not throw yet');

    }

    };

// 会抛出错误，因为 x 属性被执行了

    let runtimeError ={

    ...a,

    ...{

        getaa(){

            throw new Error('throw now');

        }

    }

    };
```
## 二．Es6对象的新增方法

### 1.Object.is()

ES5 比较两个值是否相等：相等运算符（==）和严格相等运算符（===）。

Object.is它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。
```javascript
    Object.is('foo','foo')

    // true

    Object.is({},{})

    // false
```
不同之处只有两个：一是+0不等于-0，二是NaN等于自身。
```javascript
    +0===-0 //true

    NaN===NaN // false

    Object.is(+0,-0) // false

    Object.is(NaN,NaN) // true
```
### 2.Object.assign()

用于对象的合并，将源对象（source）的所有可枚举属性，复制到目标对象（target）。
```javascript
    const target ={ a:1};

    const source1 ={ b:2};

    const source2 ={ c:3};

    Object.assign(target, source1, source2);

    target // {a:1, b:2, c:3}
```
如果目标对象与源对象有同名属性，或多个源对象有同名属性，则后面的属性会覆盖前面的属性。
```javascript
    const target ={ a:1, b:1};

    const source1 ={ b:2, c:2};

    const source2 ={ c:3};

    Object.assign(target, source1, source2);

    target // {a:1, b:2, c:3}
```
如果该参数不是对象，则会先转成对象，然后返回。
```javascript
    typeof Object.assign(2) // [object]
```
由于undefined和null无法转成对象，所以如果它们作为参数，就会报错。
```javascript
    Object.assign(undefined) // 报错

    Object.assign(null) // 报错
```
Object.assign只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false）
```javascript
    Object.assign({b:c},

    Object.defineProperty({},'invisible',{

        enumerable:false,

        value:'hello';

    })

    )

    // { b: c }
```
上面代码中，Object.assign要拷贝的对象只有一个不可枚举属性invisible，这个属性并没有被拷贝进去。

### 注意点

**（1）浅拷贝**

Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用。

**（2）同名属性的替换**

对于这种嵌套的对象，一旦遇到同名属性，Object.assign的处理方法是替换，而不是添加。
```javascript
    const target ={ a:{ b:c, d:e;}}

    const source ={ a:{ b:'hello';}}

    Object.assign(target, source)

    // { a: { b: 'hello' } }
```

**（3）数组的处理**

Object.assign可以用来处理数组，但是会把数组视为对象。
```javascript
    Object.assign([1,2,3],[4,5])
```
**（4）取值函数的处理**

Object.assign只能进行值的复制，如果要复制的值是一个取值函数，那么将求值后再复制。
```javascript
    const source ={

        getfoo(){return1}

    };

    const target ={};

    Object.assign(target, source)

    // { foo: 1 }
```
### 常见用途

**（1）为对象添加属性**
```javascript
    class Point{

        constructor(x, y){

            Object.assign(this,{x, y});

        }

    }
```
上面方法通过Object.assign方法，将x属性和y属性添加到Point类的对象实例。

**（2）为对象添加方法**
```javascript
    Object.assign(SomeClass.prototype,{

        someMethod(arg1, arg2){

            ···

        }

    });

    // 等同于下面的写法

    SomeClass.prototype.someMethod =function(arg1, arg2){

    ···

    };
```
**（3）克隆对象**
```javascript
    functionclone(origin){

        return Object.assign({}, origin);

    }
```
不过，采用这种方法克隆，只能克隆原始对象自身的值，不能克隆它继承的值。如果想要保持继承链，可以采用下面的代码。
```javascript
    functionclone(origin){

        let originProto = Object.getPrototypeOf(origin);

        return Object.assign(Object.create(originProto), origin);

    }
```
### 3.Object.getOwnPropertyDescriptors()

ES5 的Object.getOwnPropertyDescriptor()方法会返回某个对象属性的描述对象（descriptor）。

ES2017 引入了Object.getOwnPropertyDescriptors()方法，返回指定对象所有自身属性（非继承属性）的描述对象。
```javascript
    const obj ={

        foo:123,

        getbar(){return&#39;abc&#39;}

    };

    Object.getOwnPropertyDescriptors(obj)

    // { foo:

    //    { value: 123,

    //      writable: true,

    //      enumerable: true,

    //      configurable: true },

    //   bar:

    //    { get: [Function: get bar],

    //      set: undefined,

    //      enumerable: true,

    //      configurable: true } }
```
该方法的引入目的，主要是为了解决Object.assign()无法正确拷贝get属性和set属性的问题。
```javascript
    const source ={

        setfoo(value){

            console.log(value);

        }

    };

    const target1 ={};

    Object.assign(target1, source);

    Object.getOwnPropertyDescriptor(target1,&#39;foo&#39;)

    // { value: undefined,

    //   writable: true,

    //   enumerable: true,

    //   configurable: true }
```
上面代码中，source对象的foo属性的值是一个赋值函数，Object.assign方法将这个属性拷贝给target1对象，结果该属性的值变成了undefined。这是因为Object.assign方法总是拷贝一个属性的值，而不会拷贝它背后的赋值方法或取值方法。

这时，Object.getOwnPropertyDescriptors()方法配合Object.defineProperties()方法，就可以实现正确拷贝。
```javascript
    const source ={

        setfoo(value){

            console.log(value);

        }

    };

    const target2 ={};

    Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));

    Object.getOwnPropertyDescriptor(target2, &#39;foo&#39;)

    // { get: undefined,

    //   set: [Function: set foo],

    //   enumerable: true,

    //   configurable: true }
```
Object.getOwnPropertyDescriptors()方法的另一个用处，是配合Object.create()方法，将对象属性克隆到一个新对象。这属于浅拷贝。
```javascript
    const clone = Object.create(Object.getPrototypeOf(obj),

    Object.getOwnPropertyDescriptors(obj));
```
另外，Object.getOwnPropertyDescriptors()方法可以实现一个对象继承另一个对象。
```javascript
    const obj = Object.create(

        prot,

        Object.getOwnPropertyDescriptors({

            foo:123,

        })

    )
```
### 4.\_\_proto\_\_属性，Object.setPrototypeOf()，Object.getPrototypeOf()

JavaScript 语言的对象继承是通过原型链实现的。ES6 提供了更多原型对象的操作方法。

### （1）Object.setPrototypeOf()

Object.setPrototypeOf方法的作用与\_\_proto\_\_相同，用来设置一个对象的prototype对象，返回参数对象本身。它是 ES6 正式推荐的设置原型对象的方法。
```javascript
    // 格式

    Object.setPrototypeOf(object, prototype)

    // 用法

    const o = Object.setPrototypeOf({},null);
```
该方法等同于下面的函数。
```javascript
    functionsetPrototypeOf(obj, proto){

    obj.__proto__ = proto;

    return obj;

    }
```
下面是一个例子。
```javascript
    let proto ={};

    let obj ={ x:10};

    Object.setPrototypeOf(obj, proto);

    proto.y =20;

    proto.z =40;

    obj.x // 10

    obj.y // 20

    obj.z // 40
```
上面代码将proto对象设为obj对象的原型，所以从obj对象可以读取proto对象的属性。

如果第一个参数不是对象，会自动转为对象。但是由于返回的还是第一个参数，所以这个操作不会产生任何效果。

### （2）Object.getPrototypeOf()

该方法与Object.setPrototypeOf方法配套，用于读取一个对象的原型对象。
```javascript
    functionRectangle(){

    // ...

    }

    const rec =newRectangle();

    Object.getPrototypeOf(rec)=== Rectangle.prototype

    // true
```
如果参数是undefined或null，它们无法转为对象，所以会报错。
```javascript
    Object.getPrototypeOf(null)

    // TypeError: Cannot convert undefined or null to object
```
### 5.Object.keys()，Object.values()，Object.entries()

#### （1）Object.keys()

ES5 的Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名。

ES2017 [引入](https://github.com/tc39/proposal-object-values-entries)了跟Object.keys配套的Object.values和Object.entries，作为遍历一个对象的补充手段，供for...of循环使用。
```javascript
    let{keys, values, entries}= Object;

    let obj ={ a:1, b:2, c:3};

    for(let key of keys(obj)){

        console.log(key); // a,b,c

    }

    for(let value of values(obj)){

        console.log(value); // 1, 2, 3

    }

    for(let[key, value] of entries(obj)){

        console.log([key, value]); // [a, 1], [b, 2], [c, 3]

    }
```
#### （2）Object.values()

Object.values方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值。
```javascript
    const obj ={ foo:'bar', baz:42};

    Object.values(obj)

    // ['bar', 42]
```
返回数组的成员顺序，与本章的《属性的遍历》部分介绍的排列规则一致。
```javascript
    const obj ={100:'a',2:'b',7:'c'};

    Object.values(obj)

    // ['b', 'c', 'a']
```
上面代码中，属性名为数值的属性，是按照数值大小，从小到大遍历的，因此返回的顺序是b、c、a。

Object.values只返回对象自身的可遍历属性。
```javascript
    const obj = Object.create({},{p:{value:42}});

    Object.values(obj) // []
```
上面代码中，Object.create方法的第二个参数添加的对象属性（属性p），如果不显式声明，默认是不可遍历的，因为p的属性描述对象的enumerable默认是false，Object.values不会返回这个属性。只要把enumerable改成true，Object.values就会返回属性p的值。
```javascript
    const obj = Object.create({},{p:

        {

            value:42,

            enumerable:true

        }

    });

    Object.values(obj) // [42]
```
Object.values会过滤属性名为 Symbol 值的属性
```javascript
    Object.values({[Symbol()]:123, foo:'abc'});

    // ['abc']
```
如果Object.values方法的参数是一个字符串，会返回各个字符组成的一个数组。
```javascript
    Object.values('foo')

    // ['f', 'o','o']
```
#### （3）Object.entries()

Object.entries()方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组。
```javascript
    const obj ={ foo:'bar', baz:42};

    Object.entries(obj)

    // [['foo', 'bar'], ['baz', 42] ]
```
除了返回值不一样，该方法的行为与Object.values基本一致。

如果原对象的属性名是一个 Symbol 值，该属性会被忽略。
```javascript
    Object.entries({[Symbol()]:123, foo:'abc'});

    // [[ 'foo', 'abc'] ]
```
上面代码中，原对象有两个属性，Object.entries只输出属性名非 Symbol 值的属性。将来可能会有Reflect.ownEntries()方法，返回对象自身的所有属性。

Object.entries的基本用途是遍历对象的属性。
```javascript
    let obj ={ one:1, two:2};

        for(let[k, v] of Object.entries(obj)){

        console.log(

            `${JSON.stringify(k)}: ${JSON.stringify(v)}`

        );

    }

    // one: 1

    // two: 2
```
Object.entries方法的另一个用处是，将对象转为真正的Map结构。
```javascript
    const obj ={ foo:'bar', baz:42};

    const map =newMap(Object.entries(obj));

    map // Map { foo: 'bar', baz: 42 }
```
### （4）Object.fromEntries()

Object.fromEntries()方法是Object.entries()的逆操作，用于将一个键值对数组转为对象。因此特别适合将 Map 结构转为对象。
```javascript
    // 例一

    const entries =newMap([

        ['foo','bar'],

        ['baz',42]

    ]);

    Object.fromEntries(entries)

    // { foo: 'bar', baz: 42 }

    // 例二

    const map =newMap().set('foo',true).set('bar',false);

    Object.fromEntries(map)

    // { foo: true, bar: false }
```