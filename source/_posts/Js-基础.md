---
title: Js 基础
date: 2021-02-15 15:13:35
tags:
categories: 技术
---

- 判断是否在视口内：

```js
ele.getBoundingClientRect().top > window.innerHeight; // 元素在当前屏下面

ele.getBoundingClientRect().bottom < 0; // 元素在当前屏上面
```

https://app.leanboard.io/board/c90672de-658b-4058-a4cf-adb28ad4bad5

- dom bom window document 概念

```
DOM 是为了操作文档出现的 API，document 是其的一个对象；
BOM 是为了操作浏览器出现的 API，window 是其的一个对象。
```

# 跨域

## CORS

### IE8：XDR(XDomainRequest)

- 通信不携带 cookie
- 只能设置请求头里的 Content-Type 字段
- 不支持访问响应头
- 只支持 GET 与 POST

### 其他浏览器：XHR 直接支持

- 不能使用 setRequestHeader 设置自定义头部

      - 场景：

  报错：“not allowed by Access-Control-Allow-Headers in preflight response”
  措施：服务端设置 Access-Control-Allow-Headers 即可接触限制

- 不能发送和接受 cookie

  - 设置 withCredentials 字段即可携带 cookie

    - 场景：antd 文件上传 配置 withCredentials 即可在上传文件的时候进行身份认证

- 调用 getAllResponseHeaders 永远返回空字符串

### 对跨域请求出于安全的浏览器实现

- 对服务器数据可能产生副作用的请求（非简单请求）需要先进行一次 OPTIONS 方法的预检请求（preflight request）

  - 简单请求

    - REQUEST METHOD

      - GET、HEAD、POST

    - CONTENT-TYPE 属于

      - text/plain
      - multipart/form-data
      - applacation/x-www-form-urlencoded

  - 非简单请求（满足其中之一条件即可）

    - REQUEST METHOD

      - PUT、DELETE、CONNECT、OPTIONS、TRACE、PATCH

    - 人为设置除以下请求头以外的 header 字段：

      - Accept
      - Accept-Language
      - Content-Language
      - Content-Type
      - DPR
      - Downlink
      - Save-Data
      - Viewport-Width
      - Width

    - CONTENT-TYPE 不属于

      - text/plain
      - multipart/form-data
      - applacation/x-www-form-urlencoded

## 图像 Ping

### img src 属性对不同域资源不做限制

- 缺点

  - 只能 get 请求
  - 单向的，无法处理 responseText

- 场景

  - 埋点

## JSONP

### script src 属性对不同域资源不做限制

- 原理

      - get请求query传callback=JS定义的回调参数名称假定为func

  服务端 responseText=func(服务端插入 data)
  至此 JS 接收到请求会直接执行该方法，参数即服务端返回 data

- 优点

  - 双向的
  - 可以直接处理 responseText

- 缺点

  - 加载其他域代码 不安全
  - 不好确定请求是否失败 H5 onerror 方法兼容性不好

## COMET

### 释义：服务器向页面推送数据（单向的）的一种技术

### 实现方式

- 长轮询
- 流

  - XHR 监听 xhr.readyState === 3（此时为接收状态）周期性的往先前接收到的数据里拼接。

### SSE（EventSource）

- 释义：围绕 COMET 交互实现的 API

  - 服务器响应 MIMETYPE 必须为 text/event-stream
  - 创建到服务端的单向连接，服务器可以发送任意数量的数据
  - 支持短轮询，长轮询和 HTTP 流
  - 能在断开连接时自动确定何时重新连接

    - 每次请求都会返回一个 event-id，断开连接后，下次连接重新发送 Last-Event-Id 定位续传位置

## WEBSOCKET

### 提供双向的全双工通信的持久连接

### （ws:// | wss://）协议

- 自定义的协议相较于 Http 协议字节开销小很多

* ECMA-262 第五版在定义只有内部才用的特性（attribute）时，描述了属性的（property）的属性。
* 这些特性是为了实现 js 引擎使用的，所以 js 不能直接访问它们，所以规范中把它们放在了两对方括号中，例如[[Configurable]]。
* ECMAScript 中有两种属性：**数据属性**和**访问器属性**。 - - -
  1、数据属性（个人理解，数据属性包含一个数据值的位置，直接分配内存 allocation，在该内存上进行读取写入操作。）

      * `[[Configurable]]` 通过字面量定义时默认为true，通过`Object.defineProperty`时默认为false。表示能否通过delete删除属性从而重新定义属性（**此处属性指的都是对象属性，即遍历中的key，不是所谓的对象值value**），能否修改属性的特性，能否把属性修改为访问器属性。

  ![image](https://user-images.githubusercontent.com/13430709/68460528-0fd67a00-0243-11ea-9035-2d9f64cf8491.png)

      * `[[Enumrable]]` 通过字面量定义时默认为true，通过`Object.defineProperty`时默认为false，表示能否通过for in 循环遍历属性。


      * `[[writable]]` 通过字面量定义时默认为true，通过`Object.defineProperty`时默认为false，表示能否修改属性的值。


      * `[[value]]` 默认为undefined，表示当前属性数据值，读取和赋值属性值就是在这个位置上操作。

      - - -
      2、 访问器属性
      （不能直接定义，必须使用Object.definedProperty来定义.
      个人理解，访问器属性不包含数据值，**它是自身没有值的，可以理解为一个接口**指的是读取/写入该访问器属性这一过程的一些行为逻辑。）

      * `[[Configurable]]` 同上
      * `[[Enumrable]]` 同上
      * `[[Get]]` 在读取属性值时的函数，默认undefined
      * `[[Set]]` 在写入属性值时的函数，默认undefined

      在`Object.definedProperty` API之前，使用两个非标准方法`__defineGetter__()`和`__defineSetter__()`

---

`Object.defineProperties`
定义多个属性，可以同时为一个对象定义多个数据属性和访问器属性。
`Object.getOwnPropertyDescriptors`
获取对象所有数据属性和访问器属性的属性。

### 对象创建与继承

- 创建对象可以直接通过字面量创建`const a = {}`也可以通过构造函数`const a = new Object()`来创建
  考虑封装减少代码重复，所以万能的程序员想到了各种模式实现方法的创建。

#### 对象创建

---

- 工厂模式
  ![image](https://user-images.githubusercontent.com/13430709/68575652-752aa500-04a7-11ea-9287-277a7944ff90.png)
  缺点：无法识别对象类型（都是 Object new 出来的）
- 构造函数模式
  ![image](https://user-images.githubusercontent.com/13430709/68576945-14e93280-04aa-11ea-98fa-6816a4e1d774.png)
  缺点：实例中对函数方法都是和每个实例一一对应的，一是浪费内存，二是每次都得定义一遍。
- 原型模式
  ![image](https://user-images.githubusercontent.com/13430709/68578053-39460e80-04ac-11ea-8dc4-eab7f7f5b118.png)
  简易写法（不推荐）：
  ![image](https://user-images.githubusercontent.com/13430709/68579506-40bae700-04af-11ea-816e-51c4ca971a40.png)
  导致问题： constructor 丢失
  解决方法：重新手动指定 constructor
  ![image](https://user-images.githubusercontent.com/13430709/68579689-a909c880-04af-11ea-9a5f-ef3405478788.png)
  导致问题：这样指定会导致 constructor 也是[[Enumrable]]为 true，遍历时可以枚举出来的
  解决方法：使用 Object.defineProperty
  ![image](https://user-images.githubusercontent.com/13430709/68579797-fd14ad00-04af-11ea-9c40-7ee3fd03caf0.png)
  该模式缺点：单独使用原型模式共享所有的数据属性。
- 组合使用构造函数模式和原形模式（推荐，主流实现）
  ![image](https://user-images.githubusercontent.com/13430709/68580124-a9ef2a00-04b0-11ea-8684-099e5a1a0a8b.png)
  小缺点：其他语言 er 看的不舒服，方法定义在构造函数外面...
- 动态原型模式
  ![image](https://user-images.githubusercontent.com/13430709/68580201-d014ca00-04b0-11ea-9983-2610fb20911f.png)
- 寄生构造函数模式(不推荐)
  ![image](https://user-images.githubusercontent.com/13430709/68580608-aa3bf500-04b1-11ea-953e-b93d5413f1bc.png)
- 稳妥构造函数模式(与寄生构造类似，只是不使用 this 获取变量，适合在一些安全的环境中使用该模式)
  ![image](https://user-images.githubusercontent.com/13430709/68580738-eec79080-04b1-11ea-9b2c-5e0685d47bdd.png)

#### 对象继承(js 中就是依靠原型链实现「实现继承的」)

---

- 原型链
  ![image](https://user-images.githubusercontent.com/13430709/68635622-1e1ee180-0534-11ea-9850-b782e7f51e93.png)
  ![image](https://user-images.githubusercontent.com/13430709/68586666-1faec200-04c0-11ea-914c-fa105a0a78f0.png)
  问题 1： 原型链上父类构造函数如果定义了引用类型值（比如数组），那么对于使用子构造函数的每个实例，向上沿着原型链查找父类的引用类型值时指向的都是同一个原型属性。那么一个实例对于原型上的引用类型修改势必会反应到另一个实例上。如下图所示：
  ![image](https://user-images.githubusercontent.com/13430709/68636198-ace02e00-0535-11ea-9354-036407652c10.png)
  ![image](https://user-images.githubusercontent.com/13430709/68636182-a5b92000-0535-11ea-8f27-5fb294cb29e4.png)
  问题 2：可以看到单独使用原型链继承的话 无法向超类传参数，比如我 new 一个 Girl 继承自 Mankind，不能通过 Mankind 构造函数初始化 name，weight 和 height。

- 借用构造函数（使用 call/apply 借用超类构造函数）
  ![image](https://user-images.githubusercontent.com/13430709/68637553-efa40500-0539-11ea-9111-675fe32c0c77.png)
  ![image](https://user-images.githubusercontent.com/13430709/68637547-ec107e00-0539-11ea-9e36-b051d1024927.png)
  缺点：单独使用的话，就是单纯的执行父类构造函数，绑定当前构造函数的 this 而已，那么就回到了最开始上面的「构造函数模式创建对象」，缺点同理。

- 组合继承（推荐，最常用的继承方式，借用构造函数继承属性，通过原型链继承方法）
  ![image](https://user-images.githubusercontent.com/13430709/68638458-9b4e5480-053c-11ea-8490-02f545866485.png)
  ![image](https://user-images.githubusercontent.com/13430709/68638410-70fc9700-053c-11ea-8ade-e24a8dcefc52.png)
  缺点：**两次调用超类构造函数，**
  ![image](https://user-images.githubusercontent.com/13430709/68645550-613c7d00-0553-11ea-9444-eb0dd51d99ed.png)
  ![image](https://user-images.githubusercontent.com/13430709/68645620-921cb200-0553-11ea-846a-3eab7abab77a.png)
  ![image](https://user-images.githubusercontent.com/13430709/68645951-a6ad7a00-0554-11ea-9ba6-d71c997489f1.png)

- 原型式继承
  没有必要兴师动众使用构造函数创建对象，只不过想抽离公用代码，通过一个对西那个创建几个类似的对象时，可以考虑使用原型式继承，ES6 新增`Object.create`方法规范化了该继承方式。
  ![image](https://user-images.githubusercontent.com/13430709/68638865-db620700-053d-11ea-8bef-580407235214.png)
  ![image](https://user-images.githubusercontent.com/13430709/68638858-d69d5300-053d-11ea-9e6b-4d2a4df28c28.png)
  缺点：引用类型值始终是被共享的，就跟原型模式创建对象一样，一处改动，其他实例跟着受影响。

- 寄生式继承
  ![image](https://user-images.githubusercontent.com/13430709/68639465-9c34b580-053f-11ea-98ab-8f5a7c8067cc.png)
  缺点：一个缺点同上，引用类型值共享，另一个缺点是函数复用率底，同构造函数模式。

- 寄生组合式继承
  由于上面列过的 组合继承的缺点（执行两次超类构造），因此可以使用寄生组合继承规避该缺点。
  原理：在组合继承中，其实不必为了指定子类型的原型而去调用超类的构造函数，即`Woman.prototype = new Mankind();`其实没必要的，我们只不过是为了一个超类原型的一个副本，因此我们可以使用寄生式继承方式来继承超类原型，然后将结果指定给子类型的原型。

![image](https://user-images.githubusercontent.com/13430709/68646194-66023080-0555-11ea-936a-676ec58765fe.png)
![image](https://user-images.githubusercontent.com/13430709/68646206-70242f00-0555-11ea-9365-707e4ceb5d2d.png)

call 实现：

```js
Function.prototype._call = function (context = {}, ...args) {
  context.fn = this;
  const res = context.fn(...args);
  delete context.fn;
  return res;
};
```

数值转换

value -> number，几种方式：Number，parseInt，parseFloat。

- Number(一元加号操作符效果同 Number 函数)
  - if value is `boolean`, true -> 1,false -> 0
  - if value is `number`, value -> value
  - if value is `null`, value -> 0
  - if value is `undefined`, value -> NaN
  - if value is `string`
    - if stringValue only contains digit(+/- prefix symbol also counts), example:
      1. "+12" -> 12
      2. "001" -> 1
      3. "-02" -> -2
      4. "+1.2" -> 1.2
      5. "-.2" -> -0.2
    - if stringValue is valid hexadecimal/binary, turn it to decimal,example:
      1.  "0xf" -> 15
      2.  "0b111" -> 7
    - if stringValue is empty, "" -> 0
    - if stringValue contains character not mentioned above, return NaN,example:
      1.  "--2" -> NaN
      2.  "d2" -> NaN
- ParseInt
  example：

  1.  parseInt("1234ddaa") -> 1234
  2.  parseInt("") -> NaN
  3.  parseInt("0xA") -> 10 // hexadecimal
  4.  parseInt("070") -> 56 // octal ecma3 version
  5.  parseInt("070") -> 70 // ecma5 version / strict mode
  6.  parseInt(22.2) -> 22.2
  7.  parseInt("d12") -> NaN
  8.  parseInt("1d12") -> 1

  从上述 example4 和 5 可以看到不同版本的 js 引擎解析八进制是不同的，在 ES5 和严格模式下，0 开头的 string 前导 0 会被忽略。

  因此 parseInt 可以穿入第二个参数，代表转换成多少进制。example:

  1.  parseInt("0xAF",16) -> 175
  2.  parseInt("AF",16) -> 175
  3.  parseInt("AF") -> NaN
  4.  parseInt("10",2) -> 2
  5.  parseInt("10", 8) -> 8
  6.  parseInt("10", 10) -> 10

- parseFloat(只解析十进制，十六进制的都返回 0)
  example：
  1.  parseFloat("1234ddaa") -> 1234
  2.  parseFloat("") -> NaN
  3.  parseFloat("0xFA") -> 0
  4.  parseFloat("1.2.3") -> 1.2 // 只解析到第二个小数点前
  5.  parseFloat(".2.3") -> 0.2
  6.  parseFloat("12.0") -> 12
  7.  parseFloat("3.125e7") -> 31250000//科学计数法

---

value -> string

- toString（可传入参数转换为几进制，默认十进制）
  primitive value example:
  1.  0xAF.toString() -> "175"
  2.  0xAF.toString(16) -> "af"
  3.  12.toString() -> "12"
  4.  12.2.toString() -> "12.2"
  5.  0.1.toString(2) -> "0.0001100110011001100110011001100110011001100110011001101"
      reference value example:
  6.  [1,2].toString() -> "1,2"
  7.  ({}).toString(16) -> "[object Object]" 3. new Set().toString() -> "[object Set]" // new Set()[Symbol.toStringTag] === "Set" 4. new Map().toString() -> "[object Map]" 5. new WeakMap().toString() -> "[object WeakMap]" 6. new WeakSet().toString() -> "[object WeakSet]" 7. Symbol('1s').toString() -> "Symbol(1s)"
- String()
  primitive example:
  1.  String(null) -> "null"
  2.  String(true) -> "true"
  3.  String(a) -> "undefined"
      reference value example:
      same as above.
      js 中 number 的表示才用了 IEEE754 规范的双精度浮点数（64 位），字节分配如下，分别是 1 位 sign 符号位，11 位 exponent 指数位，52 位 fraction 分数位。
      ![image](https://user-images.githubusercontent.com/13430709/69146908-cbcd5a00-0b0b-11ea-8ed2-a187876ca46e.png)
      经常有讨论 js 精度丢失造成的 0.1+0.2!=0.3 的问题就是缘由此。
      如下：0.1+0.2 首先会将十进制转换为二进制（十进制小数如何表示为二进制这里不做实现讨论）
      0.1 -> 0.0001100110011001...(无限循环，需要截取多余位，这里就有精度丢失，对应的双精度 64 位表示为：
      ![image](https://user-images.githubusercontent.com/13430709/69147305-9d9c4a00-0b0c-11ea-8eeb-0e87849b2fe9.png)
      0.2 -> 0.0011001100110011...(无限循环，需要截取多余位，这里就有精度丢失，对应的双精度 64 位表示为：
      ![image](https://user-images.githubusercontent.com/13430709/69147325-a7be4880-0b0c-11ea-92e7-b721981a007d.png)
      然后是两者相加的对阶运算，这里也有精度丢失
      0.0100110011001100110011001100110011001100110011001100
      转换为十进制就是 0.30000000000000004，所以不相等。

一般处理这种 js 计算精度丢失的问题，原理就是取小数位数的最大数作为放大倍数，将小数放大为整数操作后缩小回去，自己实现考虑的不完善，当然也有第三方库 Math.js,big.js 等。
