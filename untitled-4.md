# 数据类型+判断方式+转换规则

## 数据类型

> MDN: [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data\_structures](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)

### 基本类型

> number、bigint、string、boolean、symbol、null、undefined

1. Number
   * JavaScript 中只有一种数字类型： 基于 IEEE 754 标准的双精度 64 位二进制格式的值（-\(2^53 -1\) 到 2^53 -1），超出范围数值不再安全。
   * 基本数值 Number\(''\)
     * 数值对象 new Number\(''\)
     * 如果参数无法被转换为数字，则返回 NaN。
     * 在非构造器上下文中 \(如：没有 new 操作符\)，Number 能被用来执行类型转换。
2. BigInt

   > [https://segmentfault.com/a/1190000019912017?utm\_source=tag-newest](https://segmentfault.com/a/1190000019912017?utm_source=tag-newest)

   * JavaScript 中的一个基础的数值类型， 可以用任意精度表示整数， 可以安全地存储和操作大整数，甚至可以超过数字的安全整数限制。 
   * BigInt是通过在整数末尾附加 `n`或调用构造函数来创建的。 
   * `BigInt`不能与数字互换操作。否则，将抛出`TypeError`。 

3. String
   * 用于表示文本数据。它是一组16位的无符号整数值的“元素”。 
   * 字符串字面量 \(通过单引号或双引号定义\) 和 直接调用 String 方法\(没有通过 new 生成字符串对象实例\)的字符串都是基本字符串。
     * 当基本字符串需要调用一个字符串对象才有的方法或者查询值的时候\(基本字符串是没有这些方法的\)，

        JavaScript 会自动将基本字符串转化为字符串对象并且调用相应的方法或者执行查询。
4. Boolean
   * 布尔表示一个逻辑实体，可以有两个值：`true` 和 `false`。 
5. Symbol
   * 符号类型 [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)
   * 不支持语法："new Symbol\(\)"
     * 每个从`Symbol()`返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。 
6. Null
   * null 是表示缺少的标识，指示变量未指向任何对象。把 null 作为尚未创建的对象，也许更好理解。
   * null 表示一个变量指向为空对象，而不是原始状态。

     * 当一个对象被赋值了null 以后，原来的对象在内存中就处于游离状态，GC 会择机回收该对象并释放内存,这是人为操作的结果。
     * null 有属于自己的类型 Null，而不属于Object类型，typeof 之所以会判定为 Object 类型，

        是因为JavaScript 数据类型在底层都是以二进制的形式表示的，二进制的前三位为 0 会被 typeof 判断为对象类型，而 null 的二进制位恰好都是 0 ，

        因此，null 被误判断为 Object 类型。

     * 当检测 `null` 或 `undefined` 时，注意[相等（==）与 严格相等 （===）两个操作符的区别](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators) ，前者会执行类型转换 

     > 上述写的什么鬼，重新记录
     >
     > [https://www.ruanyifeng.com/blog/2014/03/undefined-vs-null.html](https://www.ruanyifeng.com/blog/2014/03/undefined-vs-null.html)

   * **null表示"没有对象"，即该处不应该有值。**

     ```javascript
     > Number(null)
     0
     > 5 + null
     5
     ```
7. Undefined
   * 一个没有被赋值的变量会有个默认值 `undefined`

     * `undefined`是_全局对象_的一个属性。也就是说，它是全局作用域的一个变量。 
     * null不等同于undefined。
     * undefined 表示一个变量自然的、最原始的状态值。

     > 上述写的什么鬼，重新记录
     >
     > [https://www.ruanyifeng.com/blog/2014/03/undefined-vs-null.html](https://www.ruanyifeng.com/blog/2014/03/undefined-vs-null.html)

   * **undefined表示"缺少值"，就是此处应该有一个值，但是还没有定义。**

     ```javascript
     > Number(undefined)
     NaN
     > 5 + undefined
     NaN
     ```

### 对象类型

> 在计算机科学中, 对象是指内存中的可以被 `标识符` 引用的一块区域。
>
> 对象 =&gt; \(函数、数组、时间、正则\)

### 包装类型

为了便于操作基本类型值，`ECMAScript`还提供了几个特殊的引用类型，他们是基本类型的包装类型：

* `Boolean`
* `Number`
* `String`

注意包装类型和原始类型的区别：

```javascript
true === new Boolean(true); // false
123 === new Number(123); // false
'ConardLi' === new String('ConardLi'); // false
console.log(typeof new String('ConardLi')); // object
console.log(typeof 'ConardLi'); // string
```

## 判断数据类型

> 数据类型本质-&gt;构造器-&gt;对象

* typeof：（一元运算符）判断原始类型，返回 **基本类型** 或 **Object**。
* instanceof：（关系运算符）判断该对象是否是某个构造函数（数据类型）的实例。
* Object.prototype.toString.call\( \) ：判断 **Object** 具体的数据类型。
* valueOf \( \) ：返回对象的原始值。
* Array.isArray\(\)

## 转换数据类型

> 强制转换、自动转换

### 强制转换\(通过构造函数转换\)

> 强制转换主要指使用`Number`、`String`和`Boolean`三个构造函数，手动将各种类型的值，转换成数字、字符串或者布尔值。

1. Number\(\)

   1.1. 基本类型的转换规则

   * 构造函数 `Number()`

     ```javascript
     Number('a') -> NaN
     Number(null) -> 0
     Number(undefined) -> NaN
     ```

     * 使用一元加法运算符

       ```javascript
       '1.10' * 1 -> 1.1
       +'1.1' -> 1.1
       1+ +'2' -> 3(两个加号用空格隔开或小括号括起来)
       1+ +'2'+3 -> 6
       1+ +'2'+3+'4' -> '64'(有字符串时会被转成字符串)
       1+ +'2'+3+ +'4' -> 10
       1+ +'2'+3+'4'+'5' -> '645'
       1+ +'2'+3+'4'+ +'5' -> '645'
       ```

     * 方法 `parseInt(string, radix)` 只能返回整数，会丢失小数部分。
     * 方法 `parseFloat(string)` 会返回浮点数。

     1.2. 对象的转换规则

   * `Numbe()`方法的参数是对象时，将返回`NaN`，除非是包含单个数值的数组。

     ```javascript
     Number({a: 1}) -> NaN
     Number([1, 2, 3]) -> NaN
     Number([5]) -> 5
     ```

2. String\(\)
   * 构造函数 `String(x)`
     * 基本类型转换的都是字符串
     * 参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。 
     * 方法 `x.toString()`
     * +拼接，' '+x
3. Boolean\(\)
   * 构造函数 `Boolean()` 显式

     除了以下六个值的转换结果为`false`，其他的值全部为`true`。

     * `undefined`
     * `null`
     * `-0` `0` `+0`
     * `NaN`
     * `''`（空字符串）
     * 表达式! 或 !! 隐式

### 自动转换

> 自动转换是以强制转换为基础的。
>
> 遇到以下三种情况时，JavaScript会自动转换数据类型，即转换是自动完成的，对用户不可见。

```javascript
// 1. 不同类型的数据互相运算
123 + 'abc' // "123abc"

// 2. 对非布尔值类型的数据求布尔值
if ('abc') {
  console.log('hello')
}  // "hello"

// 3. 对非数值类型的数据使用一元运算符（即“+”和“-”）
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```

> 自动转换规则：预期什么类型的值，就调用该类型的转换函数。比如，某个位置\(运算符两侧\)预期为字符串，就调用String函数进行转换。如果该位置即可以是字符串，也可能是数值，那么默认转为数值。

1. 自动转比尔值

   > 当JavaScript遇到预期为布尔值的地方（比如`if`语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用`Boolean`函数。

   除了以下六个值，其他都是自动转为`true`。

   * `undefined`
   * `null`
   * `-0`
   * `0`或`+0`
   * `NaN`
   * `''`（空字符串）

2. 自动转字符串

   > 字符串的自动转换，主要发生在加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

3. 自动转数值

   > 除了加法运算符有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。

```text
var obj={a:1,b:'2'}
console.dir('输出：'+obj)  // 输出：[object Object]

问：js中输出一个object对象显示的是[object Object]是什么意思？

网友答：object的prototype链中都没有实现自己的baitoString()的话, 把object转换为String时就会调用Object.prototype.toString, 输出的格式是[object 对象的类型]。

解：字符串拼接对象出现这种情况，实则就是数据类型转换问题。将 obj 对象利用 JSON.stringify() 转成字符串即可。
var obj={a:1,b:'2'}
console.dir('输出：'+JSON.stringify(obj))  // 输出：{"a":1,"b":"2"}
```

