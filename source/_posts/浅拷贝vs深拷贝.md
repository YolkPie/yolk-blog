---
title: 浅拷贝vs深拷贝
date: 2021-03-10 10:08:07
cover: https://m.360buyimg.com/img/jfs/t1/168731/24/11527/11836/60482c1bE6bd47446/524c8b349cb6e7b3.png
---

# 浅拷贝vs深拷贝

本文主要讲一下javascript的基本数据类型以及一些堆和栈的知识和什么是深拷贝、浅拷贝，深拷贝与浅拷贝的区别，以及怎么进行深拷贝和怎么进行浅拷贝。

## 堆和栈
深拷贝和浅拷贝的主要区别在内存中的存储类型不同。堆和栈都是内存中划分出来用来存储的区域。栈（stack）为自动分配的内存空间，它由系统自动释放；而堆（heap）则是动态分配的内存，大小不定也不会自动释放。

## ECMAScript的数据类型
首先先说一下ECMAScript的数据类型。
### 基本数据类型
- undefined
- boolean
- number
- string
- null
  
基本数据类型存放在栈中。存放在栈内存中的简单数据段，数据大小确定，内存空间大小可以分配，是直接按值存放的，所以可以直接访问。

#### 基本数据类型值不可变

js中的原始值（undefined、null、布尔值、数字和字符串）与对象（包括数组和函数）有着根本区别。原始值是不可更改的。

基本数据类型的值是不可变的，动态修改了基本数据类型的值，它的原始值也是不会改变的，例如：
```
let str = "abc";
console.log(str[1]="f");    // f
console.log(str);           // abc
```

#### 基本类型的比较是值的比较
基本类型的比较是值的比较，只要它们的值相等就认为他们是相等的，例如:
```
let a = 1, b = 1;
console.log(a === b);  //true
```

### 引用类型

#### 引用类型存放在堆中
引用类型（object）是存放在堆内存中的，变量实际上是一个存放在栈内存的指针，这个指针指向堆内存中的地址。每个空间大小不一样，要根据情况开进行特定的分配，例如:
```
let person1 = {name:'jozo'};
let person2 = {name:'xiaom'};
let person3 = {name:'xiaoq'};
```
![](https://m.360buyimg.com/img/jfs/t1/166090/31/5915/69760/601fd463E65160a6e/6f84d4dbcd475154.jpg)

#### 引用类型值可变
例如：
```
var a = [1,2,3];
a[1] = 5;
console.log(a[1]); // 5
```
#### 引用类型的比较是引用的比较
所以每次我们对 js 中的引用类型进行操作的时候，都是操作其对象的引用（保存在栈内存中的指针），所以比较两个引用类型，是看其的引用是否指向同一个对象。例如：
```
let a = [1,2,3], b = [1,2,3];
console.log(a === b);  //false
```
![](https://m.360buyimg.com/img/jfs/t1/153245/23/18034/51108/601fd528E492b7b56/6d14b845e7c8f7e4.jpg)

### 传值与传址
了解了基本数据类型与引用类型的区别之后，我们就应该能明白传值与传址的区别了。在进行赋值操作的时候，基本数据类型的赋值（=）是在内存中新开辟一段栈内存，然后再把再将值赋值到新的栈中。例如：
```
let a = 10, b = a;
a ++ ;
console.log(a); // 11
console.log(b); // 10
```
![](https://m.360buyimg.com/img/jfs/t1/161856/11/5971/48056/601fd5d5Eefd12f79/bcdf1d5896d925f0.jpg)

而引用类型的赋值是传址。只是改变指针的指向，例如：
```
var a = {}; // a保存了一个空对象的实例
var b = a;  // a和b都指向了这个空对象

a.name = 'jozo';
console.log(a.name); // 'jozo'
console.log(b.name); // 'jozo'

b.age = 22;
console.log(b.age);// 22
console.log(a.age);// 22

console.log(a == b);// true
```
![](https://m.360buyimg.com/img/jfs/t1/166818/18/5850/88832/601fd6a6E0e87f529/4bcaa52d018dd200.jpg)

### 浅拷贝
#### 赋值（=）和浅拷贝的区别
以下例说明：
```
var obj1 = {
    'name' : 'zhangsan',
    'age' :  '18',
    'language' : [1,[2,3],[4,5]],
};

var obj2 = obj1;


var obj3 = shallowCopy(obj1);
function shallowCopy(src) {
    var dst = {};
    for (var prop in src) {
        if (src.hasOwnProperty(prop)) {
            dst[prop] = src[prop];
        }
    }
    return dst;
}

obj2.name = "lisi";
obj3.age = "20";

obj2.language[1] = ["二","三"];
obj3.language[2] = ["四","五"];

console.log(obj1);  
//obj1 = {
//    'name' : 'lisi',
//    'age' :  '18',
//    'language' : [1,["二","三"],["四","五"]],
//};

console.log(obj2);
//obj2 = {
//    'name' : 'lisi',
//    'age' :  '18',
//    'language' : [1,["二","三"],["四","五"]],
//};

console.log(obj3);
//obj3 = {
//    'name' : 'zhangsan',
//    'age' :  '20',
//    'language' : [1,["二","三"],["四","五"]],
//};
```

先定义个一个原始的对象 obj1，然后使用赋值得到第二个对象 obj2，然后通过浅拷贝，将 obj1 里面的属性都赋值到 obj3 中。也就是说：

obj1：原始数据
obj2：赋值操作得到
obj3：浅拷贝得到

然后我们改变 obj2 的 name 属性和 obj3 的 name 属性，可以看到，改变赋值得到的对象 obj2 同时也会改变原始值 obj1，而改变浅拷贝得到的的 obj3 则不会改变原始对象 obj1。这就可以说明赋值得到的对象 obj2 只是将指针改变，其引用的仍然是同一个对象，而浅拷贝得到的的 obj3 则是重新创建了新对象。
然而，我们接下来来看一下改变引用类型会是什么情况呢，我又改变了赋值得到的对象 obj2 和浅拷贝得到的 obj3 中的 language 属性的第二个值和第三个值（language 是一个数组，也就是引用类型）。结果见输出，可以看出来，无论是修改赋值得到的对象 obj2 和浅拷贝得到的 obj3 都会改变原始数据。
这是因为浅拷贝只复制一层对象的属性，并不包括对象里面的为引用类型的数据。所以就会出现改变浅拷贝得到的 obj3 中的引用类型时，会使原始数据得到改变。

深拷贝：将 B 对象拷贝到 A 对象中，包括 B 里面的子对象，

浅拷贝：将 B 对象拷贝到 A 对象中，但不包括 B 里面的子对象
![](https://m.360buyimg.com/img/jfs/t1/153006/4/18335/127130/601fd8aaE7f1845aa/d855d9ba0b486d45.jpg)
### 深拷贝
深拷贝就是对对象以及对象的所有子对象进行拷贝。
怎么进行深拷贝呢？
思路就是递归调用刚刚的浅拷贝，把所有属于对象的属性类型都遍历赋给另一个对象即可。

#### 扩展运算符是深拷贝还是浅拷贝
以下例说明：
```
let arr = [1, 2, 3, 4, 5, 6];
let arr1 = [...arr];
arr1.push(7);
console.log(arr); //[1, 2, 3, 4, 5, 6]
console.log(arr1); //[1, 2, 3, 4, 5, 6, 7]
```
当数组是一维数组时，扩展运算符可以进行完全深拷贝，改变拷贝后数组的值并不会影响拷贝源的值。但是，当数组为多维时：例如
```
let arr = [1, 2, 3, 4, 5, 6, [1, 2, 3]];
let arr1 = [...arr];
arr1.push(7);
arr1[arr1.length - 2][0] = 100;
console.log(arr); //[1, 2, 3, 4, 5, 6,[100, 2, 3]]
console.log(arr1); //[1, 2, 3, 4, 5, 6, [100, 2, 3],7]
```
由此可见，不难发现当改变拷贝后数组中第二层数组的值时，则拷贝前数组第二层数组的值也跟着改变了。

结论：扩展运算符，如果只是一层数组或是对象，其元素只是简单类型的元素，那么属于深拷贝（就是一层拷贝，可以理解为深拷贝吧！）
如果数组或对象中的元素是引用类型的元素，那么就是浅拷贝。

#### JSON.parse(JSON.stringfy(xxx))来实现深拷贝
利用JSON.parse(JSON.stringfy(xxx))也可实现深拷贝
`注意：JSON.parse(JSON.stringfy(xxx))的方法，如果变量中含有Promise对象，则不可以使用该方法。`
