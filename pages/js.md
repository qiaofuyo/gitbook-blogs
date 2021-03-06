# JavaScript的基础理论

## JavaScript的执行过程

> js引擎的执行过程分为：`编译阶段` 和 `执行阶段`。

### 编译阶段

> 编译阶段 = 预解析（词法分析+语法分析） + 生成代码

预解析机制 = 变量预解析 + 函数预解析：

- 变量预解析：仅提升变量的 `声明` 到 `当前作用域` 的最前面（不包括let/const，因为它们在执行阶段初始化）。

- 函数预解析：仅提升函数的 `声明` 到 `当前作用域` 的最前面。

先提升变量声明，后提升函数声明，导致 `函数` 是js的 `一等公民`。

```js
console.log(a); // ƒ a() { return 1 }
function a() { return 1 }
var a = 'a'

console.log(b); // ƒ b() { return 1 }
var b = 'b'
function b() { return 1 }
```

变量的赋值可以分为三个阶段：

1. 创建（声明）变量，在内存中开辟空间；

2. 初始化变量，将变量初始化为 `undefined`；

3. 执行阶段才真正赋值。

关于`let` 、`const`、`var` 和 `function` 在编译阶段的处理：

- `let` 、`const` 的创建过程被提升了，但是初始化没有提升。
- `var` 的创建和初始化都被提升了。

- `function` 的创建、初始化、赋值都被提升了。

```js
// 分别用let、const、var声明变量，观察变量在栈中的变化
debugger
let a = 'xg'
{
    console.log(a) // Cannot access 'a' before initialization
    let a = 'xg1'
}
```

### 执行阶段

> 事件循环Event Loop是一个程序结构，用于等待和发送消息和事件。即 `js的执行机制`。

#### 单线程

`JavaScript` 是一门 `单线程` 语言，所有任务都在一个线程上 `排队执行`，不管是什么新框架新语法糖实现的所谓异步，其实都是用 `同步` 的方法去模拟的。

如果是多线程则会引发复杂的同步问题，比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准，为了避免复杂性所以JavaScript是单线程的。

为了利用多核CPU的计算能力，HTML5提出  [**Web Worker**](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API)，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

#### 同步任务、异步任务

> js中所有任务可分为 `同步任务` 和 `异步任务`。

**同步任务**

同步任务指不会被引擎挂起，在主线程上排队执行的任务。排队（压栈）形成了 `执行栈`，只有栈顶任务执行完毕出栈后，才能执行下一任务。

Note:  `promise` 的实例化的过程是属于同步的，而 `.then` 中的回调函数是属于异步的。

**异步任务**

> 又分为 `宏任务` 和 `微任务`。
>
> 包含：IO设备的事件，用户产生的鼠标点击、页面滚动等事件，ajax请求，setTimeOut() ... ...

异步任务指会被引擎挂起，不进入主线程，而是进入`任务队列`（task queue）的任务。不需要获取到异步结果也能执行下一任务，获取到异步结果后由引擎来决定其回调函数是否进入主线程执行。

#### 任务队列

JavaScript 运行时，除了一个正在运行的主线程，引擎还提供一个`任务队列`（task queue），里面是各种需要当前程序处理的异步任务的回调函数。（根据异步任务的类型，实际存在多个任务队列。为了方便理解，这里假设只存在一个队列。）

1. 主线程会先执行所有的同步任务，遇到异步任务的代码先不予执行；
2. 执行完同步任务（由js引擎的`监控进程`检测执行栈是否为空）后，如果任务队列中没有执行完的异步任务的 `回调函数`，则会去执行其他异步任务；如果任务队列中有执行完的异步任务的 `回调函数`，则让其进入主线程作为同步任务立即执行；
3. 重复过程1、2，这就是 `事件循环`。

![事件循环流程图](https://i.loli.net/2021/10/04/r6exoPV1Z32KDmT.png)

`回调函数` 是指会被主线程 `挂起来` 的代码，异步任务必须指定回调函数，主线程执行异步任务就是执行其对应的回调函数。

```js
<script src="http://libs.baidu.com/jquery/2.0.0/jquery.min.js"></script>
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

<script type="text/javascript">
  $(document).ready(function() {
      $("button").click(function() {
          init()
      });
  });

  function init() {
      debugger
      console.log(1);
      setTimeout(function() {
          console.log(2);
      }, 1000);
      axios.get('http://39.107.221.146/api/homeslide')
          .then(function(response) {
          console.log(3);
      })
          .catch(function(error) {
          console.log(4);
      });
      console.log(5);
      axios.get('http://39.107.221.146/api/homenav')
          .then(function(response) {
          console.log(6);
      })
          .catch(function(error) {
          console.log(7);
      });
      console.log(8);
  }
</script>
```

#### 宏任务、微任务

> 异步任务分为 `宏任务` 和 `微任务`。

宏任务、微任务执行过程：

- 遇到微任务时，在处理完当前任务后率先执行遇到的微任务；
- 遇到宏任务时，当前宏任务执行完后才会执行遇到的宏任务。

宏任务是宿主级别，微任务是JS级别，它们是包含关系，所以先执行 `宏任务` 然后逐条执行当前所有的 `微任务`，然后执行下一条 `宏任务`。

**宏任务包含：**

| #                       | 浏览器 | Node |
| ----------------------- | ------ | ---- |
| `I/O`                   | ✅      | ✅    |
| `setTimeout`            | ✅      | ✅    |
| `setInterval`           | ✅      | ✅    |
| `setImmediate`          | ❌      | ✅    |
| `requestAnimationFrame` | ✅      | ❌    |
| `script`                | ✅      | ✅    |

**微任务包含：**

| #                            | 浏览器 | Node |
| ---------------------------- | ------ | ---- |
| `process.nextTick`           | ❌      | ✅    |
| `MutationObserver`           | ✅      | ❌    |
| `Promise.then catch finally` | ✅      | ✅    |

**Note：**宿主环境提供的方法是宏任务，例如setTimeout、setInterval，这些都是浏览器或者Node环境实现的。js引擎自身提供的是微任务，例如Promise。基本上平时接触到的除了Promise都是宏任务。 

![异步任务处理流程示意图](https://i.loli.net/2021/10/04/iBMbC5aPVfSs4Qq.png)

#### 定时器

> 定时器功能主要由setTimeout()和setInterval()这两个函数来完成，它们的内部运行机制完全一样，区别在于前者指定的代码是一次性执行，后者则为反复执行。 

setTimeout()最短间隔在激活的选项卡中是 `4ms`，在未激活的选项卡中是 `1000ms`。

setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行。 

## 数据类型

### 数据类型种类

> [MDN: JavaScript 数据类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)

#### 基本类型

> 非对象、无方法的数据称为基本类型。

包含7种：[number](https://developer.mozilla.org/zh-CN/docs/Glossary/Number)，[bigint](https://developer.mozilla.org/zh-CN/docs/Glossary/BigInt)，[null](https://developer.mozilla.org/zh-CN/docs/Glossary/Null)，[undefined](https://developer.mozilla.org/zh-CN/docs/Glossary/undefined)，[string](https://developer.mozilla.org/zh-CN/docs/Glossary/String)，[boolean](https://developer.mozilla.org/zh-CN/docs/Glossary/Boolean)，[symbol](https://developer.mozilla.org/zh-CN/docs/Glossary/Symbol) 。

**1. number**

基于 `IEEE 754`  标准的双精度64位二进制格式的值 `-(2^53 -1) ~ 2^53 -1`，超出范围数值不再安全（失真）。 

**2. bigint**

表示任意精度的整数，可以超过数字的安全整数限制，**`Number.MAX_SAFE_INTEGER`** 常量表示在 JavaScript 中最大的安全整数（`2^53 -1`）。

BigInt是通过在整数末尾附加 `n `或调用构造函数来创建的。

```js
let x = 2n ** 53n
x // 9007199254740992n
typeof x // "bigint"
```

**3. null**

表示变量未指向任何对象，可以理解成尚未创建的对象。

当一个对象被赋值了 `null ` 以后，原来的对象在内存中就处于游离状态，`垃圾回收机制` 会择机回收该对象并释放内存。

`typeof null` 结果为 `"object"`，是因为 JavaScript 数据类型在底层都是以二进制的形式表示的，二进制的前三位为 0 会被 typeof 判断为对象类型，而 null 的二进制位恰好都是 0 ，因此，null 被误判断为 Object 类型；`Object.prototype.toString.call(null)` 结果为 `"[object Null]"`，Null 才是它的实际类型。

典型用法：

- 作为函数的参数，表示该函数的参数不是对象。

- 作为对象原型链的终点。

**4. undefined**

一个没有被赋值的变量会有个默认值 `undefined`，来源于预解析时的初始化，null 不等同于 undefined。

典型用法：

- 变量被声明了，但没有赋值时，就等于undefined。

- 调用函数时，应该提供的参数没有提供，该参数等于undefined。

- 对象没有赋值的属性，该属性的值为undefined。

- 函数没有返回值时，默认返回undefined。

**5. string**

表示文本数据。字符串的长度是它的元素的数量，中英文都是占一个字符。第一个元素的索引为 0。

**6. boolean**

只有两个值：`true` 和 `false`。

**7. symbol**

`Symbol()` 函数会返回 symbol 类型的值，该类型具有静态属性和静态方法。但作为构造函数来说它并不完整，因为它不支持语法： `new Symbol()`。

每个从`Symbol()`返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符，这是该数据类型仅有的目的。

```js
let key = Symbol()
this[key] = function() {...}
```

当一个 symbol 类型的值在属性赋值语句中被用作标识符时，该属性（像上述代码块中 symbol ）是匿名的；并且是不可枚举的。

symbol特性：

- 唯一性。
- 不可枚举，不会在循环结构 `for(... in ...)` 中作为成员出现。
- 匿名，不会出现在 `Object.getOwnPropertyNames()` 的返回数组里。

#### 基本类型的包装对象

除了 `null` 和 `undefined` 之外，所有基本类型都有其对应的包装对象，其本质是 `特殊的引用类型`，可通过 [`valueOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf) 方法返回基本类型值。

当 `基本类型` 需要调用一个该类型的 `对象方法` 时（基本类型是没有方法的），JavaScript 会自动将基本类型转换成包装对象，并且调用相应的方法。实际上发生了下面的过程：

1. 创建一个该 `基本类型` 的包装对象实例；

2. 在实例上调用该类型对象方法；

3. 销毁实例。

注意包装类型和基本类型的区别：

```js
true === new Boolean(true) // false

123 === new Number(123) // false

'ConardLi' === new String('ConardLi') // false
typeof 'ConardLi' // "string"
typeof new String('ConardLi') // "object"
typeof new String('ConardLi').valueOf() // "string"

Object.prototype.toString.call('ConardLi') // "[object String]"
Object.prototype.toString.call(new String('ConardLi')) // "[object String]"
```

#### 引用类型

> 指内存中的可以被 [标识符](https://developer.mozilla.org/zh-CN/docs/Glossary/Identifier) 引用的一块区域。

包含且不限于：Object，Array，Map，Set，Date，RegExp。

### 判断数据类型

> 数据类型本质->构造器->对象

**1.typeof（一元运算符）**

判断原始类型，返回 `基本类型` 或 `Object` 或 `function` 字符串。

 `typeof` 的返回值：

| 类型                                                         | 结果                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [Undefined](https://developer.mozilla.org/zh-CN/docs/Glossary/undefined) | `"undefined"`                                                |
| [Null](https://developer.mozilla.org/zh-CN/docs/Glossary/Null) | `"object"` (原因见[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof#null)) |
| [Boolean](https://developer.mozilla.org/zh-CN/docs/Glossary/Boolean) | `"boolean"`                                                  |
| [Number](https://developer.mozilla.org/zh-CN/docs/Glossary/Number) | `"number"`                                                   |
| [BigInt](https://developer.mozilla.org/zh-CN/docs/Glossary/BigInt) | `"bigint"`                                                   |
| [String](https://developer.mozilla.org/zh-CN/docs/Glossary/String) | `"string"`                                                   |
| [Symbol](https://developer.mozilla.org/zh-CN/docs/Glossary/Symbol) | `"symbol"`                                                   |
| 宿主对象（由 JS 环境提供）                                   | *取决于具体实现*                                             |
| [Function](https://developer.mozilla.org/zh-CN/docs/Glossary/Function) 对象 | `"function"`                                                 |
| 其他任何对象                                                 | `"object"`                                                   |

```js
typeof 1 // "number"
typeof(1) // "number"
typeof(() => 1) // "function"
typeof({a: 1}) // "object"
```

**2. valueOf ( ) **

返回指定对象的原始值，一般由js自动调用，用于判断类型时需搭配 `typeof` 使用。

不同类型对象的 `valueOf()` 方法的返回值：

| **对象** | **返回值**                                               |
| :------- | :------------------------------------------------------- |
| Array    | 返回数组对象本身。                                       |
| Boolean  | 布尔值。                                                 |
| Date     | 存储的时间是从 1970 年 1 月 1 日午夜开始计的毫秒数 UTC。 |
| Function | 函数本身。                                               |
| Number   | 数字值。                                                 |
| Object   | 对象本身。这是默认情况。                                 |
| String   | 字符串值。                                               |
|          | Math 和 Error 对象没有 valueOf 方法。                    |

```js
let str = new String('12345')

typeof(str) // "object"
str instanceof Object // true
str instanceof String // true
Object.prototype.toString.call(str) // "[object String]"

str.valueOf() // "12345"
typeof str.valueOf() // "string"
```

**3. Object.prototype.toString.call( ) **

判断最具体的数据类型，返回 "[object *type*]"，其中 `type` 是对象的类型。

```js
Object.prototype.toString.call(‘12345’) // [object String]
Object.prototype.toString.call(new Date) // [object Date]

'[object String]'.slice(8, -1) // "String"
```

**4. instanceof（关系运算符）**

检测  `constructor.prototype ` 是否存在于参数的原型链上，即某个实例对象是否是某个构造函数的实例，返回 `true` 和 `false`。

```js
let obj = new Object()
obj instanceof Object // true
```

**Note:** 在目前的ES规范中，只能读取对象的原型而不能改变它，但借助于非标准的 `__proto__` 伪属性，是可以实现的。比如执行 `obj.__proto__ = null` 之后，`obj instanceof Object` 就会返回 `false` 了。

**5. Array.isArray() 和 isNaN() **

```js
// 是不是数组
Array.isArray([1]) // true
// 是不是数值
isNaN('asd')
```

### 转换数据类型

> 分为：显式转换、隐式转换。在编译器自动赋值时,会发生*隐式转换*，但在代码中，也可以用一些写法强制要求进行*显式转换*。

#### 显式转换

>  显式转换指手动将各种类型的值，转换成数值、字符串、布尔值。 

**1. 转数值类型**

- 使用构造函数 `Number()`

```js
Number('a') // NaN

Number(true) // 1
Number(false) // 0

Number(null) // 0
Number(undefined) // NaN

// 参数是引用类型时，将返回`NaN`，除非是包含单个数值的数组
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([1]) // 1
```

 - 使用一元加法运算符

```js
'1.10' * 1 // 1.1
+'1.10' // 1.1

1+ +'2' // 3 (两个加号必须用用空格隔开 或 +'2'用小括号括起来)
1+(+'2')+3 // 6
1+(+'2.0')+3 // 6

1+(+'2')+3+'4' // '64'(表达式中字符串没先转数字时，其结果会是个字符串)
1+(+'2')+3+(+'4') // 10

// 参数是引用类型时，则会进行拼接成字符串
1 + {a: 1} // "1[object Object]"
1 + [1, 2, 3] // "11,2,3"
1 + [1] // "11"
```

 - 使用方法 `parseInt(string, radix)`

```js
// 只能返回整数，会丢失小数部分
parseInt('100', 10) // 100
parseInt('100', 2) // 4
```

**Note:** 在字符串以"0"为开始时旧的浏览器默认使用八进制基数；ECMAScript 5，默认的是十进制的基数。使用时最好带上进制(radix) 参数，用于指定使用哪一种进制。

 - 使用方法 `parseFloat(string)`

```js
// 会返回浮点数，不会丢失小数部分
parseFloat('100.10') // 100.1
parseFloat('100.01') // 100.01
```

**2. 转字符串类型**

- 使用构造函数 `String(x)`

```js
// 基本类型转换结果都是字符串
String(1) // "1"

String(true) // "true"
String(false) // "false"

String(null) // "null"
String(undefined) // "undefined"

// 参数是引用类型时
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
String([1]) // "1"
```

- 使用一元加法运算符

```js
// 都会 拼接 成字符串
'1' + 1 // "11"

'1' + true // "1true"
'1' + false // "1false"

'1' + null // "1null"
'1' + undefined // "1undefined"

'1' + {a: 1} // "1[object Object]"
'1' + [1, 2, 3] // "11,2,3"
'1' + [1] // "11"
```

- 使用方法 `string.toString()`

```js
let a = 1
a.toString() // "1"

true.toString() // "true"
```

**3. 转布尔类型**

- 使用构造函数 `Boolean()`

```js
// 除了以下五个值的转换结果为 false，其他的值全部为 true
Boolean(null)
Boolean(undefined)
Boolean(0) // +0 和 -0 转换结果也是false
Boolean(NaN)
Boolean('') // 空字符串
```

 - 使用逻辑非 (!) 运算符

```js
!0 // true
!1 // false

!true // false
!false // true

!null // true
!undefined // true

!{a: 1} // false
![] // false
```

#### 隐式转换

> 显式转换是隐式转换的基础。

遇到以下三种情况时，JavaScript编译器会自动转换数据类型：

```js
// 1. 不同类型的数据互相运算
123 + 'abc' // "123abc"

// 2. 对非布尔类型的数据求布尔值
if ('abc') { // true
  ...
}

// 3. 对非数字类型的数据使用一元运算符（即“+”和“-”）
+{a: 1} // NaN
+'1' // 1
```

**隐式转换规则：** 某个位置预期什么类型的值，就调用该类型的 `构造函数` 进行转换。比如，运算符两侧预期为字符串，就调用构造函数 `String()` 进行转换；如果该位置既可以是字符串，也可能是数字，那么默认转换为数字。

**1. 自动转数字类型**

除了 `一元加法` 运算符有可能把运算子转换为字符串，其他运算符都会把运算子转换成数值。

**2. 自动转字符串类型**

字符串的自动转换，主要发生在加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

**3. 自动转布尔类型**

比如 `if语句` 的条件部分预期布尔类型，编译器会自动调用构造函数 `Boolean()` 进行转换。

 除了以下五个值，其他都是自动转为`true`。

```js
// 除了以下五个值的转换结果为 false，其他的值全部为 true
Boolean(null)
Boolean(undefined)
Boolean(0) // +0 和 -0 转换结果也是false
Boolean(NaN)
Boolean('') // 空字符串
```

**Note:**

```
var obj={a:1,b:'2'}
console.dir('输出：'+obj)  // [object Object]

问：js中输出一个object对象显示的是[object Object]是什么意思？

网友答：object的prototype链中都没有实现自己的baitoString()的话, 把object转换为String时就会调用Object.prototype.toString, 输出的格式是[object 对象的类型]。

解：字符串拼接对象出现这种情况，实则就是数据类型转换问题。将 obj 对象利用 JSON.stringify() 转成字符串即可。
var obj={a:1,b:'2'}
console.dir('输出：'+JSON.stringify(obj))  // 输出：{"a":1,"b":"2"}
```

## 闭包和作用域

### 闭包

**什么是闭包**

在js中，通常函数执行完后在其内部定义的变量和函数（作用域）就会被清理，内存随之回收，但是由于闭包是建立在一个 `父函数` 内部的 `子函数`，如果 `子函数` 有访问 `父函数` 的作用域，则 `父函数` 执行完后其作用域不会被清理，导致 `子函数` 仍可以访问 `父函数` 的作用域，这就是 **闭包**。（大括号包裹起来形成 **代码块** 可形成闭包）

```js
function a() {
    let a = 1
    // 返回子函数
    return () => a
}
// 执行父函数a，得到返回的子函数，a执行完后其作用域仍在需供子函数使用
let b = a()
// 执行b，即子函数，访问上级作用域打印出1
console.log(b())
```

**解决了什么**

使用闭包主要是为了设计私有的方法和变量。闭包的优点是可以避免全局污染，缺点是闭包会常驻内存，会增大内存使用量，使用不当很容易造成内存泄露。

闭包有三个特性：

1. 函数嵌套函数
2. 子函数可以访问父函数能访问到的所有变量和函数，但父函数不能访问子函数内部的变量和函数。
3. 父函数执行完后其作用域只有在没被子函数访问时才会被垃圾回收机制回收。

**应用场景**

- 作用域链
- 路由懒加载
- 异步方法的回调函数
- 一个函数内部返回另一个函数
- ... ...

### 作用域

> 函数执行时所在的作用域，是 **定义时** 的作用域，而不是调用时所在的作用域。

**作用域**

作用域就是当前的 `执行上下文环境`，包含了当前代码块（即闭包，用大括号括起来的；小括号括起来，末尾再添加一个小括号，是自调用函数）定义的变量和函数。

es6之前只有全局作用域和函数作用域，之后增加了块作用域，这也是为什么在全局作用域代码块中let/const声明的变量无法在代码块外使用，但var声明的仍可使用。

```js
let a = 1;
{ // 代码块，拥有块作用域
  console.log(a); // 1
  var b = 2;
  let c = 3;
}
console.log(b); // 2
console.log(c); // Uncaught ReferenceError: c is not defined
foo();
function foo() { // 子函数，拥有函数作用域
  var d = 4;
};
console.log(d); // Uncaught ReferenceError: d is not defined
```

**作用域链**

作用域可以被多层嵌套。例如，函数A可以包含函数B，函数B可以再包含函数C。B和C都形成了闭包，所以B可以访问A，C可以访问B和A。形成了 `window全局作用域 => A作用域 => B作用域 => C作用域`，这个称之为 `作用域链`。 

访问原则：

- 当前作用域没有所需变量就到上层作用域找，直至全局作用域 `window对象`。
- 当前作用域无权访问下层作用域的变量和函数，只有调用方法的执行权限。

**作用域和函数堆栈**

|          | 函数作用域                                                   | 函数堆栈                                                     |
| :------: | :----------------------------------------------------------- | ------------------------------------------------------------ |
|   作用   | 可以访问代码块范围内的任何变量和函数。                       | 产生代码执行所需的环境，包含了被调用函数内部上下文环境及this指向。 |
| 产生过程 | 1、在预解析时产生一个全局作用域，其全局对象是window。<br />2、在代码执行时用var定义的变量、函数都会添加到这个作用域（全局）上；如果存在闭包，以及用let、const定义的，则添加到下级作用域（非全局）上，从而形成作用域链。 | 1、在预解析时把全局对象window压进栈（栈名Global），从而在栈底产生首个执行上下文环境。<br />2、在代码执行时的动作（非函数调用）都处于这个执行上下文环境中；如果存在函数调用，则把被调函数内部的上下文环境和继承的this压进栈，代码执行完时生成返回值并出栈。 |
| 访问方式 | 如果当前作用域没有所需值，便到上一层作用域中查找，直至找到全局作用域。 | 运行程序时先把全局环境（全局对象）window压进栈。当调用函数（跟访问对象不一样）时，把被调用的函数内部环境压进栈，执行完生成返回值出栈。 |

**Note:** 当前作用域 == 当前执行上下文 == 当前闭包

## 函数

### 创建函数的四种方法

**1. 函数声明**

> 使用 **function** 关键字声明函数。

```js
function named() {
	return '声明一个函数'
}
```

**2. 函数表达式（字面量）**

```js
let fun1 = function() {
    return '这是一个匿名函数'
}
let fun2 = function named() {
    return '这是一个命名函数'
}
```

`命名函数` 的函数名只在该函数体内部有效，函数体外部无效。这样做的好处有两个，一是可以在函数体内部调用自身，二是调试时观测堆栈将显示函数名而不是一个匿名。

**3. new Function()**

> `Function()` 是内置的构造函数，创建一个新的函数对象，但它只能在全局作用域中运行。

可以传递任意数量的参数给Function构造函数，但只有最后一个参数会被当做函数体，如果只有一个参数，该参数就是函数体。

```js
const sum = new Function('a', 'b', 'return a + b');  // 可以不要new
console.log(sum(2, 6));  // 8
```

**4. 自定义构造函数**

> 为什么要使用构造函数？因为可以批量创建函数。
>
> 为什么要使用this？因为每次调用构造函数都是新建一个对象，this用来指向新创建的对象 。

```js
function CreateFunction(name, age, alive, sayHi) {
	this.name = name
	this.age = age
	this.alive = alive
	this.sayHi = sayHi
}
let fun = new CreateFunction('wife', 20, true, function() {  // 必须要new
    return 'i love you'
})
```

### 函数参数

**arguments对象**

函数的所有实参会被保存在伪数组 `arguments` 对象中，它带有数组索引和length属性，但没有全部的Array对象的操作方法，如push、pop等。

**默认参数**

函数形参的默认值是 `undefined`。默认值可以手动设置，如下例所示。

```js
function multiply(a, b = 2) {
  return a * b
}
// es6以前的做法
// b = (typeof b !== 'undefined') ?  b : 2

multiply(5) // 10
```

**剩余参数**

[剩余参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/rest_parameters)语法允许将不确定数量的参数表示为数组。在下面的例子中，使用剩余参数收集从第二个到最后参数。然后，将这个数组的每一个数与第一个参数相乘。

```js
function multiply(multiplier, ...theArgs) {
  return theArgs.map(x => multiplier * x);
}

var arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6]
```

### 箭头函数

`箭头函数表达式` 的语法比 `函数表达式` 更简洁，但不能使用 `this`，`arguments`，`super` 和 `new.target`。箭头函数总是匿名的，它适用于那些本来需要匿名函数的地方，并且它不能用在构造函数中。

- 箭头函数捕捉闭包上下文的 `this` 值 ，即指向了父级作用域的上下文。

- 有两个因素会影响引入箭头函数：`更简洁的函数` 和 `this`。

普通函数跟箭头函数对比：

```js
var a = [
  "Hydrogen",
  "Helium",
  "Lithium",
  "Beryllium"
];

// 普通函数
var a2 = a.map(function(s){ return s.length })

// 箭头函数
var a3 = a.map( s => s.length )
```

### this关键字

> 所述`this`关键字的计算结果为当前执行上下文的ThisBinding的值，由其函数的调用上下文及其调用位置确定 （通常是上一级作用域）（箭头函数this指向定义时的对象）。 

**Tisps:** 使用 `this.xx` 访问的是this指向的那个对象，而使用 `xx` 访问的是作用域链。

#### this指向

> 由其函数的调用上下文及其调用位置确定 （通常是上一级作用域）。
>
> （函数作用域、箭头函数是在定义函数、对象时确定的）

**1. 全局环境**

无论是否在严格模式下，在全局执行环境中（在任何函数体外部）`this` 都指向全局对象 **window**。

```js
// 在浏览器中, window对象同时也是全局对象
console.log(this === window); // true

a = 37;
console.log(window.a); // 37

this.b = "MDN";
console.log(window.b)  // "MDN"
console.log(b)         // "MDN"
```

**2. 简单函数调用**

没有使用new，没有以对象的形式调用，就单纯的声明函数然后调用，此时this指向window对象。

**3. bind、apply、call方法**

这三个方法定义在 `Function.prototype`，它们都能改变this指向，此时this指向它们的第一个参数。（严格模式下函数是没有绑定到 this 上，这时候this是 undefined）

**4. 箭头函数**

this指向箭头函数被创建时的上下文环境。简单说就是根据外层作用域来决定this，继承外层函数绑定的this，且箭头函数绑定了父级作用域的上下文。如在全局环境下指向全局对象。

**5. 调用对象的方法**

在典型的面向对象的编程中，需要一种识别和引用当前正在使用的 `对象` 的方法，this指向该对象，使对象能够访问自身的属性。 

**6. 原型链中的this**

使用原型链上的方法时，指向调用该方法的对象。

**7. 作为构造函数**

每次调用构造函数都是新建一个对象，this用来指向新创建的对象 。

**8. 作为一个DOM事件处理函数**

使用DOM找到节点添加事件，this指向该节点对象。

**9. 作为一个内联事件处理函数**

在html节点上行内编写事件，调用JavaScript中的函数执行，this指向window对象。

**综上所述：**

-  在全局环境中，this指向全局对象window。
-  在简单调用中，this指向window对象。（严格模式下函数是没有绑定到 this 上，这时候this是 undefined）
-  在bind、apply、call方法中，this指向第一个参数。
-  在箭头函数中，this继承父级作用域的this。
-  在对象方法中，this指向调用它所在方法的对象。 
-  在原型链中，this指向调用该方法的对象。
-  在构造函数中，this指向新创建的对象。
-  在DOM事件中，this指向节点对象。
-  在行内事件中，this指向window对象。

**总结：**

> 如果要判断一个函数的 `this` 绑定，就需要找到这个函数的直接调用位置。然后可以顺序按照下面四条规则来判断 `this `的绑定对象：
>
> 1.  由 `new` 调用：绑定到新创建的对象
> 2.  由 `call`、 `apply`、 `bind` 调用：绑定到指定的对象
> 3.  由上下文环境的对象调用：绑定到上下文环境的对象（在哪调用便指向哪）
> 4.  全局执行环境中（在任何函数体外部）：全局对象window

**Note:**

```js
let num = 123
console.log(this)  // window
console.log(this.num);  // undefined
console.log(num);  // 123

问：为什么 this.num 是undefine？

答：这是 `执行上下文栈，this指向` 问题。js引擎先压window对象进栈产生全局作用域；接下来作用域扩张产生作用域链，当调用函数时会把被调函数内部的环境（上下文环境）压进栈。
let和const会产生 `块作用域`，其声明的变量会存放于 `当前上下文环境（非栈底，即非全局作用域）` 中，而 this 指向的是 window （栈底，全局作用域）故访问不到，即打印undefined；如果使用 var，则声明的变量存放于 window 中（var声明的变量只存在于函数作用域和全局作用域中），此时可以访问到。
```

#### 手动设置this指向

**1. bind()**

> 改变this，不自动调用函数，可接受列表和数组作为参数。

创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

一旦使用bind后this会被永久的绑定到第一个参数，无论之后链式使用多少次bind修改this。

**2. apply()**

> 改变this，自动调用函数，接受数组或伪数组作为参数。

在一个对象的上下文中应用另一个对象的方法；参数能够以数组或伪数组形式传入。

**3. call()**

> 改变this，自动调用函数，接受逗号隔开的参数。

在一个对象的上下文中应用另一个对象的方法；参数能够以列表形式传入。

**Note:** `call()` 的作用和 `apply()` 类似，区别就是 `call()` 接受的是 `参数列表`，而 `apply()` 接受的是`一个包含多个参数的数组` 。 

```js
let first_object = {
	num: 42
}
let second_object = {
	num: 24
}

function multiple(mult) {
	return this.num * mult
}

Function.prototype.bind = function(obj) {
	let method = this
	temp = function() {
		return method.apply(obj, arguments)
	}
	return temp
}

let first_multiply = multiple.bind(first_object)
first_multiply(5) // 42 * 5

let second_multiply = multiple.bind(second_object)
second_multiply(5) // 24 * 5
```

```js
<button type="button" id="thebutton">点击</button>

function BigComputer(answer) {
	this.the_answer = answer;
	this.ask_question = function() {
		alert(this.the_answer);
	}
}

function addhandler() {
	var deep_thought = new BigComputer(42)
	the_button = document.getElementById('thebutton')

	the_button.onclick = deep_thought.ask_question.call(deep_thought)
}

window.onload = addhandler; // 弹窗显示42
```

## 对象

> JavaScript中的对象分为三种类型：自定义对象，JS内置对象，浏览器对象（BOM+DOM）。

### new 运算符

>  构造函数 ，是一种特殊的函数。主要用来在创建对象时初始化对象， 即为对象成员变量赋初始值，总与new运算符一起使用在创建对象的语句中。
>
>  1. 构造函数用于创建对象，建议首字母要大写。
>  2. new要和构造函数要一起使用才有意义。构造函数是特殊（创建对象）的函数，如果不使用new自定义封装一个构造函数需要手动返回创建的实例化对象，而使用new则不需。

**new 在执行时会做五件事情：**

1. 在内存中创建一个新的空对象
2. 将新对象的原型指向构造函数
3. 让this指向这个新对象
4. 执行构造函数，给这个新对象添加属性和方法
5. 返回这个新对象（所以构造函数里面不需要return）

### 使用不同的方法来创建对象和生成原型链

> `构造函数` 通过 `prototype` 属性访问原型；
>
> `实例化对象` 通过 `__proto__`（非标准） 或 `Object.getPrototypeOf(obj)` 访问原型；
>
> `原型` 通过 `constructor` 访问构造函数。

**Note:** 在目前的ES规范中，只能读取对象的原型而不能改变它，但借助于非标准的 `__proto__` 伪属性，是可以实现的。比如执行 `obj.__proto__ = null` 之后，`obj instanceof Object` 就会返回 `false` 了。

**1. 字面量**

> let obj = { ... }  // 其中 obj 是变量，等号右侧的 { ... } 便是字面量。

```js
let obj = {
    name: 'wife',
    age: 20,
    alive: true,
    sayHi: function() {
        return 'i love you'
    }
}
// 原型链：obj ---> Object ---> null
```

**2. new Object()**

> `Object()` 是内置的构造函数，通过执行 `new` 运算符创建一个对象包装器，参数为对象（可选），创建后还可不断增加属性和方法。

```js
// 不带参数
let obj = new Object()  // 可以不要new
obj.name = 'wife'
obj.age = 20
obj.alive = true
obj.sayHi = function() {
    return 'i love you'
}
// 原型链：obj ---> Object ---> null
```

**3. 自定义构造函数**

> 为什么要使用构造函数？因为可以批量创建对象。
>
> 为什么要使用this？因为每次调用构造函数都是新建一个对象，this用来指向新创建的对象 。

```js
function CreateObject(name, age, alive, sayHi) {
    this.name = name
    this.age = age
    this.alive = alive
    this.sayHi = sayHi
}
let obj = new CreateObject('wife', 20, true, function() {  // 必须要new
    return 'i love you'
})
// 原型链：obj ---> CreateObject ---> Object ---> null
```

**4. Object.create()**

> es5:  `Object.create()` 方法创建一个新对象，新对象的原型就是调用 create 方法时传入的第一个参数。

```js
let a = {foo: 1}
// 原型链：a ---> Object ---> null

let b = Object.create(a)
// 原型链：b ---> a ---> Object ---> null
console.log(b.foo);  // 1 (继承a而来)
```

**5. 关键字Class**

> es6: 引入了一套新的关键字用来实现 [class](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)。使用基于类语言的开发人员会对这些结构感到熟悉，但它们是不同的，JavaScript 仍然基于原型。

```js
"use strict";

class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}

class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength);
  }
  get area() {
    return this.height * this.width;
  }
  set sideLength(newLength) {
    this.height = newLength;
    this.width = newLength;
  }
}

var square = new Square(2);
// ?原型链：square ---> Polygon ---> Object ---> Object ---> Object ---> null
```

### JS内置对象

> 指JavaScript自身预定义的对象，在ECMAScript标准中定义，由浏览器厂家具体实现，由于标准的统一故兼容性问题不大。

#### 全局

```js
// 编码URL
encodeURI() // 适用于：协议+端口+域名+路径
encodeURIComponent() // 适用于：参数

// 解码URL
decodeURI() // 适用于：协议+端口+域名+路径
decodeURIComponent() // 适用于：参数
```

#### [Math](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Numbers_and_dates#%E6%95%B0%E5%AD%A6%E5%AF%B9%E8%B1%A1%EF%BC%88Math%EF%BC%89)

跟其他对象相比，Math对象比较特殊，它不是构造函数，所以使用时不需要使用new。

```js
Math.PI // 圆周率
Math.floor() // 向下取整
Math.ceil() // 向上取整
Math.round() // 四舍五入
Math.abs() // 绝对值
Math.random() // 随机数 [0, 1)
Math.reduce() // 累加器
Math.toFixed() // 指定小数位数

// 返回两个数之间的随机整数
Math.floor(Math.random()*(max-min+1))+min
```

#### [Date](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date)

```js
const date = new Date()

date.getFullYear() // 获取当年
date.getMonth() // 获取当月（0—11）
date.getDate() // 获取当日
date.getDay() // 获取当周（0-6，星期天为第一天）
date.getHours() // 获取当前时
date.getMinutes() // 获取当前分
date.getSeconds() // 获取当前秒

date.getTime() // 获取 1970 年 1 月 1 日至今的毫秒数（UTC +0）
```

##### 产生时间戳

```js
new Date() // Thu Oct 07 2021 02:31:32 GMT+0800 (中国标准时间)

// 获取 1970 年 1 月 1 日至今的毫秒数
new Date().valueOf()
new Date().getTime() 
+new Date()
Date.now()
```

```js
// 倒计时
function conutDown(time) {
	let nowTime = +new Date()
	let inputTime = +new Date(time)
	let times = (inputTime - nowTime) / 1000

	let d = parseInt(times / 60 / 60 / 24)
	d = d < 10 ? '0' + d : d
	
	let h = parseInt(times / 60 / 60 % 24)
	h = h < 10 ? '0' + h : h
	
	let m = parseInt(times / 60 % 60)
	m = m < 10 ? '0' + m : m
	
	let s = parseInt(times % 60)
	s = s < 10 ? '0' + s : s

	return `${d}天${h}时${m}分${s}秒`
}
console.log(conutDown('2020-08-15 18:00:00'));
```

#### [Array](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)

```js
// 添加、删除数组元素
Array.push() // 在数组的末尾增加一个或多个元素，并返回数组的新长度。
Array.unshift() // 在数组的开头增加一个或多个元素，并返回数组的新长度。
Array.pop() // 删除数组的最后一个元素，并返回这个元素。
Array.shift() // 删除数组的第一个元素，并返回这个元素。
Array.splice() // 在任意的位置给数组添加或删除任意个元素。

// 数组排序
Array.reverse() // 颠倒数组中元素的排列顺序。
Array.sort() // 对数组元素进行排序。

// 查找元素
Array.indexOf() // 返回数组中第一个与指定值相等的元素的索引。
Array.lastIndexOf() // 返回数组中最后一个与指定值相等的元素的索引。
Array.includes() // 判断当前数组是否包含某指定的值。

// 数组转字符串
Array.toString() // 返回一个由所有数组元素组合而成的字符串。
Array.join() // 连接所有数组元素组成一个字符串。

// 拼接、截取数组
Array.slice() // 抽取当前数组中的一段元素组合成一个新数组。
Array.concat() // 返回一个由当前数组和其它若干个数组或者若干个非数组值组合而成的新数组。

// ES6新增迭代方法
Array.forEach() // 为数组中的每个元素执行一次回调函数。
Array.filter() // 将所有在过滤函数中返回 true 的数组元素放进一个新数组中并返回。
Array.map() // 返回一个由回调函数的返回值组成的新数组。
Array.reduce() // 从左到右为每个数组元素执行一次回调函数，并把上次回调函数的返回值放在一个暂存器中传给下次回调函数，并返回最后一次回调函数的返回值。
```

#### [String](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)

```js
// 查找字符——根据字符返回索引
String.indexOf() // 从字符串对象中返回首个被发现的给定值的索引值。
String.lastIndexOf() // 从字符串对象中返回最后一个被发现的给定值的索引值。

// 查找字符——根据索引返回字符
String.charAt() // 返回特定位置的字符。
String.charCodeAt() // 返回表示给定索引的字符的Unicode的值。
String.str[index] // 返回特定位置的字符。

// 编码转字符
String.fromCharCode() // 将 Unicode 编码转为一个字符

// 是否为开头、结尾
String.startWith() // 判断当前字符串是否以给定的字符开头
String.endsWith() // 判断当前字符串是否以给定的字符结尾

// 拼接、截取字符串
String.concat() // 连接两个字符串文本，并返回一个新的字符串。
String.substring() // 返回在字符串中指定两个下标之间的字符。
String.slice() // 摘取一个字符串区域，返回一个新的字符串。

// 大小写转换
String.toLowerCase() // 将符串中的字符转换成小写。
String.toUpperCase() // 将字符串转换成大写并返回。

// 替换字符串
String.replace() // 被用来在正则表达式和字符串直接比较，然后用新的子串来替换被匹配的子串。

// 字符串转数组
String.split() // 通过分离字符串成字串，将字符串对象分割成字符串数组。
```

#### [Number](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)

```js
Number.MAX_VALUE // 可表示的最大的数
Number.MIN_VALUE // 可表示的最小的数

Number.MIN_SAFE_INTEGER // 最小的安全的整型数字(-(2^53) - 1))
Number.MAX_SAFE_INTEGER // 最大的安全的整型数字(+(2^53) - 1))

Number.toFixed() // 四舍五入为指定小数位数的数字
Number.isInteger() // 判断给定的参数是否为整数
```

#### [RegExp](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp)

```js
RegExp.test() // 检测一个字符串是否匹配某个模式

// 支持正则表达式的 String 对象的方法
String.search() // 检索与正则表达式相匹配的值
String.match() // 找到一个或多个正则表达式的匹配
String.replace() // 替换与正则表达式匹配的子串
String.split() // 把字符串分割为字符串数组
```

### 浏览器对象

> 指JavaScript运行环境（即浏览器）提供的对象，由浏览器厂家自定义并实现，一些主要的对象已被大部分浏览器兼容。
>
> 浏览器对象分为两类：DOM（文档对象模型）、BOM（浏览器对象模型）。

#### BOM

[BOM](https://www.runoob.com/jsref/obj-window.html)

#### DOM

> DOM（Document Object Model——文档对象模型）是用来呈现以及与任意 HTML 或 XML文档交互的API。DOM 是载入到浏览器中的文档模型，以节点树的形式来表现文档，每个节点代表文档的构成部分（例如:页面元素、字符串或注释等等）。 
>
> DOM 接口中包含许多接口，包含Document、Element、Node、Event等等。

##### ParentNode

> 混合了所有(拥有子元素的) [`Node`](https://developer.mozilla.org/zh-CN/docs/Web/API/Node) 对象接口包含的共有方法和属性。 
>
> 它在 [`Element`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element)、[`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document) 和 [`DocumentFragment`](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment) 对象接口上被实现。

```js
/* 属性 */
ParentNode.childElementCount // 返回父节点的子元素数
ParentNode.children // 返回父节点的子元素列表
ParentNode.firstElementChild // 返回父节点的第一个 Element 后代，没有时返回 null
ParentNode.lastElementChild // 返回父节点的最后一个子元素，如果没有子元素，则返回 null

/* 方法 */
ParentNode.append() // 在 ParentNode的最后一个子节点之后插入一组 Node 对象或 DOMString 对象。被插入的 DOMString 对象等价为 Text 节点
ParentNode.prepend() // 在父节点的第一个子节点之前插入一系列Node对象或者DOMString对象。DOMString会被当作Text节点对待（也就是说插入的不是HTML代码）
ParentNode.querySelector() // 返回以当前元素为根元素，匹配给定选择器的第一个元素 Element
ParentNode.querySelectorAll() // 返回一个节点列表，表示以当前元素为根元素的匹配给定选择器组的元素列表
```

##### Document

> 可理解为 Document 代表 html 节点。
>
> Document 接口继承自 [ParentNode](https://developer.mozilla.org/zh-CN/docs/Web/API/ParentNode)   [Node](https://developer.mozilla.org/zh-CN/docs/Web/API/Node)   [EventTarget](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget) 的接口。

```js
/* 属性 */
// 返回列表的api
Document.all // 返回文档中所有元素节点的集合
Document.all[1]
Document.all['id'] // id选择器所在节点也会被返回，其他选择器不会
Document.scripts // 返回文档中所有的 <script> 元素(浏览器会自动添加4个)
Document.forms // 返回一个包含当前文档中所有表单元素 <form> 的列表
Document.images // 返回当前文档中所包含的图片的列表
Document.links // 返回一个包含文档中所有超链接的列表
Document.anchors // 返回当前文档中的所有锚点元素

// 返回非列表的api
Document.documentElement // 返回文档的根元素<html>元素
Document.head // 返回当前文档中第一个<head>元素
Document.body // 返回当前文档中的<body>元素或者<frameset>元素
Document.characterSet // 返回当前文档正在使用的字符编码（用户可重写编码方式）
Document.documentURI // 以字符串的类型，返回当前文档的路径
Document.mozSyntheticDocument // 文档是否是合成文档，即表示独立的图像，视频，音频等的文档

// 文档可见性
Document.hidden // 返回一个布尔值，表明当前页面用户是否可见（应用场景：当用户停留在当前页面时播放视频，离开当前页面时停止播放）
Document.visibilityState // 返回文档的可见性，返回结果如下：
    // 'visible' : 此时页面内容至少是部分可见. 即此页面在前景标签页中，并且窗口没有最小化.
    // 'hidden' : 此时页面对用户不可见. 即文档处于背景标签页或者窗口处于最小化状态，或者操作系统正处于 '锁屏状态' .
    // 'prerender' : 页面此时正在渲染中, 因此是不可见的 (对于document.hidden会被认为是隐藏的). 文档只能从此状态开始，永远不能从其他值变为此状态.注意: 浏览器支持是可选的.
    // 当此属性的值改变时, 会递交 visibilitychange 事件给Document.
    // 典型用法是防止当页面正在渲染时加载资源, 或者当页面在背景中或窗口最小化时禁止某些活动.

/* ParentNode接口（见上节） */
```

##### Element

```js
Element.appendChild() // 向元素添加新的子节点，作为最后一个子节点（只能添加一个节点）
```

[更多DOM API](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)

## 对象拷贝方法

```js
// 原始对象
Object.prototype.a = '1'
Object.prototype.aa = () => 'a1'
let obj = {
    name: 'wife',
    age: 20,
    alive: true,
    adress: { city: 'xg' },
    sayHi: function() { return 'i love you' }
}
obj
```

### 原始方式

>  复制对象的原始方法是循环遍历原始对象，然后一个接一个地复制每个属性。该方法属于浅拷贝。 

```js
// 得出缺陷1、2、3、4
function copy(mainObj) {
    let objCopy = {}  // 存储 obj 对象的副本
    let key
    for(key in mainObj) {
        objCopy[key] = mainObj[key]  // 复制每个属性到objCopy
    }
    return objCopy
}
copy(obj)
```

**缺陷：**

-  `objCopy` 对象具有一个新的 `Object.prototype`方法，这与 `mainObj` 对象的原型方法不同，这不是我们想要的。我们需要精确的拷贝原始对象。 
-  属性描述符不能被复制。值为 `false` 的 “可写(`writable`)” 描述符在 `objCopy` 对象中为 `true` 。 
-  只复制了 `mainObj` 的可枚举属性。 

-  如果原始对象存在一个值为对象的属性，那么该属性被拷贝的是引用地址，该 **属性中的属性** 是被 **原始对象** 和 **副本对象** 共同使用的。即修改**属性中的属性**时 **原始对象** 和 **副本对象** 都将产生变化；而修改 **属性** 改动的是引用地址， **原始对象** 和 **副本对象** 只会单方面变化。

### 浅拷贝对象

>   当拷贝源对象的顶级属性被复制而没有任何引用，并且拷贝源对象存在一个值为对象的属性，被复制为一个引用时，那么我说这个对象被浅拷贝。如果拷贝源对象的属性值是对象的引用，则只将该引用值复制到目标对象。 
>
>   浅层复制将复制顶级属性，但是嵌套对象将在原始（源）对象和副本（目标）对象之间是共享。 

```js
// 得出缺陷1
// Object.assign() 方法用于将从一个或多个原始对象中的所有可枚举的属性值复制到目标对象。
let objCopy = Object.assign({}, obj)
objCopy
```

```js
// 得出缺陷1
// 扩展运算符
let objCopy = { ...obj }
objCopy
```

```js
// 得出缺陷1、2
// 这里新创几个对象验证
// Object.create()：创建新对象，第一个参数为新对象的__proto__，第二个参数(可选)则是要添加到新对象的不可枚举（默认）属性（即其自身定义的属性，而不是其原型链上的枚举属性）对象的属性描述符以及相应的属性名称。
// enumerable：可枚举(enumerable) 属性描述符，true为可枚举。（可枚举的会被复制）
let someObj = {
  a: 2,
}

let obj = Object.create(someObj, { 
  b: {
    value: 2,  
  },
  c: {
    value: 3,
    enumerable: true,  
  },
});
 
let objCopy = Object.assign({}, obj);  // 进行浅拷贝
objCopy // { c: 3 }
```

**缺陷：**

- 如果原始对象存在一个值为对象的属性，那么该属性被拷贝的是引用地址，该 **属性中的属性** 是被 **原始对象** 和 **副本对象** 共同使用的。即修改**属性中的属性**时 **原始对象** 和 **副本对象** 都将产生变化；而修改 **属性** 改动的是引用地址， **原始对象** 和 **副本对象** 只会单方面变化。
- 原型链上的属性和不可枚举的属性并没有被复制。 

### 深拷贝对象

>  深度拷贝将拷贝遇到的每个对象。原始对象和副本对象不会共享任何东西，所以它将是原始对象的副本。

```js
// 当对象没有二级属性，即没有 属性的属性 时此方法变相的为深拷贝
let objCopy = Object.assign({}, obj)
objCopy
```

```js
// 得出缺陷1
let objCopy = JSON.parse( JSON.stringify(obj) )
objCopy
```

```js
// 完美深拷贝 - 对象及原型链上的属性、方法都被深拷贝
function deepClone(obj) {
  // 对常见的“非”值，直接返回原来值
  if ([null, undefined, NaN, false].includes(obj)) return obj;
  if (typeof obj !== "object" && typeof obj !== 'function') {
    //原始类型直接返回
    return obj;
  }
  var o = Array.isArray(obj) ? [] : {};
  for (let i in obj) {
    if (obj.hasOwnProperty(i)) {
      o[i] = typeof obj[i] === "object" ? deepClone(obj[i]) : obj[i];
    }
  }
  return o;
}
deepClone(obj)
```

**缺陷：**

-  此方法不能用于复制用户定义的对象方法。 即 **obj.sayHi** 并没有被复制。

## 本地存储方式

> 浏览器提供了四种方式：localStorage、sessionStorage、cookier、IndexedDB。

**1. 区别**

1. 传递方式不同

   cookie在每次请求都会发送回服务器，即使不需要；

   localStorage和sessionStorage不会自动把数据发给服务器，仅在本地保存使用。

2. 数据大小不同

   cookie数据不能超过4k；

   localStorage和sessionStorage数据根据浏览器不同存储大小也不同，2.5MB~10MB。

3. 数据有效期不同

   cookie默认标签页关闭时删除，设置过期时间后则在到期前有效，即使标签页或浏览器关闭；

   localStorage始终有效，即使窗口或浏览器关闭也一直保存，因此用作持久保存数据；

   sessionStorage仅在当前浏览器窗口关闭前有效，用于临时会话时保存数据。

4. 作用域不同

   cookie、localStorage 在所有同源窗口中都是共享的；

   sessionStorage仅作用于当前标签页。

共同点：都保存在浏览器端。

**2. 使用**

- cookier

```js
// 读取cookie
document.cookie

// 创建cookie - 值/过期时间/路径
document.cookie = "username=John Doe; expires=Thu, 18 Dec 2043 12:00:00 GMT; path=/";

// 删除cookie - 过期时间设为过去
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT";
```

- localStorage

localStorage 只支持 string 类型的存储，其他类型会自动转成字符串；如果要存储json，则在保存时需用  `JSON.stringify(value)`，使用时 `JSON.parse(value)`。

```js
// 保存
localStorage.setItem("key", "value");
localStorage["key"] = 1;
localStorage.key = 1;

// 读取
localStorage.getItem("key");
localStorage["key"];
localStorage.key;

// 删除
localStorage.removeItem("key");

// 清空
localStorage.clear();
```

- sessionStorage

_blank 方式新开窗口，sessionStorage 存储的数据会丢失，需设置 `rel` 属性，`<a href="***" target="_blank" rel="opener noopener nofollow ugc">打开新标签页</a>`。

```js
// 保存
sessionStorage.setItem("key", "value");
sessionStorage["key"] = 1;
sessionStorage.key = 1;

// 读取
sessionStorage.getItem("key");
sessionStorage["key"];
sessionStorage.key;

// 删除
sessionStorage.removeItem("key");

// 清空
sessionStorage.clear();
```

- IndexedDB

> 浏览器提供的本地数据库，它可以被网页脚本创建和操作。

优势：允许储存大量数据，提供查找接口，还能建立索引。

[基本操作](https://www.ruanyifeng.com/blog/2018/07/indexeddb.html)

## 循环迭代

> 把一个动作重复很多次（实际上重复的次数有可能为 0）。各种循环机制提供了不同的方法去确定循环的开始和结束。不同情况下，某一种类型循环会比其它的循环用起来更简单。

- **for** 语句

```js
for (let i = 0; i < arr.length; i++) {
    ...
}
```

- **while** 语句

```js
let i = 0
while (i < 10){
    i++
}
console.log(i);
```

- **do...while** 语句

```js
let i = 0
do {
    i++
} while (i > 5)
console.log(i);
```

- **label** 语句

> **标记语句**只能和 [`break`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/break) 或 [`continue`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/continue) 语句一起使用。标记就是在一条语句前面加个可以引用的标识符，从而清晰的表示接下来执行哪里的代码。
>
> 标记语句非常罕见，通常情况下，可以使用函数调用而不是（基于标记的）循环跳转。

```js
let str = ''

loop1:
for (let i = 0; i < 5; i++) {
  if (i === 1) {
    continue loop1
  }
  str = str + i
}

console.log(str);
// expected output: "0234"
```

- **break** 语句

> 用于终止循环，或者是链接到 **label** 语句。
>
> 注：**return** 语句是终止函数的执行，并返回一个指定的值给函数调用者。

```js
for (let i = 0; i < 3; i++) {
    if (i == 1) break
    console.log(i);
}
// 0
```

`js` 没有类似 `goto` 语句可以跳转到任意地方的语句，所以 `break` 语句必须内嵌在它引用的标签中。

```js
// break 内嵌在 loop2 中，loop2 内嵌在 loop1 中，这样是可以的
// break loop1：满足条件时会终止外层循环
// break loop2：满足条件时会终止内层循环
let i = 0, j = 0
loop1:while (i < 3){
    loop2:while (j < 3){
        if (i === 1 && j === 1) break loop1
        console.log(i,j)
        j++
    }
    i++
    j = 0
}
```

代码块外层不是循环或选择结构时，即为顺序结构时使用 `break` 必须附带使用 `label` 语句：

```js
// 跳转的外层不是循环或选择结构
loop: {
    console.log('1');
    break loop
    console.log(':-(');
}
```

```js
// 跳转的外层是循环结构
let i = 0
while (i<3){
    {
        console.log(i);
        break
        console.log(':-(');
    }
    i++
}
```

- **continue** 语句

> 用于跳过代码块的剩余部分并进入下一次循环，或者是链接到 **label** 语句。

```js
for (let i = 0; i < 3; i++) {
    if (i == 1) continue
    console.log(i);
}
// 0
// 2
```

- **for...in** 语句

> 利用关键字 **in** 以任意顺序遍历一个对象的除[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)以外的[可枚举](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)属性。

```js
// 用来迭代对象的属性名（包括它的原型链上的可枚举属性）
// 不建议用来遍历数组，其key会是索引
for (let key in obj) {
    ...
}
```

- **for...of** 语句

> [可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols) 包括 [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)，[`Map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)，[`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)，[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String)，[`TypedArray`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)，[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments) 对象等等。

```js
// 用来遍历可迭代对象的属性值（不包括它的原型链上的可枚举属性）
// 遍历obj会报错obj不可迭代
for (let value of arr) {
    ...
}
```

- **Note:** **for...in** 与 **for...of** 的区别

> 两者都是迭代一些东西，区别在于迭代方式不同:
>
> 前者以任意顺序迭代对象的可枚举属性（包含原型链）;
>
> 后者遍历可迭代对象定义要迭代的值（不包含原型链）。

```js
// 给构造函数 Object 和 Array 分别增加一个自定义属性
Object.prototype.objCustom = function() {}
Array.prototype.arrCustom = function() {}

// 构造函数Object实例化，其原型链上有 objCustom 属性
let obj = {
    name: 'wife',
    age: 18,
    alive: true,
    address: {
        city: 'xg'
    },
    say: () => 'i love you'
}
// 构造函数Array实例化，其原型链上 arrCustom 和 objCustom 属性
// 因为 Array 是 Object 的实例化继承到了 objCustom 属性
let arr = [17, 18, 19, 20]
arr.foo = 21
arr.bar = obj

console.log(arr);
// 迭代arr的所有可枚举属性（包含原型链上的）
for (let key in arr) {
    // 当前的枚举属性是自身的（不是继承的，即不是原型链上的）返回true
    if (arr.hasOwnProperty(key)) {  // 该方法继承于Object
        console.log(arr[key]);
    }
}
//
for (let key of arr) {
    console.log(key);  // 17 18 19 20
}
```

## 路径

**1. 什么是相对路径和绝对路径**

- 相对路径

以当前文件所在位置为参考基准的目录路径。当保存于不同目录的文件引用同一个文件时，所使用的路径将不相同。

- 绝对路径

以当前项目根目录为参考基准的目录路径。对项目中所有文件而言，根目录这个参考点对所有文件都是一样的。

**2. 路径特殊符号**

以下为建立路径所使用的几个特殊符号，及其所代表的意义。

```text
// 相对路径
" " -- 代表目前所在的目录。 如：<img src="abc.png" />
"./" -- 代表目前所在的目录。 如：<img src="./abc.png" />
"../" -- 代表上一层目录。 如：<img src="../abc.png" />

// 绝对路径
"/" -- 代表根目录。 如：<img src="/abc.png" />
"D:/abc.png" -- 代表D盘根目录。
```

**3. 使用场景**

高耦合时（功能模块）文件需要放在同目录（非根目录）下使用 `相对路径`，低内聚时（引入公众文件）使用 `绝对路径`，这样发生路由变更时不需要做过多的改动。

## 术语解释

### 回调、闭包、上下文

|  名词  | 解释                                                   |
| :----: | ------------------------------------------------------ |
|  回调  | 把函数当作值传入到另一个函数里面执行                   |
|  闭包  | 把一个函数作为值返回到外部执行                         |
| 上下文 | 即 运行环境（context），代码运行时所定义的变量和函数等 |

### 阻塞非阻塞与同步异步

老张爱喝茶，废话不说，煮开水。
出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）。
1 老张把水壶放到火上，立等水开。（同步阻塞）
老张觉得自己有点傻
2 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有。（同步非阻塞）
老张还是觉得自己有点傻，于是变高端了，买了把会响笛的那种水壶。水开之后，能大声发出嘀~~~~的噪音。
3 老张把响水壶放到火上，立等水开。（异步阻塞）
老张觉得这样傻等意义不大
4 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了，响了再去拿壶。（异步非阻塞）
老张觉得自己聪明了。



所谓同步异步，只是对于水壶而言。
普通水壶，同步；响水壶，异步。
虽然都能干活，但响水壶可以在自己完工之后，提示老张水开了。这是普通水壶所不能及的。
同步只能让调用者去轮询自己（情况2中），造成老张效率的低下。

所谓阻塞非阻塞，仅仅对于老张而言。
立等的老张，阻塞；看电视的老张，非阻塞。
情况1和情况3中老张就是阻塞的，媳妇喊他都不知道。虽然3中响水壶是异步的，可对于立等的老张没有太大的意义。所以一般异步是配合非阻塞使用的，这样才能发挥异步的效用。

|  名词  | 解释                                                         |
| :----: | :----------------------------------------------------------- |
|  同步  | 在发出一个功能调用后，在该调用没有得到结果之前，不会return返回。 |
|  异步  | 当一个异步功能调用发出后，调用者不能立刻得到结果；当该异步功能完成后，通过状态、通知（nodeJS中的res.send()）或回调来通知调用者。 |
|  阻塞  | 主线程会在调用时进入阻塞态，直到结果返回，再继续运行，相当于需要整个操作全部结束了，调用才会返回。 |
| 非阻塞 | 进程不用等待函数返回，可以做做别的事情。（如果接下来马上需要返回的结果，那么与阻塞并无区别） |

**Note:** 

进程的五大状态：创建、就绪、运行、阻塞、终止。

同步异步针对 *被调用者* 而言描述调用的特性；

阻塞非阻塞针对 *调用者* 而言，描述进程状态。

### 可枚举属性

可枚举属性是指那些内部 “可枚举” 标志设置为 `true` 的属性，对于通过直接的赋值和属性初始化的属性，该标识值默认为即为 `true`，对于通过 [Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 等定义的属性，该标识值默认为 `false`。

```js
const object1 = {};

Object.defineProperty(object1, 'property1', {
  value: 42,
  writable: false
});

object1.property1 = 77;
// throws an error in strict mode

console.log(object1.property1);
// expected output: 42
```

**判断指定属性：**

> 属性的所有权是通过判断该属性是否直接属于某个对象决定的，而不是通过原型链继承。
>
> `Object.hasOwnProperty()` 方法会返回一个布尔值，指示对象自身属性中是否具有指定的属性，会忽略掉那些从原型链上继承到的属性；
>
> `in` 运算符则会返回一个布尔值，指示对象自身属性和原型链上是否具有指定的属性。

- 对于自身对象

`Object.hasOwnProperty()` 判断指定属性是否存在后使用 `Object.propertyIsEnumerable()` 查询属性是否可枚举。

- 对于自身对象及其原型链

`in` 判断指定属性是否存在后使用 `Object.propertyIsEnumerable()` 查询属性是否可枚举。

**访问所有属性：**

> `Object.getOwnPropertyNames()` 方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括Symbol值作为名称的属性）组成的数组。
>
> `Object.getOwnPropertySymbols()` 方法返回一个给定对象自身的所有 Symbol 属性的数组。

- 对于自身对象

`Object.getOwnPropertyNames()` 和 `Object.getOwnPropertySymbols()` 获取到包含自身所有属性名的数组后使用 `Object.propertyIsEnumerable()` 查询属性是否可枚举。

- 对于自身对象及其原型链

需要额外代码实现。

**迭代：**

> `Object.keys()` 方法会返回一个由一个给定对象的自身可枚举属性组成的数组，数组中属性名的排列顺序和正常循环遍历该对象时返回的顺序一致 。

- 对于自身对象（可枚举属性）

`Object.keys()` 可直接迭代自身的可枚举属性。

- 对于自身对象（不可枚举属性）

`Object.getOwnPropertyNames()` 和 `Object.getOwnPropertySymbols()` 获取到包含自身所有属性名的数组后使用 `Object.propertyIsEnumerable()` 查询属性是否可枚举。

- 对于自身对象及其原型链（可枚举属性）

`for..in` （同时会排除 Symbol）

- 对于自身对象及其原型链（不可枚举属性）

需要额外代码实现。

**[统计表](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties#统计表)**

|                    | `in` | `for..in` | `obj.hasOwnProperty` | `obj.propertyIsEnumerable` | `Object.keys` | `Object.getOwnPropertyNames` |
| :----------------- | :--- | :-------- | :------------------- | :------------------------- | :------------ | :--------------------------- |
| 自身的可枚举属性   | true | true      | true                 | true                       | true          | true                         |
| 自身的不可枚举属性 | true | false     | true                 | false                      | false         | true                         |
| 自身的Symbol 键    | true | false     | true                 | true                       | false         | false                        |
| 继承的可枚举属性   | true | true      | false                | false                      | false         | false                        |
| 继承的不可枚举属性 | true | false     | false                | false                      | false         | false                        |
| 继承的 Symbol 键   | true | false     | false                | false                      | false         | false                        |

### 对象的属性

> ECMAScript 定义的对象中有两种属性：数据属性和访问器属性。

**1. 数据属性**

数据属性是键值对，并且每个数据属性拥有下列特性:

| 特性             | 数据类型           | 描述                                                         | 默认值    |
| :--------------- | :----------------- | :----------------------------------------------------------- | :-------- |
| [[Value]]        | 任何Javascript类型 | 包含这个属性的数据值。                                       | undefined |
| [[Writable]]     | Boolean            | 如果该值为 `false，`则该属性的 [[Value]] 特性 不能被改变。   | false     |
| [[Enumerable]]   | Boolean            | 如果该值为 `true，`则该属性可以用 [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) 循环来枚举。 | false     |
| [[Configurable]] | Boolean            | 如果该值为 `false，`则该属性不能被删除，并且 除了 [[Value]] 和 [[Writable]] 以外的特性都不能被改变。 | false     |

**2. 访问器属性**

访问器属性有一个或两个访问器函数 (get 和 set) 来存取数值，并且有以下特性:

| 特性             | 类型                   | 描述                                                         | 默认值    |
| :--------------- | :--------------------- | :----------------------------------------------------------- | :-------- |
| [[Get]]          | 函数对象或者 undefined | 该函数使用一个空的参数列表，能够在有权访问的情况下读取属性值。另见 `get。` | undefined |
| [[Set]]          | 函数对象或者 undefined | 该函数有一个参数，用来写入属性值，另见 `set。`               | undefined |
| [[Enumerable]]   | Boolean                | 如果该值为 `true，则该属性可以用` [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) 循环来枚举。 | false     |
| [[Configurable]] | Boolean                | 如果该值为 `false，则该属性不能被删除，并且不能被转变成一个数据属性。` | false     |

