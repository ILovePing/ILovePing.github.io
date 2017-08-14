---
title: ES6基础入门小结
date: 2016-11-04 19:05:43
categories: 技术
tags: 前端 es6
---
* **let** 块级作用域、不存在变量提升、暂时性死区、不允许重复声明
> 暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。
* **const** 声明一个只读的常量
        const a = [];
        a.push({a:1});//es6下可执行
        a = [{a:2}];//报错，因此如果想要用const声明的对象完全冻结，例如数组[]也无法push，可以使用      Object.freeze()方法

        const foo = Object.freeze([]);
        foo.a = 1;//常规模式下执行不起作用，严格模式下报错
下面是一个将对象彻底冻结的函数。
        var constantize = (obj) => { 
              Object.freeze(obj);//冻结父节点对象
              Object.keys(obj).forEach( (key, value) => { //遍历子节点
                      if ( typeof obj[key] === 'object' ) { //如果子节点也是对象的话，递归调用本函数
                              constantize( obj[key] ); 
                      } 
               });
            };
<!-- more -->
 tips:在全局里let定义的变量不是全局window对象的属性，而var定义的则是。

* 变量解构赋值含义
        let {x} = {x:1};
        let {sin,cos,tan} = Math;  
        //x = 1;sin = Math.sin;cos = Math.cos;tan = Math.tan;
        let [a1,a2,[a3],a4=1,...a5]=[1,[2,3],[4]];
        //a1 = 1;a2 = [2,3];a3 = 4; a4 = 1;a5 = [];如果解构不成功，变量就是undefined
        let {length:len} = 'hello';//len = hello.length = 5
>**解构本质**，就是把等号右侧的变量转成object，然后调用Iterator接口遍历对象进行赋值。
> * rest element (...延展对象)要放在最后一个参数里面
> * 如果一个数组成员不严格等于undefined的话，默认值是不会生效的。
> * 可以使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。

* 变量解构赋值**用途**
> 1. **交换变量**
         [x,y] = [y,x]
> * **从函数返回多个值**
        function a(){return [1,2,3];} var [a,b,c] = a();
> * **函数参数的定义**
        function f([x, y, z]) { ... }f([1, 2, 3]);
> * **提取JSON数据**
        var jsonData = { id: 42, status: "OK", data: [867, 5309]};
        let { id, status, data: number } = jsonData;
> * **函数参数的默认值**
        jQuery.ajax = function (url, { 
                async = true, 
                beforeSend = function () {}, 
                cache = true, 
                complete = function () {}, 
                crossDomain = false, 
                global = true,  
                // ... more config}) {  
            // ... do stuff
        };
> * **遍历Map结构**
        for (let [key, value] of map) { console.log(key + " is " + value);}
        // 获取键名
        for (let [key] of map) {  // ...}
        // 获取键值
        for (let [,value] of map) {  // ...}
> * **输入模块的指定方法**
         const { SourceMapConsumer, SourceNode } = require("source-map");
* 字符串扩展
挖坑待填。。。
* 正则扩展
 regexp方法第一个参数为正则表达式时，第二个状态参数会覆盖第一个正则的状态。

        new RegExp(/abc/ig, 'i').flags
        // "i"
         new RegExp(/abc/ig, 'i').source
        //abc
* 数组扩展
**Array.from()**
该方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）。
        Array.from(arrayLike, x => x * x);//第二个参数类似于.map方法，返回每个值的平方
**Array.of()**
该方法用于将一组值转换成数组。基本可以用来替代Array()构造函数。
该方法可以用以下代码模拟实现。
        function ArrayOf(){ return [].slice.call(arguments);}
**[].copyWithin.call(array,target[, start[, end])**
该方法对array数组从start(默认为从下标为0开始)到end替换到从target下标开始位置，覆盖原有成员、
        [].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
        // Int32Array [4, 2, 3, 4, 5]
**[...arguments].find**
该方法用于找到第一个符合条件的对象。
eg1.
         [1, 4, -5, 10].find((n) => n < 0)//-5
eg2.
         [1, 5, 10, 15].find(function(value, index, arr) { return value > 9;}) // 10
        //回调里的三个参数分别为当前值，当前值的索引，目标原数组
eg3.
        [NaN].indexOf(NaN)// -1,indexOf发现不了NaN
        [NaN].findIndex(y => Object.is(NaN, y))// 0,findIndex可以借助Object方法来识别数组里有无NaN
**[...arguments].fill()**
填充数组，可接受三个参数，只有一个参数的话填充数组的值所有都为该参数，接受第二和第三个参数为填充的起始位置和结束位置。
         ['a', 'b', 'c'].fill(7, 1, 2)// ['a', 7, 'c']
**数组实例的entries(),keys()和values()**
分别为返回数组实例的键值对，键名和键值，类似于java的语法。都返回一个Iterator接口，可以使用let of 来遍历。当然也可以使用next()来进行手动调用遍历。
        for (let index of ['a', 'b'].keys()) { console.log(index);}
        // 0// 1
        for (let elem of ['a', 'b'].values()) { console.log(elem);}
        // 'a'
        // 'b'
        for (let [index, elem] of ['a', 'b'].entries()) { console.log(index, elem);}
        // 0 "a"
        // 1 "b"
**数组实例的includes**
Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。该方法属于ES7，但Babel转码器已经支持。

* 函数的扩展
1.函数参数里可以直接写等于默认值function(x=0,y=1){}
2.函数参数的默认值可以和结构相结合。
eg1.
        // 写法一
        function m1({x = 0, y = 0} = {}) { return [x, y];}
        // 写法二
        function m2({x, y} = { x: 0, y: 0 }) { return [x, y];}
        //上面两种写法都对函数的参数设定了默认值，区别是写法一函数参数的默认值是空对象，但是设置了对象解构赋值的默认值；写法二函数参数的默认值是一个有具体属性的对象，但是没有设置对象解构赋值的默认值。


![执行结果.png](http://upload-images.jianshu.io/upload_images/2570325-81df80438133e226.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3.函数的length属性，返回没有指定默认值的参数个数。如果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了。
4.利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2570325-973f3b4649cabf9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5.rest参数function(...items)将多余的参数放进数组里面。
6.
...运算符，类似于rest参数的逆运算，将数组转为逗号个隔开的参数序列。
应用：6.1. 可以用来替代apply方法.

        // ES5的写法Math.max.apply(null, [14, 3, 77])
        // ES6的写法Math.max(...[14, 3, 77])
        // 等同于Math.max(14, 3, 77);

6.2.合并数组。

        var a = [1,2];
         var b = [3,4];
        var b = [5,6];
        [...a,...b,...c]
        //[1,2,3,4,5,6]

6.3.和解构赋值结合

        [a,...rest] = [1,2,3,4,5]
        //a= 1;rest = [2,3,4,5]

6.4.字符串的转换，涉及到unicode32的运算的都建议使用...来解析成数组，否则转换错误。
ps：本质上对于具有iterator接口的对象建议都使用...来转化成数组对象。

7.箭头函数（很常见）
常见的用法就不列举了，这里说一下一个重要的特性，箭头函数自己是没有this的，所以它都是让this指向定义时所在的最外层的作用域，这样有利于封装回调函数。

8.函数绑定。
es7里的提案 ::
挖坑待填。。。

9.尾调用、尾递归优化。
##尾调用优化##
        //函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。
        function f() { let m = 1; let n = 2; return g(m + n);}f();
        // 等同于
        function f() { return g(3);}f();
        // 等同于
        g(3);

`以上就是一个简单的尾调用优化过程。直接优化成执行g(3)这样就不用使用内存来保存f(x)的调用帧，大大节省内存。否则必须要g()执行完毕g的调用帧删除之后才能删除f的调用帧。ps：当然如果函数内部用到了外层的内部变量那就不能进行尾调用优化了`

##尾递归##
函数逻辑循环能递归尽量递归，这样一直都是用一个自身的调用栈，复杂度为0(1),所以在es6中，要使用尾递归，就不会发生堆栈溢出。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2570325-1ce3daa48dd53f78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 对象的扩展