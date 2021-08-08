# 对象拷贝方法

```javascript
// 原始对象
let obj = {
    name: 'wife',
    age: 20,
    alive: true,
    adress: { city: 'xg' },
    sayHi: function() { return 'i love you' }
}
obj
```

## 1. 原始方式

> 复制对象的原始方法是循环遍历原始对象，然后一个接一个地复制每个属性。该方法属于浅拷贝。

```javascript
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

* `objCopy` 对象具有一个新的 `Object.prototype`方法，这与 `mainObj` 对象的原型方法不同，这不是我们想要的。我们需要精确的拷贝原始对象。 
* 属性描述符不能被复制。值为 `false` 的 “可写\(`writable`\)” 描述符在 `objCopy` 对象中为 `true` 。 
* 只复制了 `mainObj` 的可枚举属性。
* 如果原始对象存在一个值为对象的属性，那么该属性被拷贝的是引用地址，该 **属性中的属性** 是被 **原始对象** 和 **副本对象** 共同使用的。即修改**属性中的属性**时 **原始对象** 和 **副本对象** 都将产生变化；而修改 **属性** 改动的是引用地址， **原始对象** 和 **副本对象** 只会单方面变化。

## 2. 浅拷贝对象

> 当拷贝源对象的顶级属性被复制而没有任何引用，并且拷贝源对象存在一个值为对象的属性，被复制为一个引用时，那么我说这个对象被浅拷贝。如果拷贝源对象的属性值是对象的引用，则只将该引用值复制到目标对象。
>
> 浅层复制将复制顶级属性，但是嵌套对象将在原始（源）对象和副本（目标）对象之间是共享。

```javascript
// 得出缺陷1
// Object.assign() 方法用于将从一个或多个原始对象中的所有可枚举的属性值复制到目标对象。
let objCopy = Object.assign({}, obj)
objCopy
```

```javascript
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

* 如果原始对象存在一个值为对象的属性，那么该属性被拷贝的是引用地址，该 **属性中的属性** 是被 **原始对象** 和 **副本对象** 共同使用的。即修改**属性中的属性**时 **原始对象** 和 **副本对象** 都将产生变化；而修改 **属性** 改动的是引用地址， **原始对象** 和 **副本对象** 只会单方面变化。
* 原型链上的属性和不可枚举的属性并没有被复制。

## 3. 深拷贝对象

> 深度拷贝将拷贝遇到的每个对象。原始对象和副本对象不会共享任何东西，所以它将是原始对象的副本。

```javascript
// 得出缺陷1
let objCopy = JSON.parse( JSON.stringify(obj) )
objCopy
```

```javascript
// 当对象没有二级属性，即没有 属性的属性 时此方法变相的为深拷贝
let objCopy = Object.assign({}, obj)
objCopy
```

```javascript
// 使用递归的方式，得出缺陷2
function deepCopy(obj) {
  var objCopy = Array.isArray(obj) ? [] : {};  // 根据 原始对象 类型创建对应的 副本对象
  if ( obj && typeof obj === "object" ) {  // 原始对象不能为空，并且是对象
    for (key in obj) {  // 遍历可枚举属性
      if (obj.hasOwnProperty(key)) {  // 返回一个布尔值，指示对象自身属性中是否具有指定的属性
        if (obj[key] && typeof obj[key] === "object") {  // 对象拷贝
          objCopy[key] = deepCopy(obj[key]);
        } else {  // 数组拷贝 and 复制属性
          objCopy[key] = obj[key];
        }
      }
    }
  }
  return objCopy;
}
let objCopy = deepCopy(obj)
console.log(objCopy)
```

**缺陷：**（完美深拷贝参看 5. 使用扩展运算符）

* 此方法不能用于复制用户定义的对象方法。 即 **obj.sayHi** 并没有被复制。
* 原型链上的属性和不可枚举的属性并没有被复制。

## 4. 复制循环引用对象

> 循环引用对象是具有引用自身属性的对象。

```javascript
let obj = { 
  a: 'a',
  b: { 
    c: 'c',
    d: 'd',
  },
}

obj.c = obj.b;
obj.e = obj.a;
obj.b.c = obj.c;
obj.b.d = obj.b;
obj.b.e = obj.b.c;

let newObj2 = Object.assign({}, obj);

console.log(newObj2);
```

**结果：**

![img](https://user-gold-cdn.xitu.io/2017/12/7/1602fdc09dcd68fe?imageslim)

**缺陷：**

* `Object.assign()` 适用于浅拷贝循环引用对象，但不适用于深度拷贝。 

## 5. 使用扩展运算符 \(...\)

> 扩展运算符（spread）是三个点（`...`）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。
>
> 该运算符主要用于函数调用。

```javascript
let objCopy = { ...obj }
objCopy
```

**缺陷：**

* 这是深拷贝\(完美\)

> 真的吗？待测试
>
> [https://www.cnblogs.com/daiwenru/p/9139810.html](https://www.cnblogs.com/daiwenru/p/9139810.html)

