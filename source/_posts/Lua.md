---
title: Lua
date: 2021-04-01 10:55:31
keywords: Lua
cover: https://m.360buyimg.com/img/jfs/t1/173862/25/1441/3103/606534b0E2b798e76/9354c85c780d6d86.png
---

# Lua简介
Lua 是一种轻量小巧的脚本语言，用标准C语言编写并以源代码形式开放， 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。是巴西里约热内卢天主教大学里的一个研究小组于 1993 年开发的。

## 设计目的
其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。

## 特性
- **轻量级:** 它用标准C语言编写并以源代码形式开放，编译后仅仅一百余K，可以很方便的嵌入别的程序里。
- **可扩展:** Lua提供了非常易于使用的扩展接口和机制：由宿主语言(通常是C或C++)提供这些功能，Lua可以使用它们，就像是本来就内置的功能一样。
- **其他特性:**
  + 支持面向过程编程和函数式编程；
  + 自动内存管理；只提供了一种通用类型的表（table），用它可以实现数组，哈希表，集合，对象；
  + 语言内置模式匹配；闭包(closure)；函数也可以看做一个值；提供多线程（协同进程，并非操作系统所支持的线程）支持；
  + 通过闭包和table可以很方便地支持面向对象编程所需要的一些关键机制，比如数据抽象，虚函数，继承和重载等。

## 应用场景
- 游戏开发
- 独立应用脚本
- Web 应用脚本
- 扩展和数据库插件如：MySQL Proxy 和 MySQL WorkBench
- 安全系统，如入侵检测系统

# 安装Lua
## Mac 系统
```curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz
tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make macosx test
make install
```
或者
```
brew install lua
```

## Windows 系统
>下载地址：https://github.com/rjpcomputing/luaforwindows/releases，
双击安装后即可在该环境下编写 Lua 程序并运行。
windows下你可以使用一个叫"SciTE"的IDE环境来执行lua程序。

![](https://m.360buyimg.com/img/jfs/t1/162297/36/15434/38802/6062bfedEc939a251/3a32aa64ba897736.png)

# 变量
## 变量定义
Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。
局部变量的作用域为从声明位置开始到所在语句块结束。
变量的默认值均为 nil。
```
a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end

joke()
print(c,d)          --> 5 nil

do
    local a = 6     -- 局部变量
    b = 6           -- 对局部变量重新赋值
    print(a,b);     --> 6 6
end

print(a,b)      --> 5 6
```
> do ... end这种表达方式只是一种定义局部变量和执行一个语句块的组合，没有其他特殊含义。

## 变量赋值
Lua 可以对一个变量进行赋值，也可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量。
```
a = 1 
i,j = 2,3
x,y,z = 4,5
print(a, i, j, x, y, z)
```
# 数据类型
Lua 是动态类型语言，变量不要类型定义,只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。
Lua 中有 8 个基本类型分别为：nil、boolean、number、string、userdata、function、thread 和 table。

|  数据类型   | 类型描述  |
|  :----:  | ----  |
| nil  | 只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）|
| boolean  | 包含两个值：false和true |
| number  | 表示双精度类型的实浮点数 |
| string  | 字符串由一对双引号或单引号来表示 |
| table  | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表 |
| function  | 由 C 或 Lua 编写的函数 |
| thread  | 表示执行的独立线路，用于执行协同程序 |
| userdata  | 表示任意存储在变量中的C数据结构 |

我们可以使用 type 函数测试给定变量或者值的类型：
```
print(type("Hello world"))      --> string
print(type(10.4*3))             --> number
print(type(print))              --> function
print(type(type))               --> function
print(type(true))               --> boolean
print(type(nil))                --> nil
print(type(type(X)))            --> string
```

## nil（空）
- nil 类型表示一种没有任何有效值，它只有一个值 -- nil，例如打印一个没有赋值的变量，便会输出一个 nil 值
- 对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉，执行下面代码就知：
```
a=1
print(a)
a=nil
print(a)
print(b)
```
- **nil 作比较时应该加上双引号 "**，试下如下代码
```
type(X)==nil
type(X)=="nil"
```
type(X)==nil 结果为 false 的原因是 type(X) 实质是返回的 "nil" 字符串，是一个 string 类型，不信你执行一下 `type(type(X))`

## boolean（布尔）
boolean 类型只有两个可选值：true和 false，Lua 把 false 和 nil 看作是 false，其他的都为 true，**数字 0 也是 true**:
```
if false or nil then
    print("至少有一个是 true")
else
    print("false 和 nil 都为 false")
end

if 0 then
    print("数字 0 是 true")
else
    print("数字 0 为 false")
end
```

## number（数字）
Lua 默认只有一种 number 类型 -- double（双精度）类型，以下几种写法都被看作是 number 类型：
```
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```
## string（字符串）
### 字符串的定义
- 字符串由一对双引号或单引号来表示：
```
string1 = "this is string1"
string2 = 'this is string2'
```
- 也可以用 2 个方括号 "[[]]" 来表示"一块"字符串：
```
html = [[
<html>
<head></head>
<body>
    <a href="http://www.baidu.com/">百度</a>
</body>
</html>
]]
print(html)
```
- 在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字,试运行如下代码:
```
print("2" + 6)
```
### 字符串常见的操作
Lua 提供了很多的方法来支持字符串的操作：
- 字符串全部转为大写字母。
```
string.upper('lua')
```
- 字符串全部转为小写字母。
```
string.lower('LUA')
```
- 在字符串中替换。
```
string.gsub("aaa","a","z",3); -- aaa要操作的字符串；a 被 z 替换 3 次 (如果忽略3，则表示全部替换)
```
- 在一个指定的目标字符串中搜索指定的内容,返回其具体位置。不存在则返回 nil。
```
string.find("Hello Lua user", "Lua") 
```
- 返回一个类似printf的格式化字符串
```
string.format("the value is:%d",4) -- %d, %i 接受一个数字并将其转化为有符号的整数格式
string.format("the value is:%f",9.990000) -- %f 接受一个数字并将其转化为浮点数格式
string.format("the value is:%s",'a') -- %s 接受一个字符串并按照给定的参数格式化该字符串
```
- 计算字符串长度。
```
str = 'abc'
string.len(str) 或者
print(#str)
```
- 连接两个字符串，使用的是 ..
```
print("abc".."xyz")
```
- 字符串截取
```
str='m.jd.com'
string.sub(str, 4, -1) -- str要操作的字符串；4 截取开始的位置；截取结束位置，默认为 -1，最后一个字符。
```

## table（表）
### table（表）的定义
在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:
```
-- 创建一个空的 table
local tbl1 = {}
 
-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```

Lua 中的表（table）其实是一个"关联数组"，数组的索引可以是数字或者是字符串:
```
a = {}
a["key"] = "value"
key = 10
a[key] = 22
a[key] = a[key] + 11
for k, v in pairs(a) do
    print(k .. " : " .. v)
end
```
- **不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引默认以 1 开始。**
```
local tbl = {"apple", "pear", "orange", "grape"}
for key, val in pairs(tbl) do
    print("Key", key)
end
```
- **table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。**
```
a = {}
for i = 1, 5 do
    a[i] = i
end
for key, val in pairs(a) do
  print(key, val)
end
a["key1"] = "val"
print(a["key1"])
print(a["key2"])
```

### table（表）的操作
- Table 连接
- 插入和移除
- Table 排序
- Table 最大值
#### **Table 连接**
可以使用 concat() 输出一个列表中元素连接成的字符串:
```
fruits = {"banana","orange","apple"}
-- 返回连接后的字符串
print("连接后的字符串 ",table.concat(fruits))

-- 指定连接字符
print("连接后的字符串 ",table.concat(fruits,", "))

-- 指定索引来连接
print("连接后的字符串 ",table.concat(fruits,", ", 2,3))
```
#### **插入和删除**
可以使用 insert() 和 remove() 来分别实现 table的插入和删除：
```
fruits = {"banana","orange","apple"}

-- 在末尾插入
table.insert(fruits,"mango")
print("索引为 4 的元素为 ",fruits[4])

-- 在索引为 2 的键处插入
table.insert(fruits,2,"grapes")
print("索引为 2 的元素为 ",fruits[2])


for key, val in pairs(fruits) do
    print(val)
end
table.remove(fruits)
print("------------")
for key, val in pairs(fruits) do
    print(val)
end
```

#### **Table 排序**
使用sort() 方法对 Table 进行排序，默认排序顺序是根据字符串Unicode码点。
```
fruits = {"banana","orange","apple","grapes"}
for k,v in ipairs(fruits) do
    print(k,v)
end

table.sort(fruits)
print("--------------------")
for k,v in ipairs(fruits) do
    print(k,v)
end
```
#### Table 最大值
table.maxn 在 Lua5.2 之后该方法已经不存在了。

## function（函数）
在 Lua 中，函数是被看作是"第一类值"，函数可以存在变量里。
> 第一类值表示函数与其他传统类型的值（例如数字和字符串类型）具有相同的权利。即函数可以存储在变量中，可以作为实参传递给其他函数，还可以作为其他函数的返回值，如下两段代码:
```
--函数可以存储在变量中:

local function f1()
    print("Hello")
end
--等价于
local f1 = function()
    print("Hello")
end
```

```
--以匿名函数的方式通过参数传递:

function testFun(tab,fun)
    for k ,v in pairs(tab) do
        print(fun(k,v));
    end
end

tab={key1="val1",key2="val2"};
testFun(tab,
    function(key,val)--匿名函数
        return key.."="..val;
    end
);
```

### 函数的定义
```
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```
- optional_function_scope: 该参数是可选的制定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。
- function_name: 指定函数名称。
- argument1, argument2, argument3..., argumentn: 函数参数，多个参数以逗号隔开，函数也可以不带参数。
- function_body: 函数体，函数中需要执行的代码语句块。
- result_params_comma_separated: 函数返回值，Lua函数可以返回多个值，每个值以逗号隔开。
#### 示例
- 以下实例定义了函数 max()，参数为 num1, num2，用于比较两值的大小，并返回最大值：
```
function max(num1, num2)
   if (num1 > num2) then
      result = num1;
   else
      result = num2;
   end
   return result;
end

-- 调用函数
print("两值比较最大值为 ",max(10,4))
```  
- 可以将函数作为参数传递给函数，如下实例：
```
myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1,num2,functionPrint)
   result = num1 + num2
   -- 调用传递的函数参数
   functionPrint(result)
end
-- myprint 函数作为参数传递
add(2,5,myprint)
```
### 多返回值
Lua函数可以返回多个结果值，在return后列出要返回的值的列表即可返回多值，如：
```
-- 获取table中的最大值

function maximum (a)
    local mi = 1             -- 最大值索引
    local m = a[mi]          -- 最大值
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5}))
```
### 可变参数
Lua 函数可以接受可变数目的参数。
- 在函数参数列表中使用三点 ... 表示函数有可变的参数；
- 可以通过 "#" 来获取可变参数的数量。
```
-- 求平均值

function average(...)
   result = 0
   local arg={...}    -- arg 为一个表，局部变量
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```


## thread（线程）
在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

|  方法   | 描述  |
|  :----:  | :----  |
| create() | 创建 coroutine，返回 coroutine， 参数是一个函数，当和 resume 配合使用的时候就唤醒函数调用 |
| resume() | 启动 coroutine，和 create 配合使用 |
| yield() | 挂起 coroutine，将 coroutine 设置为挂起状态，这个和 resume 配合使用能有很多有用的效果 |
| status() | 查看 coroutine 的状态 (coroutine 的状态有三种：dead，suspended，running) |
| running() | 返回正在跑的 coroutine，一个 coroutine 就是一个线程，当使用running的时候，就是返回一个 corouting 的线程号 |


### 简单示例
```
co = coroutine.create(
    function(i)
        print(i);
    end
)
coroutine.resume(co, 1)   -- 1
print(coroutine.status(co))  -- dead

print("-------------")

co2 = coroutine.create(
    function()
        for i=1,10 do
            print(i)
            if i == 3 then
                print(coroutine.status(co2))  --running
                print(coroutine.running()) --thread:XXXXXX
            end
            coroutine.yield()
        end
    end
)
 
coroutine.resume(co2) --1
coroutine.resume(co2) --2
coroutine.resume(co2) --3
 
print(coroutine.status(co2))   -- suspended
print(coroutine.running())
```
> 当create一个coroutine的时候就是在新线程中注册了一个事件。当使用resume触发事件的时候，create的coroutine函数就被执行了，当遇到yield的时候就代表挂起当前线程，等候再次resume触发事件。

### 详细示例
```
function foo (a)
    print("foo 函数输出", a)
    return coroutine.yield(2 * a) -- 返回  2*a 的值
end
 
co = coroutine.create(function (a , b)
    print("第一次协同程序执行输出", a, b) -- co-body 1 10
    local r = foo(a + 1)
     
    print("第二次协同程序执行输出", r)
    local r, s = coroutine.yield(a + b, a - b)  -- a，b的值为第一次调用协同程序时传入
     
    print("第三次协同程序执行输出", r, s)
    return b, "结束协同程序"                   -- b的值为第二次调用协同程序时传入
end)
       
print("main", coroutine.resume(co, 1, 10)) -- true, 4
print("--分割线----")
print("main", coroutine.resume(co, "r")) -- true 11 -9
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- true 10 end
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- cannot resume dead coroutine
print("---分割线---")
```

> resume和yield的配合强大之处在于，resume处于主程中，它将外部状态（数据）传入到协同程序内部；而yield则将内部的状态（数据）返回到主程中。

### 生产者-消费者问题
```
local newProductor

function productor()
    local i = 0
    while true do
        i = i + 1
        send(i)     -- 将生产的物品发送给消费者
    end
end

function consumer()
    while true do
        local i = receive()     -- 从生产者那里得到物品
        print(i)
    end
end

function receive()
    local status, value = coroutine.resume(newProductor)
    return value
end

function send(x)
    coroutine.yield(x)     -- x表示需要发送的值，值返回以后，就挂起该协同程序
end

-- 启动程序
newProductor = coroutine.create(productor)
consumer()
```

以上代码执行步骤：
- 创建newProductor协同程序
- 执行consumer函数
- 到consumer中执行recieve（）
- recieve中启动newProductor，执行productor函数
- 到productor中执行send（1）//生产者中i = 1
- send（1）挂起newProductor，并发送1
**注意协同程序下次从这里继续**
- 继续执行receive，value=1，
- 到consumer（）中local i=1，打印i
- 执行consumer（）中的while循环
- ......

## userdata（自定义类型）
userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。

# 数组
Lua并没有提供专门的数组对象来对数组进行操作，Lua的数组是使用table来实现的。当 key 为整数时，table 就可以当成数组来用。而且这个数组是一个 索引从1开始 ，没有固定长度，可以根据需要自动增长的数组。可以是一维数组和多维数组。

## Table的长度
### 对于一个数组我们通常可以使用#和table.getn来获取其长度
```
array = {"a", "b"}
print(#array)
```
### 如果不是数组，使用#是不能获取table长度的，可以使用如下方法进行获取：
```
tabletest = {a=1,b=2,c=3,d=4,e=5}
local count=0
for k,v in pairs(tabletest) do
    count = count + 1
end
print(count)
```

## 一维数组
```
array = {"a", "b"}
for i= 0, #array do
   print(array[i])
end
```
## 二维数组
多维数组即数组中包含数组或一维数组的索引键对应一个数组。
```
array = {{1,2},{'a','b'},3,'c'}
for i=1, #array do
  if (type(array[i]) == 'table') then 
    for j=1, #array[i] do
      print(array[i][j])
    end
  else
    print(array[i])
  end
end
```

# Lua 流程控制
Lua 编程语言流程控制语句通过程序设定一个或多个条件语句来设定。在条件为 true 时执行指定程序代码，在条件为 false 时执行其他指定代码。
## if 语句
```
--[ 0 为 true ]
if(0)
then
    print("0 为 true")
end
```
## if...else 语句
```
a = 100;

if( a < 20 )
then
   print("a 小于 20" )
else
   print("a 大于 20" )
end
```
## if 嵌套语句
```
a = 100;
b = 200;

if( a == 100 )
then
   if( b == 200 )
   then
      print("a 的值为 100 b 的值为 200" );
   end
end
```

# 循环
很多情况下我们需要做一些有规律性的重复操作，因此在程序中就需要重复执行某些语句。
一组被重复执行的语句称之为循环体，能否继续重复，决定循环的终止条件。
循环结构是在一定条件下反复执行某段程序的流程结构，被反复执行的程序被称为循环体。
循环语句是由循环体及循环的终止条件两部分组成的。
## for 循环
### 数值for循环
```
for var=exp1,exp2,exp3 do  
    <循环体>  
end  
```
> var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。

```
for i=1,10 do
    print(i)
end
```
### 泛型for循环
泛型 for 循环通过一个迭代器函数来遍历所有值。
```
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end 
```
> i是数组索引值，v是对应索引的元素值。ipairs是Lua提供的一个迭代器函数，用来迭代数组。
## while 循环
Lua 编程语言中 while 循环语句在判断条件为 true 时会重复执行循环体语句。
```
while(循环条件)
do
   <循环体>
end
```
```
a=10
while( a < 20 )
do
   print("a 的值为:", a)
   a = a+1
end
```

## 循环嵌套
以下实例使用了for循环嵌套:
```
for m=1,9 do                           
    local s = ""                       
    for n=1,9 do                       
        if n <= m then                 
            s = s..m.."x"..n.."="..m*n 
            if n ~= m then   -- ~= 不等于           
                s = s..", "            
            end
        end        
    end
    print(s)
end
```

## 循环控制语句
循环控制语句用于控制程序的流程， 以实现程序的各种结构方式。
- break
- goto
### **break**
Lua 编程语言 break 语句插入在循环体中，用于退出当前循环或语句，并开始脚本执行紧接着的语句。
如果你使用循环嵌套，break语句将停止最内层循环的执行，并开始执行的外层的循环语句。
```
for i=1,10 do
  if(i>5) 
  then 
    break
  else  
    print(i)
  end
end
```
### **goto**
goto 语句允许将控制流程无条件地转到被标记的语句处。
语法格式如下所示：
```
goto label
```
Label 的格式为：
```
:: Label ::
```

以下实例中使用了 goto：
```
for i=1, 3 do
    if i <= 2 then
        print(i, "yes continue")
        goto continue
    end
    print(i, " no continue")
    ::continue::
    print([[i'm end]])
end
```
